# SRE-submission
To ensure seamless performance and proactive alerting, we’ll set up robust monitoring for the external API at https://api.nasa.gov/ using Prometheus and Grafana. This comprehensive guide will walk you through each step to configure Prometheus for effective API tracking, visualize data insights on Grafana dashboards, and receive real-time alerts directly in your designated Slack channel, ensuring you stay informed on any critical events or disruptions.

# 1.First Step 
we need to install prometheus/grafana with Alertmanager and Prometheus blackbox exporter in order to take metrics form external API
Add Prometheus Helm Repository
Start by adding the Prometheus Community Helm repository to your environment:

```bash
# Add the Prometheus Helm repository
helm repo add prometheus-community https://prometheus-community.github.io/helm-charts
helm repo update  # Update your Helm repository cache
