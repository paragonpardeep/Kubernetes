Advanced Kubernetes Scenarios: Volumes, Scheduling, Networking, and Helm

Simple memory hooks, problem statements, solutions, and step-by-step handling approaches for real Kubernetes scenarios.

# How to Remember Kubernetes Troubleshooting Forever

**Always remember this flow:** Pod → Events → Config → Dependency → Logs → Fix → Validate.

**Pod:** check whether the Pod is Running, Pending, CrashLoopBackOff, or Ready.

**Events:** describe the Pod first; Kubernetes usually tells the reason in events.

**Config:** verify YAML, labels, selectors, affinity rules, ports, values, and storage configuration.

**Dependency:** check PVC, PV, Service, endpoints, DNS, node, CNI, ingress controller, or Helm chart dependency.

**Logs:** check application logs and controller logs.

**Fix:** apply the smallest safe change first.

**Validate:** confirm rollout, endpoint health, traffic flow, and application behavior.

# 1. Kubernetes Volumes: PersistentVolume and PersistentVolumeClaim

**Memory hook:** PV is the storage; PVC is the request for storage.

**Simple concept:** Pods can die, but application data should not die with the Pod.

**PV:** actual storage available in the cluster.

**PVC:** application request for storage size, access mode, and StorageClass.

**StorageClass:** decides how storage is dynamically created.

**Access mode:** controls how the storage can be mounted, such as ReadWriteOnce or ReadWriteMany.

**Never forget:** storage has its own lifecycle, location, access rule, reclaim policy, and performance behavior.

**Production thought:** for databases, always check backup, restore, zone mapping, and storage expansion before making changes.

**Most common risk:** deleting a PVC can mean deleting production data if reclaim policy and backup are not understood.

## Complex Scenario: Pod Stuck in Pending Due to Volume Node Affinity Conflict

**Problem statement:** A StatefulSet Pod is stuck in Pending after node maintenance.

**Why it happened:** The PVC is bound to a PV that can only attach to one specific node or zone.

**Impact:** Application cannot start because Kubernetes cannot mount the required volume on another node.

**Simple solution:** either restore scheduling on the original node or redesign storage to use a CSI-backed network volume that supports the required availability model.

**Approach to handle:** start from Pod status, then read events, then map PVC to PV, then check node or zone restriction.

Run **kubectl get pods -n \<namespace>** to confirm the Pod is Pending.

Run **kubectl describe pod \<pod-name> -n \<namespace>** and check scheduling events.

Run **kubectl get pvc -n \<namespace>** and identify the bound PV.

Run **kubectl get pv \<pv-name> -o yaml** and check nodeAffinity or zone mapping.

Run **kubectl describe node \<node-name>** and verify if the node is cordoned, drained, tainted, or unhealthy.

If safe, uncordon the original node using **kubectl uncordon \<node-name>**.

If the workload needs high availability, move to a network or cloud CSI StorageClass instead of node-local storage.

**Final validation:** ensure the Pod becomes Running and the application data is intact.

# 2. Affinity and Anti-Affinity

**Memory hook:** affinity means attract; anti-affinity means keep away.

**Node affinity:** place Pods on specific types of nodes.

**Pod affinity:** place Pods near other Pods.

**Pod anti-affinity:** spread Pods apart for high availability.

**Required rule:** scheduler must follow it.

**Preferred rule:** scheduler tries to follow it but can ignore it if needed.

**Never forget:** strict scheduling rules can protect availability, but they can also block deployment.

**Production thought:** use required rules only when the rule is mandatory; otherwise use preferred rules or topology spread constraints.

**Most common risk:** too many replicas with too few eligible nodes causes Pending Pods.

## Complex Scenario: High-Availability Application Cannot Scale Because Anti-Affinity Is Too Strict

**Problem statement:** A six-replica application cannot fully scale because only three Pods are running.

**Why it happened:** hard anti-affinity allows only one replica per node, but the cluster has only three worker nodes.

**Impact:** the application cannot reach the desired replica count, which may reduce capacity and resilience.

**Simple solution:** add more nodes, relax required anti-affinity to preferred anti-affinity, or use topology spread constraints.

**Approach to handle:** check Pending Pods, read scheduler events, confirm node count, then validate anti-affinity logic.

Run **kubectl get pods -n \<namespace> -o wide** to see where replicas are placed.

Run **kubectl describe pod \<pod-name> -n \<namespace>** to confirm anti-affinity scheduling failure.

Run **kubectl get nodes --show-labels** to check eligible nodes and topology labels.

Check whether the topology key is **kubernetes.io/hostname** or **topology.kubernetes.io/zone**.

