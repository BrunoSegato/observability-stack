# 📊 Observability Stack

Este projeto define uma stack local de observabilidade usando Docker Compose com suporte a **OpenTelemetry Collector**, **Prometheus**, **Grafana** e **Tempo**.

---

## 📦 Serviços

### 🔍 otel-collector
- Imagem: `otel/opentelemetry-collector-contrib:latest`
- Portas:
  - `8889`: Exportador Prometheus (spanmetrics)
  - `13133`: Healthcheck
  - `14317`: OTLP gRPC
  - `14318`: OTLP HTTP
- Configuração: `./otel/otel-collector-config.yaml`

### 📈 Prometheus
- Imagem: `prom/prometheus:latest`
- Porta: `9090`
- Configuração: `./prometheus/prometheus.yml`
- Scrape de métricas do OTEL Collector

### 📊 Grafana
- Imagem: `grafana/grafana:latest`
- Porta: `3000`
- Provisionamento automático:
  - **Datasources**: `grafana/provisioning/datasources/datasources.yaml`
  - **Dashboards**: `grafana/provisioning/dashboards/*.json`

### ⏱ Tempo (Tracing backend)
- Imagem: `grafana/tempo:latest`
- Porta: `3200`
- Configuração: `./tempo/tempo.yaml`
- Armazenamento de traces: `./tempo-data`

---

## 🗂 Estrutura do projeto

```
observability-stack/
├── docker-compose.yaml
├── grafana/
│   └── provisioning/
│       ├── dashboards/
│       │   ├── api-observability.json
│       │   └── dashboards.yaml
│       └── datasources/
│           └── datasources.yaml
├── otel/
│   └── otel-collector-config.yaml
├── prometheus/
│   └── prometheus.yml
├── tempo/
│   ├── tempo.yaml
│   └── tempo-data/
│       ├── wal/
│       └── tempo_cluster_seed.json
```

---

## 🚀 Subindo o ambiente

```bash
docker compose up -d
```

Aguarde até que os containers estejam com status `healthy`.

---

## 🌐 Acessos

| Serviço     | URL                        |
|-------------|----------------------------|
| Grafana     | http://localhost:3000      |
| Prometheus  | http://localhost:9090      |
| Tempo       | http://localhost:3200      |
| OTEL Exporter (metrics) | http://localhost:8889/metrics |

---

## 📊 Dashboards

O Grafana já carrega automaticamente:
- `api-observability.json` para visualização de métricas `p95`, `p99`, throughput e traces via spanmetrics

---

## ✅ Healthcheck

Todos os serviços possuem **healthchecks ativos**, garantindo que só iniciem em sequência quando seus dependentes estiverem prontos.

---

## 📌 Observações

- A latência em `p95`, `p99` pode ser monitorada via:
  ```promql
  histogram_quantile(0.95, rate(traces_span_metrics_duration_milliseconds_bucket[5m]))
  ```

---

## 📄 Licença

MIT