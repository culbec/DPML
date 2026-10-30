# Report 1

Profir Alexandru, ICA 256/2

**Title:** Genomaly: ethical analysis of an interpretable failure-detection assistant for cloud microservice systems

## Description

### What the application does

Genomaly is the application side of my dissertation thesis, *Evolving Interpretable Ensembles with Learned Fuzzy-Integral Fusion for Failure Detection and Root Cause Analysis in Microservice Systems*.

The user uploads, or streams in batches, telemetry from a running system: metrics (CPU, memory, latency, error rates), trace summaries and log counts. Genomaly answers four questions for each time window:

1. **Detection:** is the system in an anomalous or failure state?
2. **Diagnosis:** what kind of fault is it (CPU, memory, disk, network delay or loss, code-level)?
3. **Localisation:** which service, node or metric is the most likely root cause?
4. **Explanation:** which rules and signals produced the answer?

The model is an ensemble of small, readable classifiers evolved by Genetic Programming (GP). Their scores are fused by a Choquet fuzzy integral with a learned fuzzy measure. The Shapley and interaction indices of that measure show which members, and so which services and signals, drove each decision. Genomaly only reports and explains. It never changes the user's system.

### Who it is for

- **Primary users:** DevOps engineers, Site Reliability Engineers (SREs) and on-call developers who run cloud or Kubernetes applications. They are adults with technical training, from startups to large companies, and the system makes decisions about software, not about people.
- **Secondary users:** less experienced maintainers, such as small teams, students and solo developers, who deployed an application but have no dedicated operations staff. They need plain-language explanations and support for common telemetry formats (Prometheus, OpenTelemetry).
- **Indirect beneficiaries:** the end users of the monitored services, because less downtime means more reliable public and private digital services.
- **Administrators:** the Genomaly operators. They vet accounts, manage tenant data, retrain and version models, and handle incidents. Training data comes from public research datasets (below).

### What users gain

- Faster triage: a ranked list of suspect services and metrics instead of manual dashboard search through hundreds of series.
- Explanations they can check against their own knowledge, which builds or withholds trust for good reasons.
- Fewer false alarms on benign load events (such as metric scraping or log rotation), which reduces alert fatigue.
- For less experienced teams, a guided path into incident analysis that would otherwise need operations expertise.

## Ethical AI Principles Analysis

The analysis uses the datasets planned for training and evaluation:

