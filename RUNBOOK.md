
Runbook: Robot Shop Web Frontend Recovery

Alert Name: RobotShopWebAvailabilitySLOBreached
Severity: Critical (P1)
Target Component: deployment/web in namespace robot-shop

1. Initial Triage & Diagnostics

When the availability alert fires in Slack, immediately check the health of the frontend pods using your terminal:
bash
kubectl get pods -n robot-shop -l app=web
Use code with caution.

Identify the Problem State:

• If status shows ErrImagePull or ImagePullBackOff: An invalid container image tag was deployed. Proceed to Step 2 (Rollback).
• If status shows CrashLoopBackOff: The container is crashing on startup. Check application logs via Loki or run:bash
kubectl logs deployment/web -n robot-shop --tail=50
Use code with caution.

2. Immediate Mitigation (Emergency Rollback)

If the pods are failing to pull due to a bad image tag (e.g., bad-image), immediately force a rollback to the stable production release tag (latest) to restore service availability:
bash
kubectl set image deployment/web web=web-robotshop/rs-web:latest -n robot-shop
Use code with caution.

3. Verify Deployment Recovery

Monitor the rolling update to ensure the healthy pods are completely spun up and the broken ones are terminated:
bash
kubectl rollout status deployment/web -n robot-shop
Use code with caution.
Verify that all active pods return to a 1/1 READY and Running status:
bash
kubectl get pods -n robot-shop
Use code with caution.

4. Verify System SLO in Grafana

1. Open your Grafana dashboard.
2. Check the Request Rate and Errors panels.
3. Confirm that the error lines drop back to 0 and traffic returns to normal.
4. Verify that the #monitoring channel in Slack receives a green [RESOLVED] notification.
