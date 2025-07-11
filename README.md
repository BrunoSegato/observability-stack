# Observability Stack with OpenTelemetry, Prometheus, Tempo, ClickHouse and Grafana

## Visão Geral

Este projeto provê uma stack completa de observabilidade local baseada em:

* **OpenTelemetry Collector**: coleta métricas, traces e logs das aplicações.
* **Prometheus**: armazena e consulta métricas expostas pelo OTEL.
* **Grafana**: interface para visualização de métricas e traces.
* **Grafana Tempo**: armazena e consulta traces.
* **ClickHouse**: armazena eventos e erros de negócio para análises personalizadas.

A stack é provisionada com dashboards prontos e datasources configurados automaticamente.

---

## Como iniciar localmente

```bash
docker compose up -d
```

A stack sobe os seguintes serviços:

| Serviço        | URL                                            | Descrição                                             |
| -------------- | ---------------------------------------------- | ----------------------------------------------------- |
| Grafana        | [http://localhost:3000](http://localhost:3000) | Interface de visualização de métricas, traces e erros |
| Prometheus     | [http://localhost:9090](http://localhost:9090) | Armazenamento e consulta de métricas                  |
| Tempo (traces) | [http://localhost:3200/metrics](http://localhost:3200/metrics) | Backend de traces                                     |
| ClickHouse     | [http://localhost:8123](http://localhost:8123) | Banco para dados analíticos e de negócios             |
| OTEL Collector | OTLP GRPC: 4317                                | Receptor de métricas e traces no formato OTLP         |

Usuário padrão do Grafana:

* **Usuário**: `admin`
* **Senha**: `admin`

---

## Serviços detalhados

### 🔭 OpenTelemetry Collector

* Recebe dados no formato OTLP (traces e métricas)
* Exporta para Prometheus, Tempo e ClickHouse
* Configurado via `otel-collector-config.yaml`

### 📈 Prometheus

* Armazena e serve métricas Prometheus
* Utiliza `prometheus.yml` com scrape automático do OTEL Collector

### 📊 Grafana

* Visualização unificada de métricas (Prometheus), traces (Tempo) e dados de negócios (ClickHouse)
* Dashboards padrão incluídos automaticamente via `dashboards.yaml` e `datasources.yaml`

### 🧵 Grafana Tempo

* Armazena spans e traces
* Integra com o painel de trace e painel de latência por endpoint

### ⚡ ClickHouse

* Armazena eventos e exceções das aplicações (via OTEL Collector)
* Permite análises SQL rápidas e avançadas

---

## Dashboards incluídos

### 1. `API System Observability`

Painel técnico com métricas extraídas via Prometheus:

* **Throughput geral e por endpoint**
* **Latência média e percentis (p50, p70, p95, p99)**
* **Códigos HTTP agrupados (2xx, 4xx, 5xx)**
* **Taxa de erro (% errors / total)**
* **Top traces lentos (via Tempo)**
* **Top exceções por rota (via ClickHouse)**

### 2. `API Business Observability`

Painel focado em fluxos de negócio, com base em spans de processos como `begin_checkout`, `payment` e `get.checkout`:

* **Tempo médio e percentis por etapa do fluxo**
* **Erros agrupados por status code (ClickHouse)**
* **Top tipos de exceção por etapa do processo (ClickHouse)**

Ambos os dashboards suportam variáveis como `service_name` e `endpoint` para filtrar os dados por serviço ou rota específica.

---

## Como utilizar

### Enviando dados

Para ver dados nos dashboards, envie spans ou métricas para o OTEL Collector local:

* **Endpoint OTLP gRPC**: `http://localhost:4317`
* **Formato recomendado**: OTLP

Pode-se utilizar bibliotecas do OpenTelemetry nas suas aplicações Python, Node, Java etc.

### Visualizando

1. Acesse `http://localhost:3000`
2. Vá em **Dashboards > Browse**
3. Selecione: `API System Observability` ou `API Business Observability`

---

## Estrutura dos arquivos

* `docker-compose.yaml`: orquestra todos os serviços
* `otel-collector-config.yaml`: configuração do collector OTEL
* `prometheus.yml`: scrape config Prometheus
* `tempo.yaml`: configuração do Tempo
* `datasources.yaml`: define conexões do Grafana com Tempo, Prometheus e ClickHouse
* `dashboards.yaml`: inclui dashboards padrão automaticamente
* `api-system-observability.json`: painel de métricas técnicas
* `api-business-observability.json`: painel de fluxo de negócios

---

## Contribuindo

Sinta-se livre para clonar e adaptar para seus projetos. Sugestões e melhorias são bem-vindas!

---

## Licença

MIT License
