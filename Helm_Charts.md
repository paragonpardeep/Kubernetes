# Helm Charts in Kubernetes

## 🧠 Helm in One Line

**Helm = Package Manager for Kubernetes**

Instead of manually creating many Kubernetes YAML files, Helm lets you **package, configure, install, upgrade, and rollback** an application.

```text
Helm Chart → Kubernetes Resources → Helm Release
```

---

## 📦 What is a Helm Chart?

A **Chart** is a folder containing Kubernetes templates and configuration.

Typical structure:

```text
my-app/
├── Chart.yaml        # Chart information
├── values.yaml       # Default configuration
└── templates/        # Kubernetes YAML templates
    ├── deployment.yaml
    ├── service.yaml
    └── ingress.yaml
```

### Remember:

**Chart = Kubernetes application package**

---

## ⚙️ The 4 Things to Remember

### 1. Chart

The **blueprint/template** of your application.

```text
Chart → What to deploy
```

### 2. values.yaml

Contains configurable values.

```yaml
replicaCount: 3
image:
  repository: nginx
  tag: "1.25"
```

```text
Values → How to configure it
```

### 3. templates/

Contains Kubernetes YAML with Helm variables.

```yaml
replicas: {{ .Values.replicaCount }}
```

```text
Templates + Values → Final Kubernetes YAML
```

### 4. Release

A **running installation of a Chart**.

```bash
helm install myapp ./my-app
```

Here:

```text
my-app = Chart
myapp  = Release
```

---

## 🌍 Different Environments

Use the same Chart with different values.

```text
             Same Chart
                 │
       ┌─────────┼─────────┐
       ↓         ↓         ↓
    Dev         Test      Prod
 values-dev   values-test values-prod
```

Example:

```bash
helm install myapp ./chart -f values-dev.yaml
helm install myapp ./chart -f values-prod.yaml
```

**Remember:**

> One Chart → Many Environments → Different Values

---

## 🔄 Helm Upgrade

Change configuration or application version:

```bash
helm upgrade myapp ./chart
```

**Remember:**

> Change Values → Helm Upgrade → Kubernetes Updated

---

## ↩️ Helm Rollback

If the new deployment causes a problem:

```bash
helm history myapp
helm rollback myapp 1
```

**Remember:**

> Upgrade failed → Check History → Rollback

---

## 🔍 Check Before Deploying

Render the Kubernetes YAML without installing:

```bash
helm template myapp ./chart
```

Test an installation:

```bash
helm install myapp ./chart --dry-run
```

**Remember:**

> `helm template` = See what will be created

---

## 🛠️ Most Important Commands

```bash
helm create my-app             # Create a chart
helm lint ./my-app             # Validate chart
helm template myapp ./my-app   # Render YAML
helm install myapp ./my-app    # Install
helm list                      # List releases
helm upgrade myapp ./my-app    # Upgrade
helm history myapp             # Release history
helm rollback myapp 1          # Rollback
helm uninstall myapp           # Remove release
```

---

## 🧠 Interview Memory Trick

Remember this flow:

```text
CHART
  ↓
VALUES
  ↓
TEMPLATES
  ↓
RENDERED YAML
  ↓
RELEASE
  ↓
UPGRADE / ROLLBACK
```

### One-line definition:

> **Helm uses Charts as Kubernetes application packages, Values for configuration, Templates to generate Kubernetes manifests, and Releases to manage deployed applications.**



