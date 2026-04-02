# Docker scenario

System tests use a telemetrygen + OTel Collector setup to produce OTLP data into Kafka, which the Elastic Agent's Kafka receiver then consumes.

## Architecture

```
telemetrygen  -->  otelcol (OTLP recv + Kafka exporter)  -->  Kafka  -->  Elastic Agent (Kafka receiver)  -->  Elasticsearch
```

- **telemetrygen**: Emits logs, metrics, and traces via OTLP gRPC. Uses the [telemetrygen](https://github.com/open-telemetry/opentelemetry-collector-contrib/tree/main/cmd/telemetrygen) tool from opentelemetry-collector-contrib.
- **otelcol**: OpenTelemetry Collector Contrib with OTLP receiver and Kafka exporter. Receives from telemetrygen and writes to Kafka topics `otlp_logs`, `otlp_metrics`, `otlp_spans`.
- **Kafka**: Broker (KRaft mode) that buffers telemetry before the Agent consumes it.
- **Elastic Agent**: Runs the Kafka input (Kafka receiver). Test config uses `initial_offset: earliest` so the Agent reads from the start of topics after telemetrygen has produced data.
