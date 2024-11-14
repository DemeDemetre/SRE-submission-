## SRE-submission
To ensure seamless performance and proactive alerting, we’ll set up robust monitoring for the external API at https://api.nasa.gov/ using Prometheus and Grafana. This comprehensive guide will walk you through each step to configure Prometheus for effective API tracking, visualize data insights on Grafana dashboards, and receive real-time alerts directly in your designated Slack channel, ensuring you stay informed on any critical events or disruptions. We will use helm to deploy infrastructure on Azure AKS cluster

### 1.First Step 
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
```

![Deployed pods in KUbernetes](https://github.com/user-attachments/assets/6f840941-d25b-4d80-9021-1b9265138cfe)


### 2.Second Step
After installing all three components we need to add Prometheuse as a datasource in Grafana
![DataSource prometheus](https://github.com/user-attachments/assets/68ca9056-9025-4857-9bf3-43d4925eb1e5)

### 3. The third step
We need to add Prometheus alerting Rules in Helm Value for https://api.nasa.gov/planetary/apod?api_key=H2L2Xew086YQlXrkgIpA9ilmQAFJduFPgBMfmCwH API
```yaml
 #Prometheus rules:

 serverFiles:
  ## Alerts configuration
  ## Ref: https://prometheus.io/docs/prometheus/latest/configuration/alerting_rules/
  alerting_rules.yml:
    groups:
      - name: nasa_api_alerts
        rules:
         #1 Instance down 
          - alert: NASAApiDown
            expr: probe_success{job="nasa_api"} == 0
            for: 1m
            labels:
              severity: "critical"
            annotations:
              description: "NASA API has been unreachable for more than 1 minutes."
              summary: "NASA API is down"
          # 2. Alert for High Response Time
          - alert: NASAApiHighResponseTime
            expr: probe_duration_seconds{job="nasa_api"} > 1
            for: 1m
            labels:
              severity: "warning"
            annotations:
              summary: "High response time for NASA API"
              description: "NASA API response time is above 1 second for more than 1 minutes."
          # 3. Alert for High Error Rate
          - alert: NASAApiErrorRate
            expr: probe_success{job="nasa_api"} == 0
            for: 1m
            labels:
              severity: "critical"
            annotations:
              summary: "High Error Rate for NASA API"
              description: "The NASA API is currently returning errors. It has been unreachable for more than 1 minute."
          # 4. Alert for High Utilization of Prometheus or Blackbox Exporter (as proxy)
          - alert: HighCpuUsage
            expr: rate(container_cpu_usage_seconds_total{pod=~"prometheus.*|blackbox-exporter.*"}[1m]) > 0.8
            for: 1m
            labels:
              severity: "warning"
            annotations:
              summary: "High CPU Usage on Prometheus or Blackbox Exporter"
              description: "The CPU usage for Prometheus or Blackbox Exporter is above 80% for more than 1 minutes."

          - alert: HighMemoryUsage
            expr: container_memory_usage_bytes{pod=~"prometheus.*|blackbox-exporter.*"} / container_spec_memory_limit_bytes{pod=~"prometheus.*|blackbox-exporter.*"} > 0.8
            for: 5m
            labels:
              severity: "warning"
            annotations:
              summary: "High Memory Usage on Prometheus or Blackbox Exporter"
              description: "The memory usage for Prometheus or Blackbox Exporter is above 80% for more than 5 minutes."
```

![s](https://github.com/user-attachments/assets/32b5ce14-0a0d-4cdd-95e2-809a0574f4ea)

### 4. The fourth step
We need to add Prometheus Blackbox configuration in Helm value  in order to collect metrics from external API https://api.nasa.gov/planetary/apod?api_key=H2L2Xew086YQlXrkgIpA9ilmQAFJduFPgBMfmCwH
```yaml
#Prometheus  blackbox rule

extraScrapeConfigs: |
  - job_name: "nasa_api"
    metrics_path: /probe
    params:
      module: [http_2xx]
    static_configs:
      - targets:
          - https://api.nasa.gov/planetary/apod?api_key=H2L2Xew086YQlXrkgIpA9ilmQAFJduFPgBMfmCwH
    relabel_configs:
      - source_labels: [__address__]
        target_label: __param_target
      - source_labels: [__param_target]
        target_label: instance
      - target_label: __address__
        replacement: blackbox-exporter-prometheus-blackbox-exporter.test.svc.cluster.local:9115
```
### 5. The fifth step
Add alerting rules for alert manager in order to send notification to slack channel 
```yaml
#Alertmanager and Slack integration :

alertmanager:
  enabled: true
  config:
    global:
      resolve_timeout: 5m

    route:
      group_by: ["alertname"]
      group_wait: 30s
      group_interval: 5m
      repeat_interval: 3h
      receiver: "slack-notifications"

    receivers:
      - name: "slack-notifications"
        slack_configs:
          - send_resolved: true
            api_url: "https://hooks.slack.com/services/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"
            channel: "#prometheus-testing"
            username: "Prometheus Alert"
            title: "{{ .CommonAnnotations.summary }}"
            text: "{{ .CommonAnnotations.description }}"

    inhibit_rules:
      - source_match:
          severity: "critical"
        target_match:
          severity: "warning"
        equal: ["alertname", "job"]
```
![Alertmnager1](https://github.com/user-attachments/assets/44d6a84a-4bad-4729-8ab0-5871bcf2ec19)



