# Senior SRE Scenario-Based Interview Questions and Answers

> **Purpose:** Practical preparation for senior SRE interviews at Google-style and other product engineering companies.
>
> **Important note:** These are original practice scenarios modeled on commonly assessed SRE areas. They are **not claimed to be leaked or verbatim questions** from any company.

---

## How to Answer Any Senior SRE Scenario

Remember this order:

> **IMPACT → STABILIZE → INVESTIGATE → FIX → VERIFY → PREVENT**

1. **Impact:** Who and what is affected? Is an SLO at risk?
2. **Stabilize:** Roll back, fail over, scale, throttle, or disable a feature.
3. **Investigate:** Use metrics, logs, traces, events, and recent changes.
4. **Fix:** Apply the smallest safe correction.
5. **Verify:** Confirm recovery from the user's point of view.
6. **Prevent:** Write a blameless postmortem and automate prevention.

### Memory Line

> **Stop the bleeding before studying the blood.** Restore service first, then perform deep root-cause analysis.

---

# 1. Production Incident and Troubleshooting

## Q1. A critical API suddenly returns 5xx errors. What will you do?

### Simple answer

1. Confirm customer impact, affected regions, endpoints, and error rate.
2. Check the four golden signals: latency, traffic, errors, and saturation.
3. Compare the incident start time with deployments, configuration changes, certificate changes, and traffic spikes.
4. Inspect application logs and distributed traces.
5. Check dependencies such as databases, caches, queues, DNS, and external APIs.
6. If a recent change is suspicious, roll it back or disable it with a feature flag.
7. Confirm recovery using synthetic tests and real-user metrics.
8. Complete a blameless postmortem and prevention actions.

### Senior-level point

Do not spend 30 minutes proving the root cause while customers are affected. Mitigate safely first.

### Remember

> **5xx = Change, Capacity, Code, or Dependency.**

---

## Q2. Production is down, but every pod and health check is green. How is that possible?

### Simple answer

A green pod only proves that the container is running and its configured health endpoint is passing. It does not prove that the complete user journey works.

Check:

- DNS resolution and load-balancer routing
- Ingress, service, and endpoint mapping
- Authentication and authorization
- Database or downstream connectivity
- Certificate validity
- Feature flags and configuration
- A real transaction through black-box or synthetic monitoring

### Senior-level point

Health checks should test meaningful readiness. A `/health` endpoint that always returns `200` creates false confidence.

### Remember

> **Alive is not the same as useful.**

---

## Q3. API latency is high, but CPU and memory are normal. What will you investigate?

### Simple answer

Check whether the application is waiting rather than computing:

- Database query latency and locks
- Connection-pool or thread-pool exhaustion
- External API latency
- Queue depth and consumer lag
- Disk or network I/O
- DNS delays
- Garbage-collection pauses
- Lock contention
- Retry storms

Use traces to find which span consumes most of the request time. Compare p50, p95, and p99 because averages can hide slow users.

### Remember

> **Normal CPU can mean the service is waiting.**

---

## Q4. Error rate increased immediately after deployment, but Kubernetes reports all pods healthy. What will you do?

### Simple answer

1. Compare the exact deployment time with the error increase.
2. Separate metrics by application version, pod, zone, and endpoint.
3. Check deployment events, logs, traces, configuration, secrets, and feature flags.
4. Compare canary and stable versions.
5. Roll back if customer impact is material and the previous version is known good.
6. Verify the rollback using service-level and user-facing signals.

### Senior-level point

A health check may only confirm that the process started. It may not detect a broken business operation.

### Remember

> **Healthy process, unhealthy product.**

---

## Q5. Only 5% of users are failing. How will you locate the issue?

### Simple answer

Slice telemetry by dimensions:

- Region or availability zone
- Application version
- Device/browser
- Tenant or customer tier
- Load-balancer target
- Database shard
- Kubernetes node or pod
- Feature-flag cohort

Look for one dimension that matches the affected 5%. If one cell, shard, or canary is failing, isolate it and route traffic away.

### Remember

> **Small percentage often means one slice is bad.**

---

## Q6. A dependency is slow and your service begins failing. How do you stop a cascading failure?

### Simple answer

Use:

- Strict timeouts
- Limited retries with exponential backoff and jitter
- Circuit breakers
- Concurrency limits and bounded queues
- Load shedding
- Cached or degraded responses
- Bulkheads to isolate resource pools

Do not retry every failure instantly. That multiplies traffic and makes the dependency weaker.

