# Observability Stack with OpenTelemetry, Prometheus, Tempo, ClickHouse and Grafana

## Visão Geral

Este projeto provê uma stack completa de observabilidade local baseada em:

* **OpenTelemetry Collector**: recebe traces e métricas das aplicações e gera métricas a partir dos traces (span metrics).
* **Prometheus**: armazena e consulta as métricas recebidas e geradas pelo OTEL Collector.
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
| Tempo (traces) | [http://localhost:3200/ready](http://localhost:3200/ready) | Backend de traces                                     |
| ClickHouse     | [http://localhost:8123](http://localhost:8123) | Banco para dados analíticos e de negócios             |
| OTEL Collector | OTLP gRPC: 4317 / OTLP HTTP: 4318              | Receptor de traces e métricas no formato OTLP         |

O Grafana está configurado com acesso anônimo como **Admin** e sem tela de login, o que é adequado apenas para uso local.

---

## Serviços detalhados

### 🔭 OpenTelemetry Collector

* Recebe traces e métricas no formato OTLP (não há pipeline de logs)
* Envia os traces para o Tempo e o ClickHouse
* Expõe as métricas das aplicações para o Prometheus na porta 8889, com o label `service_name`
* Gera métricas de chamadas e latência (connector `span_metrics`) e as expõe para o Prometheus na porta 8889
* Aceita as convenções HTTP antigas (`http.method`, `http.status_code`) e as estáveis (`http.request.method`, `http.response.status_code`); ambas viram os labels `http_method` e `http_status_code`
* Configurado via `otel/otel-collector-config.yaml`

### 📈 Prometheus

* Armazena e serve métricas Prometheus
* Utiliza `prometheus/prometheus.yml` com scrape automático do OTEL Collector

### 📊 Grafana

* Visualização unificada de métricas (Prometheus), traces (Tempo) e dados de negócios (ClickHouse)
* Datasources e dashboards provisionados automaticamente
* Regras de alerta podem ser adicionadas em `grafana/provisioning/alerting`

### 🧵 Grafana Tempo

* Armazena spans e traces
* Integra com o painel de trace e painel de latência por endpoint

### ⚡ ClickHouse

* Armazena os spans na tabela `default.otel_traces`, incluindo os eventos de exceção (via OTEL Collector)
* Permite análises SQL rápidas e avançadas

---

## Dashboards incluídos

### 1. `System Observability`

Painel técnico com métricas extraídas via Prometheus:

* **Throughput geral e por endpoint**
* **Latência média e percentis (p50, p70, p95, p99)**
* **Throughput por status code e erros 4xx/5xx**
* **Taxa de erro (% errors / total)**
* **Top traces lentos (via Tempo)**
* **Top exceções por rota (via ClickHouse)**

### 2. `Business Observability`

Painel focado em fluxos de negócio, com base em spans de processos como `begin_checkout`, `payment` e `get.checkout`:

* **Tempo médio e percentis (p50, p95, p99) por etapa do fluxo**
* **Spans de `get.checkout` agrupados por status do span (ClickHouse)**
* **Top tipos de exceção em `get.checkout` (ClickHouse)**

O dashboard `System Observability` tem as variáveis `service_name` e `endpoint` para filtrar por serviço ou rota. O `Business Observability` não tem variáveis: os nomes dos spans de negócio estão fixos nas queries.

---

## Como utilizar

### Enviando dados

Envie traces e métricas para o OTEL Collector local:

* **Endpoint OTLP gRPC**: `http://localhost:4317`
* **Endpoint OTLP HTTP**: `http://localhost:4318`
* **Formato recomendado**: OTLP

Pode-se utilizar bibliotecas do OpenTelemetry nas suas aplicações Python, Node, Java etc.

Para os painéis mostrarem dados:

* Os painéis técnicos consideram apenas spans do tipo `SERVER` (gerados pelas instrumentações HTTP).
* As exceções são lidas dos eventos `exception` dos spans, criados por `span.record_exception(...)` ou automaticamente pelas instrumentações.

Sobre as métricas das aplicações:

* Ficam disponíveis no Prometheus com o label `service_name` (igual às span metrics) e com `job` = nome do serviço.
* Os nomes seguem a conversão padrão do OpenTelemetry para Prometheus: pontos viram `_`, counters ganham `_total` e a unidade vira sufixo (ex.: `orders.created` → `orders_created_total`; histograma `payment.duration` em `ms` → `payment_duration_milliseconds_bucket`; gauge com unidade `1` → `_ratio`).
* Métricas com temporalidade delta são convertidas para cumulativas pelo collector.
* Os dashboards incluídos usam apenas as span metrics; métricas das aplicações podem ser exploradas no Grafana (Explore) ou em painéis próprios.

### Visualizando

1. Acesse `http://localhost:3000`
2. Vá em **Dashboards** e abra a pasta **Observability**
3. Selecione: `System Observability` ou `Business Observability`

---

## Estrutura dos arquivos

* `docker-compose.yaml`: orquestra todos os serviços
* `otel/otel-collector-config.yaml`: configuração do collector OTEL
* `prometheus/prometheus.yml`: scrape config Prometheus
* `tempo/tempo.yaml`: configuração do Tempo
* `grafana/provisioning/datasources/datasources.yaml`: define conexões do Grafana com Tempo, Prometheus e ClickHouse
* `grafana/provisioning/dashboards/dashboards.yaml`: inclui dashboards padrão automaticamente
* `grafana/provisioning/dashboards/default/api-system-observability.json`: painel de métricas técnicas
* `grafana/provisioning/dashboards/default/api-business-observability.json`: painel de fluxo de negócios
* `grafana/provisioning/alerting/`: regras de alerta provisionadas (vazio por enquanto)

---

## Contribuindo

Sinta-se livre para clonar e adaptar para seus projetos. Sugestões e melhorias são bem-vindas!

---

## Licença

MIT License
