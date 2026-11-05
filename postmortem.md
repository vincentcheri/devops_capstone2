
Blameless Postmortem: Incident #1042 - Web Storefront Outage

Date: November 5, 2026
Author: Vince (DevOps SRE)
Status: RESOLVED
Duration of Incident: 18 seconds

1. Executive Summary

An unexpected deployment modification to the core user-facing frontend component (web) caused a complete storefront outage for users. The deployment configuration pointed to a non-existent container registry asset image (web-robotshop/rs-web:bad-image). This triggered a cluster-wide deployment block, throwing ErrImagePull states.
The issue was mitigated by manually triggering an image rollover fix back to a verified production asset tag (latest). Total downtime was restricted to 18 seconds with 100% recovery.

2. Timeline of Events (All times in UTC)

• 06:50:00 — Scheduled configuration updates executed on the cluster.
• 06:50:10 — SRE executes kubectl set image deployment/web web=web-robotshop/rs-web:bad-image -n robot-shop.
• 06:50:20 — Automated Slack alerts trigger via Prometheus Alertmanager indicating RobotShopWebAvailabilitySLOBreached.
• 06:50:28 — SRE runs triage with kubectl get pods -n robot-shop -w and isolates new pods stuck in ErrImagePull and ImagePullBackOff.
• 06:50:40 — Mitigation Step: Emergency rollback path executed: kubectl set image deployment/web web=web-robotshop/rs-web:latest -n robot-shop.
• 06:50:45 — Verification via kubectl rollout status deployment/web confirms successful rolling update path.
• 06:50:58 — Final confirmation check via kubectl get pods verifies all pods are healthy and running. Slack alerts return to a green status.

3. Root Cause Analysis (RCA)

The root cause was an invalid image tag string reference passed straight into the active deployment manifest. Kubernetes was unable to authenticate or pull the target layers from the cloud registry. This resulted in an immediate replica termination sequence without viable replacement infrastructure.

4. Corrective & Preventative Actions

To ensure this class of human or pipeline error cannot crash production again, the following infrastructure rules will be implemented:
1. Image Tag Enforcement: Modify continuous integration rules to forbid mutable or untagged descriptors (such as bad-image). All code changes must reference immutable semantic hashes or exact build IDs.
2. Admission Controllers: Implement an open-policy agent web-hook to inspect deployment manifests and reject image alterations that don't match verified pattern standards.
3. Runbook Automation: Attach an absolute remediation runbook link directly inside the Alertmanager configuration to fast-track tier-1 on-call mitigation response.
