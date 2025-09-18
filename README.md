## Deploying the OpenTelemetry Demo app on Grafana Cloud 
This repository is a small guide how to deploy the https://github.com/open-telemetry/opentelemetry-demo?tab=readme-ov-file#-opentelemetry-demo app to send data to Grafana Cloud using the Kubernetes Monitoring Helm Chart to be the OpenTelemetry Collector. 

The supplied values.yaml otelDemoValues.yaml makes only a couple small changes:

1. Disables components from being deployed: 
 - Otel Collector
- Prometheus
- Grafana
- Jaeger
- OpenSearch

Since all of these will be done by Alloy and Grafana Cloud

2. Updates the env variable OTEL_COLLECTOR_NAME value to grafana-k8s-monitoring-alloy-receiver.default.svc.cluster.local which is the default service endpoint for Alloy's OTLP Receiver in K8s.

## Steps to Deploy 

### 1. Infrastructure/K8s Observability
Easiest way to instrument the Infra monitoring component is to follow the wizard that is built into your [Cloud Account](https://grafana.com/docs/grafana-cloud/monitor-infrastructure/kubernetes-monitoring/configuration/helm-chart-config/#activate-and-send-data-from-your-account). 

The only non-default option you should change is to enable the "Application receivers" option. 

Take note of the OTLP/gRPC endpoint provided in step 6. 

Once the helm chart is deployed, it should start collecting Infra Metrics/Logs and ready to receive Application data. 