### Remember

> **Timeout, back off, break the circuit, degrade gracefully.**

---

# 2. SLOs, Error Budgets, and Monitoring

## Q7. A service has a 99.9% monthly availability SLO. How much downtime is allowed in 30 days?

### Simple answer

- Total minutes: `30 × 24 × 60 = 43,200`
- Failure allowance: `100% - 99.9% = 0.1%`
- Error budget: `43,200 × 0.001 = 43.2 minutes`

So the service can be unavailable for approximately **43.2 minutes** during that 30-day window before missing the SLO.

### Remember

> **Three nines means roughly 43 minutes per 30 days.**

---

## Q8. The error budget is exhausted halfway through the month. Should releases stop?

### Simple answer

First confirm that the SLI and measurement are correct. Then:

1. Pause or restrict risky changes according to the agreed error-budget policy.
2. Permit emergency, security, and reliability fixes.
3. Identify which failure modes consumed the budget.
4. Prioritize corrective engineering work.
5. Resume normal release velocity when risk is controlled under the policy.

### Senior-level point

An error budget is a decision tool, not a punishment. It balances reliability and delivery speed.

### Remember

> **Spend the budget on innovation; repay it with reliability.**

---

## Q9. Which alerts should wake an engineer at 2 AM?

### Simple answer

Page only when all three are true:

1. Users are affected or an SLO is burning quickly.
2. Immediate human action is required.
3. A useful action can be taken now.

Create a ticket or dashboard notification for slow, non-urgent conditions.

### Senior-level point

Prefer symptom-based and multi-window burn-rate alerts. Avoid paging only because CPU crossed an arbitrary threshold.

### Remember

> **Page on pain, not on every signal.**

---

## Q10. Your team receives hundreds of alerts every day. How will you reduce alert fatigue?

### Simple answer

- Remove alerts that require no action
- Deduplicate correlated alerts
- Route alerts to the owning team
- Use SLO and user-impact signals
- Tune thresholds and evaluation windows
- Suppress expected maintenance noise
- Add dependency awareness
- Track noisy alerts as engineering work
- Ensure every page has a runbook and owner

### Remember

> **Every alert must have an owner and an action.**

---

## Q11. Average latency is acceptable, but customers still complain. Why?

### Simple answer

The average hides the slow tail. For example, most requests may be fast while a small but important percentage is extremely slow.

Review:

- p50 for the typical request
- p95 for the slower user experience
- p99 for tail latency
- Distribution by endpoint, region, and customer

### Remember

> **Customers experience requests, not averages.**

---

# 3. Kubernetes and Container Reliability

## Q12. Pods are in `CrashLoopBackOff`. How will you troubleshoot?

### Simple answer

1. Run `kubectl describe pod` and study events.
2. Check current and previous container logs.
3. Inspect exit code, command, arguments, and startup behavior.
4. Check configuration, secrets, mounted volumes, and permissions.
5. Review startup, readiness, and liveness probes.
6. Check `OOMKilled`, CPU limits, and dependency availability.
7. Compare with the last working deployment.

### Remember

> **Events tell what Kubernetes saw; logs tell what the application felt.**

---

## Q13. A Kubernetes deployment caused failures. How do you roll it back safely?

### Simple answer

- Stop further rollout.
- Check whether the previous image and configuration are compatible.
- Roll back the deployment or shift traffic to the stable version.
- Be careful with irreversible database schema changes.
- Verify readiness, error rate, latency, and a business transaction.

For prevention, use canary or blue-green delivery, automated analysis, feature flags, and backward-compatible database migrations.

### Remember

> **Application rollback is easy; data rollback is hard.**

---

## Q14. One Kubernetes node is unhealthy. What should happen?

### Simple answer

1. Cordon the node so no new pods are scheduled there.
2. Confirm whether workloads have enough replicas and disruption budget.
3. Drain the node safely.
4. Allow workloads to reschedule on healthy nodes.
5. Investigate node conditions, kubelet, disk, network, runtime, and recent changes.
6. Replace rather than manually repair an immutable worker when appropriate.

### Remember

> **Cordon stops new work; drain moves existing work.**

---

## Q15. An EKS pod cannot access an S3 bucket. How will you troubleshoot?

### Simple answer

First identify the error type:

- **Access denied:** Check the pod's service account, IRSA association, IAM trust policy, permissions policy, bucket policy, encryption-key permissions, and object path.
- **Timeout or connection error:** Check DNS, route tables, NAT or S3 VPC endpoint, security controls, network policies, and proxy settings.

