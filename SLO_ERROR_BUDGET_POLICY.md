# SLO ERROR BUDGET POLICY

## 1. Purpose

This policy defines the availability Service Level Objectives (SLOs) for the Stan's Robot Shop application deployed on AWS EKS. It establishes how service availability is measured, how much downtime is acceptable, and what actions should be taken when the error budget is consumed.

## 2. Service Level Objectives (SLOs)

The availability objectives are measured over a rolling 30-day window.

| Service | Availability SLO | Allowed Error Rate |
|---|---:|---:|
| Web | 99.0% | 1.0% |
| Payment | 99.5% | 0.5% |

### Service Level Indicators (SLIs)

Availability is measured using the proportion of successful requests compared with total valid requests.

**Availability formula:**

Availability (%) = (Successful Requests / Total Valid Requests) × 100

- Successful requests are requests completed successfully.
- Failed requests are requests that result in service errors or unsuccessful responses.
- Total valid requests include both successful and failed requests.

## 3. Error Budget Calculation

An error budget is the amount of unavailability permitted within the SLO measurement window.

A 30-day period contains 43,200 minutes.

| Service | Error Budget | Allowed Downtime |
|---|---:|---:|
| Web | 1.0% | 432 minutes |
| Payment | 0.5% | 216 minutes |

**Calculation:**

- Web: 43,200 × 0.01 = 432 minutes.
- Payment: 43,200 × 0.005 = 216 minutes.

When a service consumes its entire error budget, it has failed to meet its availability objective for that measurement period.

## 4. Error Budget Burn Rate

Burn rate measures how quickly a service consumes its available error budget.

| Burn Rate | Severity | Required Response |
|---|---|---|
| 1x | Normal | Continue normal operations and monitoring. |
| 6x | High | Investigate the cause and review recent deployments. |
| 14.4x | Critical | Prioritize immediate mitigation and consider stopping risky releases. |

A higher burn rate means the service is consuming its error budget faster than the allowed rate.

## 5. Alerting and Monitoring

Prometheus monitors the Robot Shop services and evaluates availability-related alert rules.

The following alerts are configured:

| Alert | Service | SLO |
|---|---|---:|
| RobotShopWebAvailabilitySLOBreached | Web | 99.0% |
| RobotShopPaymentAvailabilitySLOBreached | Payment | 99.5% |

The alerts detect when the corresponding service becomes unavailable or its pods become unready.

Alert notifications are delivered to the Slack `#monitoring` channel through Alertmanager.

The monitoring stack includes:

- **Prometheus:** Collects metrics and evaluates alert rules.
- **Grafana:** Displays service and infrastructure dashboards.
- **Loki and Fluent Bit:** Collect and provide searchable application logs.
- **Jaeger and OpenTelemetry:** Provide distributed tracing for request investigation.

## 6. Error Budget Policy

The following actions apply depending on the remaining error budget.

**Healthy error budget**
- Continue normal development and deployment activities.
- Monitor service availability and performance.
- Review dashboards and alerts regularly.

**Low error budget**
- Investigate recurring failures and service degradation.
- Review recent deployments and configuration changes.
- Prioritize reliability improvements.
- Avoid unnecessary risky changes.

**Exhausted error budget**
- Pause non-essential feature releases.
- Prioritize bug fixes and reliability improvements.
- Investigate the root cause of service failures.
- Resume normal releases after service stability improves and the team reviews the incident.

## 7. Incident Response

When an availability alert fires:

1. Acknowledge the alert in Slack.
2. Identify the affected service.
3. Check pod and deployment health using `kubectl`.
4. Review Grafana metrics and Loki logs.
5. Use Jaeger traces when request-level investigation is needed.
6. Mitigate the issue and restore service availability.
7. Verify that the deployment is healthy.
8. Document the root cause and corrective actions.

## 8. Review and Continuous Improvement

The SLOs and error budget should be reviewed regularly to ensure that they reflect the application's reliability requirements.

After an incident, the team should:

- Review the impact on service availability.
- Determine how much error budget was consumed.
- Identify the root cause.
- Document corrective and preventive actions.
- Update monitoring and alert rules where necessary.

## 9. Conclusion

This policy provides a framework for managing the availability of the Stan's Robot Shop application. By defining measurable availability objectives, calculating error budgets, monitoring service health, and establishing release policies, the team can balance feature development with system reliability.

The Web service has a 99.0% availability objective, while the Payment service has a stricter 99.5% availability objective. Prometheus, Grafana, Alertmanager, Loki, Fluent Bit, and Jaeger support monitoring, alerting, and incident investigation.
