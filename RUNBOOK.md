# Operational Runbook: Robot Shop Web and Payment Recovery

**Platform:** Stan's Robot Shop on AWS EKS  
**Namespace:** `robot-shop`  
**Monitoring:** Prometheus, Grafana, Alertmanager, Fluent Bit, Loki and Jaeger  
**Notification Channel:** Slack `#monitoring`  
**Author:** Vince – DevOps SRE  
**Status:** Active  
**Last Updated:** November 5, 2026

---

## 1. Purpose and Scope

This runbook defines the standard operating procedures for detecting, investigating, mitigating and recovering availability incidents affecting the Robot Shop Web Frontend and Payment Service.

It covers:

- Web frontend availability failures.
- Payment service availability failures.
- Invalid container image deployment scenarios.
- Kubernetes-based recovery using `kubectl`.
- Monitoring and recovery verification.
- Incident escalation and post-incident documentation.

The objective is to restore service availability quickly, minimize customer impact and maintain the platform's defined Service Level Objectives (SLOs).

## 2. Quick Reference Alert Table

| Alert Name | Severity | SLO Target | Target Service |
|---|---|---|---|
| `RobotShopWebAvailabilitySLOBreached` | Critical (P1) | 99.0% | `web` |
| `RobotShopPaymentAvailabilitySLOBreached` | Critical (P1) | 99.5% | `payment` |

---

## 3. Alert: RobotShopWebAvailabilitySLOBreached

### 3.1 Overview and Severity

- **Severity:** Critical (P1)
- **Target Component:** `deployment/web`
- **Namespace:** `robot-shop`
- **SLO Target:** 99.0% availability.
- **Notification Channel:** Slack `#monitoring`

**Alert Summary:** Web Ingress Service SLO Error Budget Breached.

**Alert Details:** Web frontend ingress service is unavailable or unready in the `robot-shop` namespace.

**Impact:** Users may be unable to access the Robot Shop storefront.

### 3.2 Step 1: Initial Triage

When the availability alert fires in Slack, inspect the frontend pods:

```bash
kubectl get pods -n robot-shop -l app=web
```

Check deployment health:

```bash
kubectl get deployment web -n robot-shop
```

Inspect recent Kubernetes events:

```bash
kubectl get events -n robot-shop --sort-by=.lastTimestamp
```

### 3.3 Step 2: Diagnostic Inspection

Identify the pod failure state:

| Status | Possible Cause |
|---|---|
| `ErrImagePull` | Invalid image reference or registry access failure |
| `ImagePullBackOff` | Repeated container image pull failure |
| `CrashLoopBackOff` | Application repeatedly crashing |
| `Pending` | Scheduling or resource constraint |

Inspect application logs:

```bash
kubectl logs deployment/web -n robot-shop --tail=50
```

Search frontend errors in Loki through Grafana Explore:

```logql
{service_name="web"} |= "Error"
```

### 3.4 Step 3: Mitigation – Invalid Image Deployment

If the frontend pods are stuck in `ErrImagePull` or `ImagePullBackOff` because of an invalid image tag, inspect the current image:

```bash
kubectl get deployment web -n robot-shop \
  -o jsonpath='{.spec.template.spec.containers[*].image}{"\n"}'
```

Restore the verified production image:

```bash
kubectl set image deployment/web \
  web=robotshop/rs-web:latest \
  -n robot-shop
```

This is the recovery procedure demonstrated during the Web frontend incident drill.

### 3.5 Step 4: Recovery Verification

Monitor the rollout:

```bash
kubectl rollout status deployment/web -n robot-shop
```

Verify pod health:

```bash
kubectl get pods -n robot-shop
```

Check frontend service:

```bash
kubectl get svc web -n robot-shop
```

Test application availability using the configured frontend endpoint:

```bash
curl -I --max-time 10 http://<WEB-ENDPOINT>:8080/
```

Expected result: successful HTTP response, normally `200 OK`.

Finally:

1. Confirm frontend pods are `1/1 READY` and `Running`.
2. Check Request Rate, Errors and Latency in Grafana.
3. Confirm the availability alert has returned to resolved.
4. Confirm the Slack `#monitoring` resolution notification.

---

## 4. Alert: RobotShopPaymentAvailabilitySLOBreached

### 4.1 Overview and Severity

- **Severity:** Critical (P1)
- **Target Component:** `deployment/payment`
- **Namespace:** `robot-shop`
- **SLO Target:** 99.5% availability.
- **Notification Channel:** Slack `#monitoring`

**Alert Summary:** Payment Service SLO Error Budget Breached.

