# 🚀 FAANG-Level Helm Interview Questions

These questions focus on **how Helm behaves, production troubleshooting, design decisions, and Kubernetes integration**.

---

## 1. Helm Upgrade Succeeds, but Application Is Broken

### Scenario

A Helm deployment completes successfully:

```bash
helm upgrade myapp ./chart
```

Helm reports:

```text
STATUS: deployed
```

But the application is returning HTTP 500 errors.

### Question

If Helm says the deployment succeeded, why can the application still be broken? How would you troubleshoot?

### Expected Answer

Helm primarily manages the **Kubernetes resources**, not whether the application is functionally healthy.

Check:

```bash
helm status myapp
kubectl get pods
kubectl describe pod <pod>
kubectl logs <pod>
kubectl get events
```

Also verify:

* Readiness/Liveness probes
* ConfigMaps and Secrets
* Service configuration
* Ingress
* Image version
* Environment variables
* Application logs

### Key Concept

> **Helm success ≠ Application success.**

---

# 2. Helm Upgrade Causes Configuration Drift

### Scenario

A production Deployment was manually modified:

```bash
kubectl edit deployment myapp
```

Later someone runs:

```bash
helm upgrade myapp ./chart
```

The manual change disappears.

### Question

Why did this happen?

### Expected Answer

Helm considers the **Chart + Values** to be the desired configuration.

When Helm performs an upgrade, it renders the templates again and updates the Kubernetes resources.

Manual changes that are not represented in the Chart can therefore be overwritten.

### Key Concept

> **Helm should be the source of truth for Helm-managed resources.**

Avoid manually modifying resources managed by Helm.

---

# 3. Helm Upgrade Fails Halfway

### Scenario

A Helm release contains:

* Deployment
* Service
* ConfigMap
* Ingress

During an upgrade, some resources are updated but another resource fails.

### Question

Does Helm automatically restore everything to the previous version?

### Expected Answer

Not necessarily.

A failed upgrade does **not automatically mean every resource is restored to the previous state**.

Check:

```bash
helm status myapp
helm history myapp
kubectl get events
```

If required, explicitly rollback:

```bash
helm rollback myapp <REVISION>
```

### Key Concept

> **Failed upgrade ≠ automatic full rollback.**

Understand Helm's upgrade and rollback behavior rather than assuming transactional behavior across all Kubernetes resources.

---

# 4. `helm upgrade` vs `helm upgrade --install`

### Scenario

A CI/CD pipeline runs:

```bash
helm upgrade myapp ./chart
```

It fails because the release doesn't exist.

### Question

How would you make the pipeline work for both first deployment and subsequent deployments?

### Expected Answer

Use:

```bash
helm upgrade --install myapp ./chart
```

This means:

```text
Release exists
      ↓
    Upgrade

Release doesn't exist
      ↓
    Install
```

### Key Concept

> **`upgrade --install` = Install if missing, otherwise upgrade.**

---

# 5. Helm Values Precedence

### Scenario

You have:

```text
values.yaml
values-prod.yaml
```

`values.yaml` contains:

```yaml
replicaCount: 2
```

`values-prod.yaml` contains:

```yaml
replicaCount: 5
```

You run:

```bash
helm upgrade myapp ./chart -f values-prod.yaml --set replicaCount=10
```

### Question

What value should Helm use?

### Expected Answer

The final value is:

```yaml
replicaCount: 10
```

Because higher-precedence overrides win.

Conceptually:

```text
values.yaml
     ↓
values-prod.yaml
     ↓
--set
     ↓
Final value
```

### Key Concept

> **Later/higher-precedence values override earlier values.**

---

# 6. Helm Secret Management

### Scenario

A developer puts a database password inside:

```yaml
values.yaml
```

and commits it to Git.

### Question

Is Helm itself a secure secret-management system?

### Expected Answer

No.

Helm can create Kubernetes Secrets, but putting plaintext credentials in Git is a security risk.

Production environments commonly integrate Helm with external secret-management solutions such as:

* Vault
* Cloud secret managers
* External Secrets Operator
* Sealed Secrets

### Key Concept

> **Helm deploys Secrets; it should not automatically be treated as your secret-management solution.**

---

# 7. Helm Chart Dependency Problem

### Scenario

Your application depends on:

```text
Redis
PostgreSQL
```

You want these dependencies packaged with your application Chart.

### Question

How would you design this?

### Expected Answer

Helm supports **Chart dependencies**.

A parent Chart can declare dependent Charts in `Chart.yaml`.

Conceptually:

```text
Application Chart
      │
      ├── Redis Chart
      └── PostgreSQL Chart
```

Dependencies can then be managed using Helm dependency commands.

### Key Concept

> **Parent Chart → Dependency Charts**

However, in production, databases are often managed separately when lifecycle, persistence, upgrades, and availability requirements justify it.

---

# 8. Helm Hook Scenario

### Scenario

Before deploying a new application version, you need to run a database migration.

### Question

How could Helm help?

### Expected Answer

Helm supports **Hooks**.

A migration Job can be configured as a Helm hook, for example using:

```yaml
annotations:
  helm.sh/hook: pre-upgrade
```

The Job runs during the Helm lifecycle.

### Key Concept

> **Hooks allow Kubernetes resources to participate in Helm lifecycle events.**

But hooks should be used carefully because migration failures and retry behavior can complicate deployments.

---

# 9. Two Teams Modify the Same Release

### Scenario

Team A runs:

```bash
helm upgrade myapp ./chart
```

Team B runs another Helm upgrade shortly afterward using different values.

### Question

What problems could occur?

### Expected Answer

The release can end up with configuration determined by the latest successful upgrade.