Test from the affected pod using its actual identity. Do not give broad permissions just to make the error disappear.

### Remember

> **403 means identity/policy; timeout means path/network.**

---

## Q16. How will you perform a zero-downtime Kubernetes deployment?

### Simple answer

- Run multiple replicas across failure domains.
- Configure accurate readiness and startup probes.
- Use rolling update, canary, or blue-green deployment.
- Set `maxUnavailable` and `maxSurge` appropriately.
- Handle graceful termination and connection draining.
- Use PodDisruptionBudgets carefully.
- Keep database changes backward compatible.
- Automatically stop or roll back on bad service-level metrics.

### Remember

> **Ready before traffic; drain before death.**

---

# 4. Linux, Networking, and Performance

## Q17. A Linux server has high load average but low CPU utilization. What does it mean?

### Simple answer

Load average includes tasks waiting to run and tasks stuck in uninterruptible I/O sleep. Low CPU with high load often points to disk, NFS, or another I/O wait problem.

Check:

- `uptime` or `top`
- `vmstat`
- `iostat`
- Process states
- Disk latency and queue depth
- NFS/storage health

### Remember

> **High load does not always mean high CPU.**

---

## Q18. Disk usage is 100%, but large files are not visible. What could be wrong?

### Simple answer

A process may still hold a deleted file open. The filename disappears, but space is not released until the process closes the file descriptor.

Check open deleted files with `lsof +L1` or an equivalent command. Restart or safely signal the owning process after evaluating impact.

Also check inode exhaustion, hidden mount points, container layers, and logs.

### Remember

> **Deleted is not released while a process still holds it.**

---

## Q19. Users see intermittent connection timeouts. How will you debug the network path?

### Simple answer

Follow the path layer by layer:

1. DNS resolution
2. Client-to-load-balancer reachability
3. Load-balancer health and target routing
4. Firewall/security rules
5. Ingress and service endpoints
6. Pod or host listening port
7. Connection pools, NAT/SNAT, and ephemeral ports
8. Packet loss, retransmission, and MTU issues

Compare a successful and failed request by region, target, and time.

### Remember

> **DNS → Route → Rule → Listener → Application.**

---

## Q20. TLS works for some clients but fails for others. What will you check?

### Simple answer

Check:

- Complete certificate chain
- Expiry and hostname/SAN match
- Supported TLS versions and cipher suites
- SNI behavior
- Client trust store
- Certificate rotation consistency across load balancers
- System clock

### Remember

> **Name, chain, time, protocol.**

---

# 5. Cloud, Capacity, and Distributed Systems

## Q21. Design a highly available service across regions. What trade-offs will you discuss?

### Simple answer

Cover:

- Active-active versus active-passive
- Global traffic routing and health checks
- Independent failure cells to reduce blast radius
- Data replication and consistency requirements
- RTO and RPO
- Capacity to survive a regional failure
- Dependency and control-plane failure
- Automated failover with tested manual override
- Regular disaster-recovery exercises

### Senior-level point

Do not say only “deploy in two regions.” Explain data consistency, failover safety, capacity, cost, and operational complexity.

### Remember

> **Compute can fail over quickly; data decides the real DR design.**

---

## Q22. Traffic suddenly becomes ten times normal. How will you protect the service?

### Simple answer

- Confirm whether traffic is legitimate, a retry storm, or abusive.
- Apply rate limiting and quotas.
- Autoscale only where dependencies can also handle the load.
- Shed low-priority work.
- Cache safe responses.
- Queue asynchronous work with bounded capacity.
- Protect the database using connection and concurrency limits.
- Serve a degraded but useful experience if necessary.

### Remember

> **Scale, limit, queue, shed, degrade.**

---

## Q23. Retries are making an outage worse. Why, and how do you fix it?

### Simple answer

If every layer retries, one failed request can become many requests. This amplifies load on a struggling dependency.

Fix it with:

- Retry only transient and safe operations
- Limit the number of attempts
- Use exponential backoff and jitter
- Define an overall deadline
- Retry at one appropriate layer
- Use idempotency keys for operations that may be repeated
- Combine retries with circuit breakers and load shedding

### Remember

> **A retry is extra traffic during failure.**

---

## Q24. How do you estimate capacity for a new service?

### Simple answer

