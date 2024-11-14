# SRE-submission
To ensure seamless performance and proactive alerting, we’ll set up robust monitoring for the external API at https://api.nasa.gov/ using Prometheus and Grafana. This comprehensive guide will walk you through each step to configure Prometheus for effective API tracking, visualize data insights on Grafana dashboards, and receive real-time alerts directly in your designated Slack channel, ensuring you stay informed on any critical events or disruptions. We will use helm to deploy infrastructure on Azure AKS cluster

# 1.First Step 
we need to install prometheus/grafana with Alertmanager and Prometheus blackbox exporter in order to take metrics form external API
Add Prometheus Helm Repository
Start by adding the Prometheus Community Helm repository to your environment:

```bash
# Add the Prometheus Helm repository
helm repo add prometheus-community https://prometheus-community.github.io/helm-charts
helm repo update  # Update your Helm repository cache

# install all three components (Prometheus , Grafana , Blackbox exporter)
helm upgrade --install --atomic --wait --timeout=120s prometheus prometheus-community/prometheus -n test -f values.yaml
helm install grafana grafana/grafana -n test
helm install blackbox-exporter prometheus-community/prometheus-blackbox-exporter --namespace test



#2. Second Step
After installing all three components we need to add Prometheuse as a datasource in Grafana

![DataSource prometheus](https://github.com/user-attachments/assets/05987998-1d24-4eff-80b5-6d9c12af63ef)
