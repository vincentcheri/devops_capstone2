# Blameless Postmortem: Incident #1042 – Robot Shop Web Storefront Outage

| Field | Details |
|---|---|
| **Incident ID** | 1042 |
| **Date** | November 5, 2026 |
| **Service** | Robot Shop Web Storefront |
| **Severity** | To be confirmed |
| **Status** | RESOLVED |
| **Incident Duration** | 58 seconds (06:50:00–06:50:58 UTC) |
| **Customer-Impact Duration** | 18 seconds (reported) |
| **Author** | Vince – DevOps SRE |

---

## 1. Executive Summary

On November 5, 2026, the Robot Shop Web Storefront experienced a temporary service outage following a deployment configuration change that introduced an invalid container image reference.

The `web` deployment was updated to use an image tag that did not exist:

`web-robotshop/rs-web:bad-image`

Kubernetes was consequently unable to retrieve the specified container image. The affected replacement pods entered `ErrImagePull` and `ImagePullBackOff` states, preventing the deployment from successfully completing its rollout.

The incident was detected by Prometheus Alertmanager through the `RobotShopWebAvailabilitySLOBreached` alert, which delivered a notification to the monitoring Slack channel.

Service was restored by replacing the invalid image reference with the previously verified image tag:

`web-robotshop/rs-web:latest`

The deployment subsequently completed its rollout, and pod health checks confirmed recovery.

**Customer impact:** The storefront was unavailable for a reported 18 seconds. The full incident response timeline covered 58 seconds, from the initial configuration change to final recovery verification.

No evidence of data loss was provided as part of this incident review.

## 2. Customer Impact

- **Affected component:** Robot Shop Web Storefront (`web`).
- **User experience:** Users were temporarily unable to access the storefront.
- **Service availability:** A temporary availability degradation triggered the configured SLO alert.
- **Data integrity:** No data loss was identified or reported.
- **Duration:** 18 seconds of reported customer impact.

The incident was isolated to the web frontend deployment. The available evidence does not establish an impact on the underlying database or other backend services.

## 3. Incident Timeline

All timestamps are in UTC.

| Time | Event |
|---|---|
| 06:50:00 | Scheduled configuration updates were executed on the Kubernetes cluster. |
| 06:50:10 | The `web` deployment was updated with the invalid image tag `bad-image`. |
| 06:50:20 | Prometheus Alertmanager triggered `RobotShopWebAvailabilitySLOBreached` and delivered a Slack notification. |
| 06:50:28 | Investigation using Kubernetes pod status identified `ErrImagePull` and `ImagePullBackOff` states. |
| 06:50:40 | Mitigation began by restoring the deployment to the verified `latest` image tag. |
| 06:50:45 | Kubernetes rollout status confirmed successful deployment progression. |
| 06:50:58 | Final pod health verification confirmed recovery and the Slack alert returned to a resolved state. |

### Commands Used During Response

**Failure injection / deployment modification**

```bash
kubectl set image deployment/web \
  web=web-robotshop/rs-web:bad-image \
  -n robot-shop
```

**Investigation**

```bash
kubectl get pods -n robot-shop -w
```

**Service mitigation**

```bash
kubectl set image deployment/web \
  web=web-robotshop/rs-web:latest \
  -n robot-shop
```

**Recovery verification**

```bash
kubectl rollout status deployment/web -n robot-shop
kubectl get pods -n robot-shop
```

## 4. Root Cause Analysis

### Primary Root Cause

The incident was caused by an invalid container image reference being applied to the active Kubernetes deployment.

The deployment referenced `web-robotshop/rs-web:bad-image`, an image tag that was unavailable in the container registry.

Kubernetes could not retrieve the specified image, resulting in image-pull failures for replacement pods. This prevented the web deployment from successfully completing its rollout and caused temporary storefront unavailability.

### Contributing Factors

1. **Insufficient image validation:** The deployment process did not prevent an invalid image reference from reaching the active deployment.
2. **Mutable image reference:** The recovery process used the `latest` tag, which does not guarantee a reproducible deployment artifact.
3. **Admission policy gap:** No enforced admission policy was in place to validate image references against an approved standard before deployment.
4. **Deployment safeguards:** Additional pre-deployment checks could have detected the invalid image before it affected the running service.

### Detection

Prometheus Alertmanager detected the availability degradation through the configured `RobotShopWebAvailabilitySLOBreached` alert.

The alert was delivered to Slack, allowing the incident to be investigated and mitigated.

## 5. Resolution and Recovery

The immediate mitigation was to replace the invalid image reference with the previously verified image tag.

Kubernetes then completed the deployment rollout successfully.

Recovery was verified through:

- Successful `kubectl rollout status` output.
- Healthy and running application pods.
- Resolution of the availability alert in Slack.

The service was considered recovered after these checks completed.

## 6. Corrective and Preventative Actions

| Action | Description | Priority | Status |
|---|---|---|---|
| Immutable image enforcement | Require image digests or immutable build identifiers instead of mutable tags such as `latest`. | High | Planned |
| CI/CD image validation | Verify that container images exist and are accessible before deployment. | High | Planned |
| Admission control | Implement OPA policies to reject unapproved or invalid image references. | High | Planned |
| Deployment safety | Introduce deployment validation and health checks before completing a rollout. | High | Planned |
| Alert runbook integration | Attach the incident remediation runbook to relevant Alertmanager notifications. | Medium | Planned |
| Recovery documentation | Document the image rollback procedure and verification steps in the operational runbook. | Medium | Planned |

## 7. Lessons Learned

1. Deployment configuration changes must be validated before they are applied to active workloads.
2. Immutable image references improve deployment reproducibility and reduce ambiguity during recovery.
3. Monitoring and SLO-based alerting provide an effective mechanism for detecting service availability degradation.
4. Operational runbooks should be directly accessible from incident notifications to reduce response time.
5. Automated admission controls can prevent invalid deployment configurations from reaching production workloads.

## 8. Blameless Review

This incident resulted from a gap in deployment validation and enforcement rather than an individual failure.

The existing deployment process permitted an invalid image reference to reach an active workload without sufficient safeguards.

The corrective actions focus on strengthening CI/CD validation, enforcing image policies, improving deployment safety, and making incident response procedures more accessible.

The objective is to make this class of failure less likely and reduce its potential impact if it occurs again.

## 9. Final Incident Status

**Status: RESOLVED**

The invalid image reference was corrected, the Kubernetes deployment recovered successfully, and application pod health was verified.

The incident's immediate impact was mitigated. Preventative actions remain to be implemented and tracked to completion.

---

**Postmortem Owner:** Vince – DevOps SRE  
**Review Date:** November 5, 2026