Potential problems include:

* Unexpected configuration changes
* Race conditions in CI/CD
* Different teams using different Chart versions
* Difficult-to-trace deployment history

A production approach should have:

* Single deployment ownership
* CI/CD controls
* Versioned Charts
* Controlled release process
* Clear environment ownership

### Key Concept

> **One release should have controlled deployment ownership.**

---

# 10. Helm Chart Version vs Application Version

### Scenario

You have:

```yaml
version: 2.1.0
appVersion: "5.4.2"
```

### Question

What is the difference?

### Expected Answer

`version` represents the **Helm Chart version**.

`appVersion` represents the **application version** being deployed.

Example:

```text
Chart version → 2.1.0
Application → 5.4.2
```

The Chart can change without changing the application version.

### Key Concept

> **Chart version ≠ Application version.**

---

# 11. Helm Rollback Is Not Always Enough

### Scenario

Version 10 introduced a database schema migration.

The application is now broken.

You run:

```bash
helm rollback myapp 9
```

### Question

Why might rollback not completely solve the problem?

### Expected Answer

Helm rollback primarily restores the Kubernetes resources represented by the previous release.

It does **not automatically undo external side effects**, such as:

```text
Database schema migration
External API changes
Cloud resources
Persistent data changes
```

For example:

```text
Release 10
   ↓
DB migration
   ↓
Rollback to Release 9
   ↓
Application code reverted
   ↓
Database schema may still be changed
```

### Key Concept

> **Application rollback ≠ Database rollback.**

---

# 12. Helm Template Debugging

### Scenario

Your Helm deployment fails with an invalid Kubernetes manifest.

### Question

How would you debug the generated YAML before applying it?

### Expected Answer

Render the Chart:

```bash
helm template myapp ./chart
```

You can also use:

```bash
helm lint ./chart
```

and:

```bash
helm install myapp ./chart --dry-run --debug
```

This helps separate:

```text
Helm templating problem
        vs
Kubernetes runtime problem
```

### Key Concept

> **First inspect what Helm actually renders.**

---

# 13. Helm + Kubernetes Rollout Problem

### Scenario

Helm upgrade succeeds, but the new Pods never become Ready.

### Question

What would you check first?

### Expected Answer

Start with:

```bash
kubectl get pods
kubectl describe pod <pod>
kubectl logs <pod>
kubectl get events
```

Then investigate:

```text
Image
↓
Container startup
↓
Environment variables
↓
ConfigMap/Secret
↓
Readiness probe
↓
Service
↓
Ingress
```

### Key Concept

> **Helm manages the desired resources; Kubernetes determines whether Pods actually become healthy.**

---

# 14. Production Helm Design

### Scenario

You have 50 microservices and 4 environments:

```text
Dev
QA
Stage
Production
```

You don't want 200 separate Helm Charts.

### Question

How would you design the Helm structure?

### Expected Answer

Prefer reusable Charts with environment-specific configuration.

For example:

```text
Chart
 │
 ├── values-dev.yaml
 ├── values-qa.yaml
 ├── values-stage.yaml
 └── values-prod.yaml
```

Or use an appropriate GitOps/configuration structure depending on organizational needs.

Avoid copying the entire Chart for every environment.

### Key Concept

> **Reuse the Chart; vary configuration.**

---

# 15. The Senior-Level Question

### Scenario

Your company uses Helm for 500+ microservices.

Engineers complain that:

* Charts are difficult to maintain
* Values files are huge
* Teams copy/paste Charts
* Upgrades frequently cause unexpected changes
* Rollbacks are unreliable for database changes

### Question

How would you improve the Helm architecture?

### Expected Answer

A strong answer should discuss:

```text
Standardized Chart structure
        ↓
Reusable templates
        ↓
Clear values design
        ↓
Chart versioning
        ↓
CI validation
        ↓
helm lint
        ↓
helm template
        ↓
Automated testing
        ↓
Controlled deployments
        ↓
Release history
        ↓
Rollback strategy
```

Also separate concerns:

```text
Application deployment
        ≠
Database lifecycle
        ≠
Secret management
        ≠
Infrastructure provisioning
```

For larger organizations, Helm can be combined with **GitOps tooling** so that Git becomes the desired-state source and deployments are consistently reconciled.

### Key Concept

> **Helm is a packaging and release-management tool—not the entire deployment architecture.**

---

# 🧠 FAANG Helm Mental Model

When you get a difficult Helm question, think in this order:

```text
              HELM
                │
       ┌────────┼────────┐
       ↓        ↓        ↓
     CHART     VALUES   RELEASE
       │        │        │
       ↓        ↓        ↓
   Templates  Config   Installed
       │                 Version
       └────────┬────────┘
                ↓
        Rendered YAML
                ↓
           Kubernetes
                ↓
      Pods / Services / etc.
```

## 🔑 10 Concepts You Should Never Forget

1. **Chart = Package/Blueprint**
2. **Templates = Kubernetes YAML with variables**
3. **Values = Configuration**
4. **Release = Installed instance of a Chart**
5. **Upgrade = Change an existing release**
6. **Rollback = Return to a previous Helm release revision**
7. **Helm success ≠ Application health**
8. **Helm rollback ≠ Database rollback**
9. **Secrets need proper secret-management practices**
10. **Helm is one component of a production deployment architecture**

### ⭐ The Golden Interview Answer

> **“Helm packages Kubernetes applications using reusable Charts. Templates define the Kubernetes resources, Values provide environment-specific configuration, and a Release represents an installed instance of the Chart. Helm manages installation, upgrades, history, and rollbacks, while Kubernetes remains responsible for actually running and maintaining the workloads.”**