1. Estimate peak requests per second, not only daily averages.
2. Measure resource cost per request.
3. Include storage growth, network, and dependency limits.
4. Add headroom for bursts and failure scenarios.
5. Load test to validate assumptions.
6. Define saturation signals and scaling thresholds.
7. Revisit the model using production data.

### Remember

> **Demand × cost per request + failure headroom.**

---

## Q25. A Kafka consumer is falling behind. How will you troubleshoot?

### Simple answer

Check:

- Consumer lag by partition
- Input rate versus processing rate
- Slow downstream systems
- Consumer errors and rebalances
- Hot or uneven partitions
- Batch size and processing efficiency
- Number of partitions and active consumers
- Offset commits and duplicate-processing behavior

Mitigate by removing the downstream bottleneck, increasing safe consumer parallelism, or temporarily reducing producer load. Adding consumers beyond the partition count does not increase active parallelism within one consumer group.

### Remember

> **Lag grows when production is faster than successful consumption.**

---

# 6. Infrastructure as Code and Delivery

## Q26. A Terraform plan wants to recreate a production database. What will you do?

### Simple answer

Stop the pipeline. Do not apply blindly.

Then:

- Identify which attribute forces replacement.
- Check configuration, provider version, state, imports, and drift.
- Back up the database and verify recovery options.
- Use lifecycle protection where appropriate.
- Separate high-risk stateful resources from routine changes.
- Peer review the plan and create a tested migration approach if replacement is truly required.

### Remember

> **Unexpected replacement means stop, understand, protect data.**

---

## Q27. Configuration drift is found in production. How will you handle it?

### Simple answer

1. Determine whether the manual change was an emergency correction or an unauthorized change.
2. Capture evidence and current state.
3. Decide the authoritative desired configuration.
4. Import or update code when the production change is valid, or reconcile production back to code when it is not.
5. Add drift detection, access controls, and break-glass auditing.

### Remember

> **One source of truth, with an audited emergency door.**

---

## Q28. A canary shows a small error increase. Do you continue the rollout?

### Simple answer

Do not decide by instinct alone. Compare canary and baseline using predefined criteria:

- Error rate and SLO burn
- p95/p99 latency
- Saturation
- Business success metrics
- Error type and affected users
- Statistical confidence and observation window

Pause or roll back if the agreed threshold is crossed. Continue only when risk is understood and within policy.

### Remember

> **A canary is useful only when it can stop the rollout.**

---

# 7. Incident Leadership and Reliability Culture

## Q29. During an outage, application and infrastructure teams blame each other. What will you do as incident commander?

### Simple answer

- Stop the blame discussion.
- Restate customer impact and incident priority.
- Assign clear roles: incident commander, technical leads, communications lead, and scribe.
- Build a shared timeline using evidence.
- Run investigations in parallel with named owners.
- Record decisions and the next update time.
- Resolve accountability later through a blameless review.

### Remember

> **During the incident: evidence and action. After the incident: learning.**

---

## Q30. What makes a strong blameless postmortem?

### Simple answer

Include:

- Customer and business impact
- Detection and response timeline
- Contributing technical and organizational conditions
- What worked and what failed
- Root cause and contributing factors
- Clear actions with owners and dates
- Priority based on recurrence risk and impact
- Follow-up tracking until closure

“Human error” is not a sufficient root cause. Ask why the system allowed one action to create that impact.

### Remember

> **Do not fix the person; fix the conditions.**

---

## Q31. How will you explain a major outage to executives?

### Simple answer

Use plain language and this structure:

1. What customers are experiencing
2. When it started and the current scope
3. What the team has done to reduce impact
4. Current status and known risks
5. Next update time

Do not guess the root cause or recovery time. Share confirmed facts and clearly label unknowns.

### Remember

> **Impact, action, status, next update.**

---

## Q32. Your team spends most of its time on repetitive operational work. What will you do?

### Simple answer

- Measure toil by type, frequency, effort, and risk.
- Remove work that provides no value.
- Standardize runbooks.
- Automate frequent, deterministic, safe tasks.
- Add approvals and rollback for risky automation.
- Track whether toil actually decreases.
- Protect engineering time for reliability improvements.

### Remember

> **Automate repetition, not confusion.**

---

# 8. Security and Safe Automation

## Q33. A critical container vulnerability is announced. What is your response?

### Simple answer