If each replica must be on a different node, add more nodes.

If strict separation is not mandatory, change required anti-affinity to preferred anti-affinity.

For balanced but flexible placement, use topology spread constraints.

**Final validation:** run **kubectl rollout status deployment/\<deployment-name> -n \<namespace>**.

# 3. Kubernetes Networking

**Memory hook:** Kubernetes networking is Pod IP → Service → Endpoint → DNS → Ingress.

**Pod IP:** every Pod gets its own IP.

**Service:** stable entry point for changing Pods.

**Endpoint:** actual backend Pod IPs behind the Service.

**DNS:** resolves service names inside the cluster.

**Ingress:** exposes HTTP or HTTPS traffic from outside the cluster.

**NetworkPolicy:** controls which traffic is allowed or blocked.

**Never forget:** if Service has no endpoints, traffic has nowhere to go.

**Production thought:** always troubleshoot networking layer by layer instead of jumping directly to Ingress.

**Most common risk:** wrong labels, wrong selectors, wrong ports, blocked policies, or ingress backend mismatch.

## Complex Scenario: Service Is Reachable Internally but External Ingress Returns 503

**Problem statement:** Application works inside the cluster, but external users get HTTP 503 from Ingress.

**Why it happened:** Ingress is pointing to the wrong Service port name or backend mapping.

**Impact:** internal traffic works, but external traffic cannot reach the application through the ingress controller.

**Simple solution:** verify Pod readiness, Service endpoints, Service port, and Ingress backend configuration.

**Approach to handle:** move from inside to outside: Pod → Service → Endpoint → DNS → Ingress → Controller logs.

Run **kubectl get pods -n \<namespace>** and confirm Pods are Running and Ready.

Run **kubectl get svc,endpoints -n \<namespace>** and confirm endpoints exist.

Test Service from inside the cluster using a temporary debug Pod.

Run **kubectl describe ingress \<ingress-name> -n \<namespace>**.

Confirm the Ingress backend Service name and port match the Service definition.

Review ingress controller logs for upstream or backend errors.

Patch the Ingress backend to the correct port name or number.

**Final validation:** test the external URL and confirm HTTP 200 or expected response.

# 4. Helm

**Memory hook:** Helm is a package manager for Kubernetes.

**Chart:** package that contains Kubernetes templates.

**Values:** input variables used to render templates.

**Release:** installed instance of a chart in a namespace.

**Upgrade:** apply a new version or new values.

**Rollback:** return to a previous working revision.

**Never forget:** Helm does not magically fix bad Kubernetes YAML; it only renders and applies manifests.

**Production thought:** always use dry-run, debug, diff, versioning, and rollback planning before production upgrade.

**Most common risk:** changing immutable fields, wrong values, failed hooks, missing dependencies, or environment-specific override mistakes.

## Complex Scenario: Helm Upgrade Fails Because StatefulSet VolumeClaimTemplate Is Immutable

**Problem statement:** Helm upgrade fails for a database StatefulSet.

**Why it happened:** the new chart changed volumeClaimTemplates or another immutable StatefulSet field.

**Impact:** upgrade fails, release becomes unhealthy, and storage changes cannot be applied directly.

**Simple solution:** rollback if needed, compare rendered manifests, avoid immutable changes, or plan a controlled migration.

**Approach to handle:** check Helm history, render manifests, compare with live object, identify immutable field, then rollback or migrate.

Run **helm history \<release-name> -n \<namespace>** to find failed and last good revision.

Run **helm upgrade --dry-run --debug** or **helm template** to inspect rendered YAML.

Run **kubectl get statefulset \<name> -n \<namespace> -o yaml** to compare live configuration.

Identify changes in immutable fields such as volumeClaimTemplates, selector, or serviceName.

If storage size is the only change, expand the PVC directly if supported by the StorageClass.

If template structure must change, take backup and plan new StatefulSet migration.

If production is impacted, run **helm rollback \<release-name> \<revision> -n \<namespace>**.

**Final validation:** confirm Pods are healthy, PVCs are attached, and application data is consistent.

# 5. Quick Troubleshooting Checklist

**For PVC/PV issues:** check PVC status, PV binding, StorageClass, access mode, reclaim policy, node affinity, and storage backend health.

**For scheduling issues:** check Pod events, node labels, taints, tolerations, resource requests, affinity rules, anti-affinity rules, and topology spread constraints.

**For networking issues:** check Pod readiness, Service selectors, endpoints, DNS, NetworkPolicy, ingress rules, TLS, and ingress controller logs.

**For Helm issues:** check rendered manifests, release history, values files, immutable fields, failed hooks, dependencies, and rollback options.