- [RCAEval](https://zenodo.org/records/14590730) RE1–RE3: 735 fault-injection cases on three demo microservice apps (Online Boutique, Sock Shop, Train Ticket). It has metrics, logs and traces, with fault-type, root-cause service and indicator labels. Licence CC-BY-4.0 / MIT.
- [SMD](https://github.com/NetManAIOps/OmniAnomaly): 28 real production machines, 38 anonymised metrics, 327 anomaly segments with interpretation labels. MIT licence.
- [Nezha](https://github.com/IntelligentDDS/Nezha): 101 multi-modal fault cases on Online Boutique and Train Ticket, including code-level faults. MIT licence.
- [AIClusterKPI](https://zenodo.org/records/19926114): 31 cluster KPIs with 145 injected faults and 453 labelled benign load events. CC-BY-4.0, no accompanying paper.

The four chosen [UNESCO principles](https://www.unesco.org/en/artificial-intelligence/recommendation-ethics) are *Fairness and non-discrimination*, *Safety and security*, *Transparency and explainability* and *Human oversight and determination*. The other UNESCO principles are covered as follows:

- **Right to privacy:** applies, because logs and traces can contain personal data. It is handled under Safety and security (S2).
- **Responsibility and accountability:** applies. It is handled under Human oversight (H4).
- **Awareness and literacy:** applies to less experienced users. It is handled under Transparency (T2).
- **Proportionality and do no harm:** partly covered by S5.
- **Sustainability:** applies only weakly. GP evolution costs compute, but the ensemble is small and cheap at inference.
- **Multi-stakeholder governance:** does not apply in a meaningful way to a single-organisation operations tool.

### Principle 1: Fairness and non-discrimination

| # | Issue | Possible mitigation |
|---|---|---|
| F1 | The datasets contain no demographics (gender, age, race). This is not a gap: the data describes machines and services, not people, so demographic bias does not apply. Bias moves to *which systems and which teams* the model serves well. | State this in the documentation. Check the telemetry for personal attributes (see S2). Define fairness as equal quality across system types, stacks and levels of instrumentation, and measure it that way (F2, F6). |
| F2 | **Narrow system coverage.** Training data comes from three demo microservice apps on Kubernetes and one company's servers (SMD). Monoliths, serverless, VM-based or other-language stacks are under-represented, so their users get worse results without knowing it. | Report results per system. Use leave-one-system-out and cross-dataset (RCAEval to Nezha) tests. Use service-agnostic features (top-k deviating services) instead of fixed service names. Publish the supported scope in a model card, and warn when input falls outside the training distribution. |
| F3 | **Injected faults differ from real failures.** Testbed faults are strong and clean: a depth-5 tree detects RCAEval failures at 0.90–0.94 balanced accuracy. Real incidents are gradual and overlap, so the model may look better in evaluation than in practice. | Base detection claims on real production data (SMD) and on AIClusterKPI's benign events. Use RCAEval mainly for diagnosis and localisation, where it is not trivial. |
| F4 | **Scarce and unbalanced data.** Labelled failure data is rare: RCAEval has 735 cases and Nezha 101. Within each set failures are a minority (SMD: 4.2% anomalous points on average; AIClusterKPI: 145 faults vs 453 benign events). Cost-sensitive learning alone cannot add diversity, and a model can score high accuracy by predicting "normal". | Generate synthetic minority samples with imbalanced-learning oversampling (e.g. SMOTE, Borderline-SMOTE, ADASYN) on the real windowed features. Apply it only to the training folds, after the case-grouped split, so no synthetic sample is derived from a test case. Evaluate on real data only, and report PR-AUC and event-wise F1 instead of accuracy. Combine with cost-sensitive GP fitness. This differs from the excluded generator-based datasets, whose rule-based labels shallow trees learned at 0.91–0.999. |
| F5 | **Common-fault bias, which oversampling can amplify.** Resource faults (CPU, memory, delay) dominate; code-level faults appear only in RCAEval RE3 (30 cases per system) and Nezha. Samples interpolated from a few rare cases repeat those cases' quirks; they do not add new failure behaviour. | Oversample per fault type and system, not globally. Compare results with and without oversampling, and check that synthetic samples stay close to real ones (distribution tests, nearest-neighbour distance). Report macro-F1 and per-class confusion matrices. Use lexicase selection to keep specialist members for rare classes, and flag low-confidence diagnoses for rare fault types. |
| F6 | **Unequal telemetry richness.** Well-equipped teams have metrics, traces and logs; small teams often have metrics only (Sock Shop has no traces either). The multi-modal variant would favour already well-resourced organisations. | Keep a metric-only model as the baseline that every user gets. Report performance per modality set. Guide users toward low-cost instrumentation (OpenTelemetry). |

### Principle 2: Safety and security

| # | Issue | Possible mitigation |
|---|---|---|
| S1 | **Dual use for attack reconnaissance.** An attacker with stolen or scraped telemetry from a target can use Genomaly to find its weakest services and fault types, then aim a targeted attack (e.g. resource exhaustion on the root-cause service). | Vet accounts (organisation verification, terms of use). Require proof of ownership of the analysed system, such as an agent token installed in the cluster or signed telemetry export. Rate-limit requests and log every analysis for audit. |
| S2 | **Data breach and personal data in telemetry.** Logs and traces can contain user IDs, IP addresses, emails and request URLs. A breach would expose both customer infrastructure details and personal data of their end users (GDPR scope). | Zero retention by default: telemetry is analysed in memory and deleted once the result is returned, so client data stays the client's responsibility. Data is kept only when a tenant opts in to contribute it for retraining. Reduce logs to counts and Drain template IDs on the client side before upload, so raw text never leaves the user. Pseudonymise service and host names, and encrypt in transit (TLS). Opt-in data is encrypted at rest under per-tenant keys and deleted on request. |
| S3 | **Model extraction and IP exposure.** The GP members are readable formulas. If they leak, attackers learn the exact thresholds that trigger an alert and can stay just below them, and competitors can copy the model. | Keep the members, thresholds and fuzzy measure private. Users get explanations at the level of signals and services (T2), not the internal formulas. Full internals are open only to internal reviewers and independent auditors under agreement. Serve the model behind access control and query limits, and detect extraction-style query patterns. Accept the remaining risk: attributions still reveal which signals matter. |
| S4 | **Data and feedback poisoning.** Malicious tenants could upload crafted telemetry or false corrections so that retraining learns to ignore a certain attack pattern. | Never retrain directly on user data or feedback. Keep tenant data separate from the global training set. Review feedback before use (H3). Validate every retrained model against a fixed, trusted benchmark (RCAEval, SMD) and roll back on regressions. |
| S5 | **Harmful errors.** A false negative lets an outage go unnoticed; frequent false positives cause alert fatigue, so operators start ignoring real alerts. | Tune the operating point with users and report both rates. Test false positives on AIClusterKPI benign events. Show confidence with each alert. Position Genomaly as a complement to, not a replacement for, existing alert rules. |
| S6 | **Silent degradation over time (concept drift).** New releases, traffic patterns and technologies change the telemetry, and accuracy can drop without anyone noticing. Online self-learning would add poisoning risk (S4). | Monitor feature distributions per tenant and warn on drift. Retrain on a schedule, offline, with versioned models and the S4 validation gate. |

### Principle 3: Transparency and explainability

| # | Issue | Possible mitigation |
|---|---|---|
| T1 | **Unfaithful explanations.** Post-hoc explainers (e.g. SHAP on gradient boosting) explain a model, not the system, and can point to the wrong cause. In incident response, a wrong cause wastes time during an outage. | Build explanations into the model: the Choquet sum splits exactly into member contributions (Shapley values) and pairwise interactions. Evaluate them against ground truth: HitRate@P% against SMD interpretation labels and indicator hit rate on RCAEval and Nezha, compared with SHAP on XGBoost. |
| T2 | **Explanations that are too technical, or that expose the model.** An internal member such as `z(p99_latency_cart) − 0.4·slope(mem_redis) > 1.2`, or an interaction index, means little to a junior developer. Showing it would also leak IP (S3). | Explain the decision, not the model. Translate member contributions into signal and service contributions. The default view is a ranked list plus a sentence such as "latency of *cart* rose sharply while *redis* memory grew". The expert view adds contribution scores per metric and the strongest signal pairs that reinforce each other. Formulas stay internal (S3). Add short documentation on reading the results. |
| T3 | **Transparency about training data and scope.** Users cannot judge trust without knowing what the model was trained on: three demo apps, testbed faults, one company's servers. | Publish a model card and datasheet: datasets, licences, systems, fault types, known gaps (F2–F6) and per-system results. Use only datasets with clear licences, and exclude Eadro, which states no licence. |
| T4 | **Hidden uncertainty.** A single "root cause: service X" reads as certain even when the evidence is weak or split across services. | Always show the top-3 candidates with scores, the agreement between members, and a "low confidence" flag. Show "no confident diagnosis" instead of forcing an answer. |
| T5 | **Explaining an accuracy–interpretability trade-off.** The interpretable ensemble may score a few points below black-box models (XGBoost). Users should know what they gain and what they give up. | Report benchmark results against strong baselines openly, including where Genomaly loses. Explain why a checkable rule can be worth a small loss in accuracy. |

### Principle 4: Human oversight and determination

| # | Issue | Possible mitigation |
|---|---|---|
| H1 | **Over-reaching automation.** An assistant that also restarts pods, scales services or rolls back releases could worsen an incident (e.g. restart the wrong service, or cascade during a misdiagnosis). | Give Genomaly no write access to user infrastructure. It suggests next steps; a human decides and acts. Any future remediation feature would require explicit, per-action human approval. |
| H2 | **Automation bias.** Under on-call pressure, operators may accept the top-ranked cause without checking it. | Show the evidence (metrics, rules) next to every claim, and link to the raw telemetry. Make T4 uncertainty visible. Include a "verify" step in the suggested workflow. |
| H3 | **Contestability without self-modification.** Users must be able to say "this diagnosis is wrong", but letting the model change itself from each correction would be unsafe (S4). | Record each correction with its incident. Use corrections immediately to suppress repeat alerts for that tenant only. Feed reviewed corrections into the next scheduled retraining. |
| H4 | **Accountability for missed incidents.** When an outage is missed or misdiagnosed, it is unclear who is responsible: the tool, the operator or the vendor. | State in the terms that Genomaly is decision support and that the operator remains responsible for actions. Keep audit logs of the model version, a hash of the input and the output of each analysis, without raw telemetry (S2). Post-incident reviews can then trace what the system showed. |

## Conclusions

**Strong points.**

- Transparency is the most achievable principle, because explanation is built into the method. The learned fuzzy measure gives Shapley and interaction attributions that can be scored against ground-truth labels instead of judged by eye.
- Human oversight is easy to guarantee by design: the system has no write access and only advises.
- Classical demographic bias does not apply, because no personal attributes are used.

**Weak points.**

- Fairness across *systems* is hard to guarantee. Public labelled data covers only a few demo applications and one production fleet, so performance on other stacks is unknown and must be stated as a limitation.
- Safety is the most delicate principle. The tool is dual-use (reconnaissance), telemetry can carry personal data, and readable models are easier to extract and evade.
- Scarce labelled data is the main limit. Oversampling balances the classes but cannot invent failure behaviour that is not in the data.
- Transparency and IP protection must be balanced. Genomaly explains *decisions* (signals and services) while keeping the model's internals private. This reduces the tension but does not remove it, since attributions still reveal what the model watches.

## Presentation notes (5 minutes)

**Elevator pitch (about 1 minute).** Teams running cloud microservices drown in metrics, traces and logs when something breaks. Genomaly reads that telemetry and tells them that something is wrong, what kind of fault it is, which service started it, and *why it thinks so*. The model is a small ensemble of readable rules evolved by genetic programming, fused by a fuzzy integral whose weights also explain the decision. Users are DevOps and SRE engineers, plus small teams without operations experience. Indirect beneficiaries are their end users. Genomaly advises; it never touches the system.

**Transparency and explainability (about 2 minutes).**

- T1: the explanations come from the model itself (Shapley values of the fuzzy measure), not a post-hoc explainer, and they are scored against real root-cause labels.
- T2: there are two levels of explanation, plain language for juniors and per-metric contributions for experts. Both explain the decision without exposing the model's formulas.
- T4: uncertainty is shown as top-3 candidates and "no confident diagnosis", which prevents false certainty.

**Safety and security (about 2 minutes).**

- S1: an attacker could use the tool for reconnaissance. Mitigations are account vetting and proof of system ownership.
- S2: logs can contain personal data. The service keeps no data by default (opt-in only for retraining), and logs become counts on the client side.
- S3: readable rules are valuable IP and help attackers evade detection, so they stay private. Users get signal-level explanations instead.
- S4: there is no online self-learning. Retraining is reviewed and gated against a trusted benchmark.
