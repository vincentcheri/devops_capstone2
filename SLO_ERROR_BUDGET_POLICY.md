
Robot Shop Service Level Objective (SLO) & Error Budget Policy

Document Status: ACTIVE
Target System: Stan's Robot Shop Microservices Application
Owner: DevOps Site Reliability Engineering (SRE) Team

1. Service Level Indicator (SLI) & Objective (SLO) Definition

To guarantee an acceptable user experience across the web storefront, the platform establishes a core Availability target based on inbound web ingress traffic.
• Service Level Indicator (SLI): The proportion of valid HTTP requests to the Robot Shop web gateway service that return a success status code (any HTTP status code less than 500).
\(\text{SLI}=\frac{\sum \text{Requests\ (Status\ Code\ }<500\text{)}}{\sum \text{Total\ Valid\ Inbound\ Requests}}\)
• Service Level Objective (SLO): 99.0% availability over a rolling 30-day window.
• Total Allowed Error Budget: 1.0% of total requests over the rolling window can fail before breaching the agreement.

2. Alerting Thresholds & Slack Notification Routing

When abnormal failure spikes burn through the monthly allocation prematurely, multi-window multi-burn-rate alerts handle routing via Prometheus Alertmanager to the #monitoring Slack workspace:
• Critical Burn Warning: Alert triggers if >2% of the total error budget is burned within any 1-hour window (indicating a catastrophic outage).
• Slack Escalation: Triggers the critical webhook payload containing the specific alert payload link to the production web console.

3. Error Budget Depletion Policy (The Escalation Rules)

If an ongoing incident or frequent minor instability consumes the error budget, the following strict engineering rules apply based on the percentage consumed:

🟢 State 1: <50% Budget Consumed (Normal Operations)

• Engineering Focus: Feature velocity remains the primary focus.
• Action Plan: Code shipments to production continue as scheduled. Normal bug-triage workflows apply.

🟡 State 2: 50% - 89% Budget Consumed (Warning Threshold)

• Engineering Focus: Reliability review.
• Action Plan: The SRE team schedules a joint review meeting with the Development team. Automated end-to-end integration tests are evaluated to identify flaky microservices (e.g., payment or cart).

🔴 State 3: ≥90% Budget Consumed (Near Breach)

• Engineering Focus: Code Freeze & Recovery Sprint.
• Action Plan: Feature development freezes immediately. All active pipelines are restricted except for emergency reliability hotfixes, performance tune-ups, or security updates.

❌ State 4: Budget Fully Exhausted (100% Breached)

• Engineering Focus: Mandatory Postmortem & Remediation.
• Action Plan: A cross-functional engineering freeze remains in place. A formal Blameless Postmortem must be logged within 48 hours of resolution. Reliability items identified during the root cause analysis are added to the immediate sprint backlog and must be resolved before regular feature deployments can resume.