1. Confirm whether the vulnerable package and version are actually deployed.
2. Determine reachability, exposure, exploitability, and business impact.
3. Identify all affected images and workloads.
4. Apply compensating controls if an immediate patch is not possible.
5. Rebuild from a trusted patched base image and redeploy through the normal pipeline.
6. Verify remediation and check for indicators of compromise.
7. Improve image scanning, inventory, and patch SLAs.

### Remember

> **Find exposure, reduce risk, rebuild, verify.**

---

## Q34. An automated remediation script can restart production services. How do you make it safe?

### Simple answer

Build guardrails:

- Clear trigger conditions
- Read-only diagnosis before action
- Small blast radius and one target at a time
- Rate limits and concurrency limits
- Idempotency
- Maintenance and dependency checks
- Approval for high-risk actions
- Automatic rollback or stop conditions
- Audit logs and notifications
- Validation after remediation

### Remember

> **Automation needs brakes, not only an accelerator.**

---

# 9. Senior System Design Challenge

## Q35. Design a reliable global checkout service.

### Strong answer structure

#### 1. Clarify requirements

- Expected peak traffic and growth
- Availability and latency SLOs
- Data consistency requirements
- Payment-provider behavior
- RTO and RPO
- Compliance scope

#### 2. High-level design

- Global traffic management
- Regionally isolated application cells
- Stateless service replicas across zones
- Transactional order store
- Idempotency key for checkout requests
- Queue for asynchronous non-critical work
- Cache only where correctness is preserved

#### 3. Failure handling

- Timeouts and bounded retries
- Circuit breakers around payment and inventory services
- Saga or compensating actions for multi-step workflows
- Duplicate-request protection
- Regional failover and degraded mode

#### 4. Observability

- Checkout success-rate SLI
- End-to-end p95/p99 latency
- Payment decline versus technical failure
- Queue lag and saturation
- Traces across checkout, inventory, and payment

#### 5. Delivery and operations

- Canary releases
- Backward-compatible schemas
- Feature flags
- Capacity tests and failure drills
- Runbooks and incident ownership

### Remember

> **For money flows: consistency, idempotency, auditability, and safe recovery come first.**

---

# Rapid-Fire Follow-Up Questions

Use these to probe senior depth after each main answer:

1. What would you do in the first five minutes?
2. What evidence would prove your hypothesis?
3. What is the safest immediate mitigation?
4. What can go wrong with your mitigation?
5. What metric confirms complete recovery?
6. How would your approach change in a multi-region system?
7. What would you automate?
8. What would you never automate without approval?
9. How would you reduce the blast radius?
10. What permanent action prevents recurrence?

---

# Final Memory Sheet

| Concept | Never-forget line |
|---|---|
| Incident response | Impact → Stabilize → Investigate → Fix → Verify → Prevent |
| Golden signals | Latency, Traffic, Errors, Saturation |
| Alerting | Page on user pain that needs immediate action |
| Error budget | Reliability budget connecting SLOs to release risk |
| High latency | The service may be waiting, not computing |
| Retry | Extra traffic during failure |
| Cascading failure | Timeout, backoff, circuit break, shed load |
| Kubernetes health | Running does not mean serving correctly |
| Rollback | Code is easy; data is hard |
| Network debugging | DNS → Route → Rule → Listener → Application |
| Capacity | Peak demand × resource cost + failure headroom |
| Postmortem | Fix conditions, not people |
| Automation | Guardrails, small blast radius, verification, rollback |
| Senior mindset | Explain trade-offs, evidence, risk, and prevention |

---

# Suggested Interview Answer Template

Use this 60-to-90-second format:

> “First, I would confirm customer impact and check whether the SLO is at risk. My immediate goal would be to stabilize the service through the safest reversible action, such as rollback, failover, scaling, or disabling the affected feature. In parallel, I would compare metrics, logs, traces, and recent changes to isolate the failure domain. After applying the fix, I would verify recovery using both service metrics and a real user transaction. Finally, I would run a blameless postmortem and create owned prevention actions.”

---

# References

- [Google SRE Book: Service Level Objectives](https://sre.google/sre-book/service-level-objectives/)
- [Google SRE Book: Monitoring Distributed Systems](https://sre.google/sre-book/monitoring-distributed-systems/)
- [Google SRE Book: Embracing Risk](https://sre.google/sre-book/embracing-risk/)
- [Google SRE Book: Production Service Best Practices](https://sre.google/sre-book/service-best-practices/)
- Internal interview material was used to align the scenarios with practical senior SRE evaluation areas, including production troubleshooting, observability, Kubernetes, AWS, Terraform, incident leadership, and automation.
