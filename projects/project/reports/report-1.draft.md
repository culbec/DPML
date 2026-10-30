# Report 1 Draft

## Introduction

### Application Scope

Disseration Thesis: **Evolving Interpretable Ensembles with Learned Fuzzy-Integral Fusion for Failure Detection and Root Cause Analysis in Microservice Systems**.

Developers, especially DevOps teams, manage lots of applications that have separate workflows,
dependencies, goals, or use cases. Nevertheless, every deployed applications has at least encountered once
liveness issues that come up from a variety of sources: network issues, resource issues, bad configuration,
provisioning delays, and so on.

I propose an ML-based application named **Genomaly** that is designed to help developers and software
maintainers alike to detect software anomalies and failure points using provided KPIs of their services
and cloud environments.

### Involved Parties

The application is designed specifically for technical peers that desire to be pro-active and strive
for continuous application uptime that they manage. These users are expected to use the system by
providing system data and cloud cluster data that is extracted in specific ways by them and comply
to an internal schema.

In order to open the doors of this application to less technical users, we strive to guide users
that somehow managed to deploy an application to analyze their system statistics in order to find
anomalies in their system macro and micro values; our goal is to make the application intuitive
enough in order to support many standard formats with the goal to make the anomaly/failure detection
self-explanatory for easier undersanding.

### User Interest

Considering the current state of software, the development lifecycle has shrinked a lot; more and more applications and releases are getting deployed in short periods of time, with more people or public services transitioning to digital materials.

Given this fact, it is most crucial to be considerate about the failure points your service might expose. As an application
grows in popularity it also might exceed prototype expectations and initial application drafts. Software maintainers
would strive to keep a high rate of uptime, thus the need of this application.

This ML-based service would provide users with actual analytics about the state of their service and whether
it is prone to failure or some anomalies regarding production telemetry.

## Ethical AI Principles Analysis

We will analyze the application workflow considering the following datasets:

- RCAEval: https://huggingface.co/datasets/phamquiluan/RCAEval
- ServerMachineDataset (SMD): https://github.com/NetManAIOps/OmniAnomaly
- Nehza-used Datasets: https://github.com/IntelligentDDS/Nezha
- AIClusterKPI: https://zenodo.org/records/19926114

These datasets all provide cloud-native telemetry and microservice data from a curated set of
cases.

We will analyze the following principles:

- Fairness, non-isolation, and bias: we will focus on the data format and analyze whether we encounter issues
while using these datasets; we should consider class unbalance, focused or less generalistic settings, and so on.
- Safety and security: we should assess whether this application is susceptible to misuse, where actors would pose
as legitimate users and use the system to assess the failure points of an application, and then attack it.
- Transparency and explainability: providing anomalies, failure points, or fault injection results from
application and cloud telemetry should be made clear enough for any user to understand the results provided
by the system, while also making it clear enough the action points that need to be addressed.
- Human oversight and determination: the application should not act on its own and begin modifying actual
software configuration and deployment components without being assessed by a human first; the goal is to
provide actual maintainers data to understand the problems in their setting, rather than blindly trusting
a system to provide modification suggestion without understanding the instrinsics of a system.

More details about these principles on [UNESCO](https://www.unesco.org/en/artificial-intelligence/recommendation-ethics).

### Fairness, non-discrimination, and bias

#### Issue 1 - Limited genuine data

The data needed for this kind of system that we want to develop is scarce. From a security perspective, providing
undisclosed data of this kind poses a security risk, given that attackers can use it to target systems
susceptible to issues.

To solve this, we would need to combine a lot of separate datasets and preprocess them in such way that we
are left with a standard format that combines multiple data sources. In this way, we can get multiple failure
points from multiple software systems that would leave as with many scenarios of failures and normal behavior.

#### Issue 2 - Unbalanced classes

Mostly, applications behave correctly. There are less cases when applications misbehave when they are mature (fault-injection
is mainly mostly present in prototypes, when the application is taking shape or when dependent services are failing).

However, given this setting we need to address class imbalance in order to provide the system data in many shapes 
and points to be diverse enough for this system to consider many scenarios.

#### Issue 3 - Common failure points bias

There are more common failure points to an application than others; for example, network failures or resource issues are more common than cluster node drain.

We should attend to mitigate this bias by generating synthethic samples that tackle multiple failure points.

#### Issue 4 - Continuous learning

The application should be able to continuously learn using online learning methods to tackle more scenarios
that may emerge when the current technologies are overran.

### Safety and security

#### Issue 1 - Misuse to attack legitimate systems

Attackers may use the system to exploit systems by using ill-retrieved data. Such data can be the result of breaches that affect
multiple systems, and attackers may use the system to distribute an attack to take down essential services.

The system shall be used by trusted peers that have authorized access to it, trusted by the ML-based system owners.
We shall not allow anyone to use the system to get insights about their applications without reviewing
their legitimacy first.

#### Issue 2 - Data breach

Considering the amounts of data that such application would need to function properly we would need to keep
an archive or checkpoints of such data in order to review what the model is trained on and how it arrived
to take decisions and to build its architecture using the provided data.

In case of a data breach, client data would be lost; we cannot allow that, and we can tackle this by using zero-encryption or
multi-tenancy to make sure that the data is sharded and no single source of truth is responsible for all data present.

#### Issue 3 - Model poisoning / exposure

Having a model of such kind (ensemble of GP-generated programs that are transparent by nature), attackers could exploit the
application into retrieving the actual programs generated by GP that drive the decision-making system.

This poses a great security risk, as the system would be compromised and would compromise the IP and also the systematics that
can be used by attackers to exploit other systems. I propose encrypting the model and making it secure by attempting
to cover the weak learners in the ensemble.

### Transparency and explainability

#### Issue 1 - Transparent decision-making process

Considering the critical domain that this application acts in (evaluating product telemetry to assess anomalies), the
decision-making process shall be explainable to peers in order to understand where their applications might fail.

#### Issue 2 - Transparent data

We shall make the data used to power the training step of the model as transparent as possible, stating the systems that the
system analyzed, the load that these analyzed systems encounter, and transparent feature analysis in order for maintainers
to be able to trust the data that was used to train such product. This is done in the design document in my opinion.

#### Issue 3 - Intuitive explanations

The final decision taken by the system considering a scenario described by data should be intuitive for maintainers
to find fast the action points that they need to take.

### Human oversight and determination

#### Issue 1 - Over-zealous acting capabilities

The system shall not be able to make modifications to the analyze system by proposing and applying changes to the system
in order to automatically fix the failure points found. An actual analyst would need to consider the points stated and
act upon them, the ML-based system being an augment to human reasoning.

#### Issue 2 - Over confident failure points

The system shall be contradictable and improve itself when it is wrong. When a user states that some failure points are wrong,
the system shall acknowledge the results and modify its internals such that this doesn't happen again.

## Conclusions

Conclusions of the system's strong and weak points revealed by the analysis.