**Alert Details:** Payment service is failing readiness or is unavailable in the `robot-shop` namespace.

**Impact:** Customers may be unable to complete checkout or process payments.

### 4.2 Step 1: Initial Triage

Inspect payment pod health:

```bash
kubectl get pods -n robot-shop
```

Check payment deployment:

```bash
kubectl get deployment payment -n robot-shop
```

Inspect the current payment image:

```bash
kubectl get deployment payment -n robot-shop \
  -o jsonpath='{.spec.template.spec.containers[*].image}{"\n"}'
```

### 4.3 Step 2: Diagnostic Inspection

Inspect payment application logs:

```bash
kubectl logs deployment/payment -n robot-shop --tail=50
```

Inspect deployment events:

```bash
kubectl describe deployment payment -n robot-shop
```

Search payment errors in Loki using Grafana Explore:

```logql
{service_name="payment"} |= "Error"
```

Use Jaeger to investigate failed or slow payment traces where available.

### 4.4 Step 3: Mitigation – Invalid Image Deployment

If the payment pods enter `ErrImagePull` or `ImagePullBackOff` because of an invalid image tag, restore the verified production image:

```bash
kubectl set image deployment/payment \
  payment=robotshop/rs-payment:latest \
  -n robot-shop
```

This is the recovery procedure demonstrated during the Payment service incident drill.

### 4.5 Step 4: Recovery Verification

Monitor the deployment rollout:

```bash
kubectl rollout status deployment/payment -n robot-shop
```

Verify pod health:

```bash
kubectl get pods -n robot-shop
```

Check payment service and endpoints:

```bash
kubectl get svc payment -n robot-shop
kubectl get endpoints payment -n robot-shop
```

Verify payment functionality using a valid test transaction and the application's actual API contract.

Finally:

1. Confirm the payment pod is `1/1 READY` and `Running`.
2. Check payment errors and latency in Grafana.
3. Confirm the availability alert has returned to resolved.
4. Confirm the Slack `#monitoring` resolution notification.

---

## 5. Monitoring and Observability

The Robot Shop platform uses a centralized monitoring and observability stack.

### 5.1 Prometheus and Alertmanager

Prometheus collects application metrics and evaluates alerting rules.

Alertmanager delivers notifications to Slack `#monitoring`.

Use these tools to:

- Inspect firing and resolved alerts.
- Verify availability alert conditions.
- Investigate metric collection issues.
- Monitor SLO consumption.

### 5.2 Grafana

Grafana provides dashboards for application and infrastructure monitoring.

The custom RED dashboard covers:

- **Rate:** Application request rate.
- **Errors:** Application error measurements.
- **Duration:** Application response latency.

The instructor-provided Robot Shop dashboard provides complementary infrastructure metrics, including network throughput, container restarts, CPU utilization and memory usage.

### 5.3 Fluent Bit and Loki

Fluent Bit collects Kubernetes container logs and forwards them to Loki.

Use Grafana Explore to search application errors.

Web query:

```logql
{service_name="web"} |= "Error"
```

Payment query:

```logql
{service_name="payment"} |= "Error"
```

### 5.4 OpenTelemetry and Jaeger

Jaeger provides distributed tracing for instrumented application requests.

Use traces to:

- Investigate failed requests.
- Identify slow service dependencies.
- Examine payment request execution.
- Trace errors across application services.

---

## 6. Escalation and Communication

| Situation | Escalation |
|---|---|
| Web frontend outage | Primary On-Call SRE |
| Payment service outage | Primary On-Call SRE / Application Team |
| Persistent application crash | Application Development Team |
| Dependency failure | Platform / Application Engineering |
| Monitoring or alerting failure | Monitoring / Platform Engineering |

During escalation, provide:

- Incident start time.
- Affected service.
- Customer impact.
- Relevant Kubernetes events and logs.
- Mitigation steps already attempted.
- Current recovery status.

Use Slack `#monitoring` for incident notifications and recovery updates.

---

## 7. Post-Incident Activities

After recovery:

1. Record incident start and resolution times.
2. Document customer impact.
3. Preserve relevant Kubernetes events and application logs.
4. Identify root cause and contributing factors.
5. Document mitigation and recovery actions.
6. Create corrective and preventative actions.
7. Update `postmortem.md`.
8. Track outstanding actions to completion.

Postmortems must remain blameless and focus on improving system reliability and preventing recurrence.

**Document Owner:** Vince – DevOps SRE  
**Platform:** Stan's Robot Shop on AWS EKS  
**File:** `RUNBOOK.md`
