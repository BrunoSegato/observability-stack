# 🚀 Observability Stack

Este repositório fornece uma stack customizada do OpenTelemetry Collector com suporte a métricas de latência (p50, p95, p99), throughput e error rate, exportadas via Prometheus e prontas para visualização no Grafana.

---

## ✅ Componentes incluídos no Collector

- `otlpreceiver` (gRPC)
- `prometheusreceiver`
- `spanmetricsprocessor` → exporta histogramas de latência
- `prometheusexporter`
- `otlpexporter`
- `loggingexporter`

---

## 📦 Estrutura do projeto

```
observability-stack/
├── README.md
├── docker-compose.yaml               # Infraestrutura local
└── otel/
    ├── builder-config.yaml           # Configuração do otelcol-builder
    ├── otel-collector-config.yaml    # Configuração do pipeline OTEL Collector
    ├── Dockerfile                    # Imagem baseada em distroless
    └── build.sh                      # Script para gerar o binário estático
```

---

## 🛠️ Requisitos

- Go ≥ 1.20
- Docker
- Docker Compose
- otelcol-builder instalado:

```bash
go install github.com/open-telemetry/opentelemetry-collector-builder@latest
```

---

## 🧰 Como rodar localmente

1. Acesse a pasta do projeto:

```bash
cd observability-stack/otel
```

2. Gere o binário customizado:

```bash
chmod +x build.sh
./build.sh
```

> O script irá gerar o `otelcol` com build estático e todos os componentes necessários.

3. Suba os serviços:

```bash
docker compose up --build
```

---

## 📈 Acessar métricas

- Prometheus scrape (spanmetrics):

  ```
  http://localhost:8889/metrics
  ```

- Grafana (se você tiver configurado):

  ```
  http://localhost:3000
  ```

> Basta importar o `grafana-dashboard.json` para visualizar p95, p99, throughput e erro por rota.

---

## 📊 Dashboard Grafana incluído

O painel contém:

- 📈 Gráfico de p95 / p99 por operação (PromQL com `histogram_quantile`)
- 📉 Error rate por operação
- 🚀 Throughput (req/s)
- 📋 Tabela de latência por endpoint (p50 / p95 / p99)

---

## 📤 Reutilizando em outros projetos

Basta copiar este repositório para dentro do seu projeto ou utilizá-lo como submódulo:

```bash
git submodule add https://github.com/seu-usuario/otel-collector-custom-stack.git observability
```

E então integrar seu serviço com o OTLP endpoint: `http://otel-collector:4317`

---

## 🤝 Contribuições

Sinta-se à vontade para abrir PRs, sugerir melhorias ou adicionar templates de dashboard para outras linguagens/frameworks.

---

## 🧑‍💻 Autor

Bruno Segato — [@seu-usuario](https://github.com/seu-usuario)

---

## 📄 Licença

Este projeto está licenciado sob a licença MIT.