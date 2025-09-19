## Deploying the OpenTelemetry Demo app on Grafana Cloud 
This repository is a guide how to deploy the [OpenTelemetry Demo App](https://github.com/open-telemetry/opentelemetry-demo?tab=readme-ov-file#-opentelemetry-demo) to send data to Grafana Cloud using the Kubernetes Monitoring Helm Chart to be the OpenTelemetry Collector. 

The supplied [values.yaml](otelDemoValues.yaml) makes only a couple small changes from the one supplied with the Demo Application:

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
<img width="771" height="446" alt="image" src="https://github.com/user-attachments/assets/75bd087f-5279-4b38-ba4e-142e0e64a602" />

Continue following the wizard until you get to step 5 that provides the helm command to deploy. We want to deploy as is, but add in this one transformation to prevent some high cardinality from a specific service. ([this is in the Otel Demo App's Otel Collector too](https://github.com/open-telemetry/opentelemetry-demo/blob/main/src/otel-collector/otelcol-config.yml#L145-L152), we are just making sure this config makes it into alloy). 
```
applicationObservability:
  traces:
    transforms:
      span:
        - replace_pattern(name, "\\?.*", "")
        - replace_match(name, "GET /api/products/*", "GET /api/products/{productId}")  
```

The full applicationObservability block should look like this after adding this in: 
```
applicationObservability:
  enabled: true
  traces:
    transforms:
      span:
        - replace_pattern(name, "\\?.*", "")
        - replace_match(name, "GET /api/products/*", "GET /api/products/{productId}")  
  receivers:
    otlp:
      grpc:
        enabled: true
        port: 4317
      http:
        enabled: true
        port: 4318
    zipkin:
      enabled: true
      port: 9411
```

Take note of the OTLP/gRPC endpoint provided in step 6. 

Finally - deploy the Helm chart with the modified values.
```
helm repo add grafana https://grafana.github.io/helm-charts &&
  helm repo update &&
  helm upgrade --install --atomic --timeout 300s grafana-k8s-monitoring grafana/k8s-monitoring \
    --namespace "default" --create-namespace --values k8svalues.yaml. 
```
Once the helm chart is deployed, it should start collecting Infra Metrics/Logs and ready to receive Application data. 



### 2. Application Observability 

