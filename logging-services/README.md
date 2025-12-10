# Microservices Lab - Logging Service

This project provides a comprehensive observability stack for microservices, including metrics, logs, traces, and visualization capabilities.

## Services Overview

### Loki

**Goal:** Loki is a horizontally-scalable, highly-available log aggregation system designed to work seamlessly with Grafana for log visualization and querying.

**Technical Details:**
- **Port:** `3100` (HTTP API)
- **Image:** `grafana/loki:latest`
- **Storage:** Uses filesystem-based storage with TSDB indexing
- **Features:**
  - Receives logs via OTLP HTTP endpoint (`/otlp`)
  - Supports structured metadata
  - In-memory ring for coordination (single instance setup)
  - Data persisted in Docker volume `loki_data`

### Prometheus

**Goal:** Prometheus is a time-series database and monitoring system that collects and stores metrics from various sources, providing powerful querying capabilities.

**Technical Details:**
- **Port:** `9090` (Web UI and API)
- **Image:** `prom/prometheus:latest`
- **Scrape Interval:** 5 seconds
- **Configuration:** Scrapes metrics from:
  - OpenTelemetry Collector (internal metrics on port 8888 and OTLP metrics on port 8889)
  - Loki (port 3100)
  - Tempo (port 3200)
  - Grafana (port 3000)
- **Storage:** Data persisted in Docker volume `prometheus_data`

### Tempo

**Goal:** Tempo is a high-volume, minimal-dependency distributed tracing backend that stores and queries traces efficiently.

**Technical Details:**
- **Port:** `3200` (HTTP API for querying traces)
- **Image:** `grafana/tempo:latest`
- **Receivers:** Supports OTLP protocol for receiving traces
  - gRPC endpoint: `0.0.0.0:4317`
  - HTTP endpoint: `0.0.0.0:4318`
- **Storage:** Local filesystem backend with data persisted in Docker volume `tempo_data`

## OpenTelemetry Collector Configuration

**Goal:** The OpenTelemetry Collector acts as a unified telemetry data pipeline that receives observability data from applications and routes it to appropriate backends (Loki, Tempo, Prometheus).

**Technical Details:**
- **Image:** `otel/opentelemetry-collector-contrib:latest`
- **Exposed Ports:**
  - `4317`: OTLP gRPC (for external services to send telemetry)
  - `4318`: OTLP HTTP (for external services to send telemetry)
  - `8888`: Collector internal metrics endpoint (for debugging)
- **Configuration (`otel-collector-config.yml`):**
  - **Receivers:** Accepts OTLP protocol (gRPC and HTTP)
  - **Exporters:**
    - Logs → Loki via OTLP HTTP (`http://loki:3100/otlp`)
    - Traces → Tempo via OTLP gRPC (`tempo:4317`)
    - Metrics → Prometheus exporter (internal endpoint `:8889`, scraped by Prometheus)
  - **Processors:** Uses batch processor to optimize data transmission
  - **Pipelines:**
    - Logs pipeline: Receives → Batches → Exports to Loki
    - Traces pipeline: Receives → Batches → Exports to Tempo
    - Metrics pipeline: Receives → Batches → Exports to Prometheus

## Grafana

**Goal:** Grafana provides a unified visualization platform for metrics, logs, and traces, enabling comprehensive observability dashboards and queries across all telemetry data.

**Technical Details:**
- **Port:** `3000` (Web UI)
- **Image:** `grafana/grafana:latest`
- **Default Credentials:**
  - Username: `admin`
  - Password: `admin`
- **Features:**
  - Pre-configured to connect to Loki, Tempo, and Prometheus
  - Anonymous access enabled with Admin role
  - Data persisted in Docker volume `grafana_data`
- **Dependencies:** Waits for Loki, Tempo, and Prometheus to be ready before starting

## Service Interactions

The observability stack follows a unified data flow pattern:

1. **Data Ingestion:**
   - External applications send telemetry data (logs, traces, metrics) to the OpenTelemetry Collector via OTLP protocol on ports `4317` (gRPC) or `4318` (HTTP)

2. **Data Routing:**
   - **Logs:** OpenTelemetry Collector receives logs via OTLP and forwards them to Loki using OTLP HTTP protocol
   - **Traces:** OpenTelemetry Collector receives traces via OTLP and forwards them to Tempo using OTLP gRPC protocol
   - **Metrics:** OpenTelemetry Collector receives metrics via OTLP and exposes them on an internal Prometheus-compatible endpoint (port `8889`)

3. **Data Collection:**
   - Prometheus periodically scrapes metrics from:
     - OpenTelemetry Collector's internal metrics (port `8888`)
     - OpenTelemetry Collector's OTLP metrics endpoint (port `8889`)
     - Loki, Tempo, and Grafana's native metrics endpoints

4. **Data Visualization:**
   - Grafana connects to all three backends:
     - Queries logs from Loki
     - Queries traces from Tempo
     - Queries metrics from Prometheus
   - All services communicate via the `observability` Docker network

## Usage

### Starting Services

To start all services:

```bash
docker-compose up -d
```

Or using the provided script:

```bash
./up.sh
```

This will start all services in detached mode:
- Prometheus (port 9090)
- OpenTelemetry Collector (ports 4317, 4318, 8888)
- Loki (port 3100)
- Tempo (port 3200)
- Grafana (port 3000)

### Stopping Services

To stop all services without removing containers:

```bash
docker-compose stop
```

### Removing Services

To stop and remove all containers, networks, and volumes:

```bash
docker-compose down
```

Or using the provided script:

```bash
./down.sh
```

**Note:** Using `docker-compose down` will remove volumes, which will delete all stored data (metrics, logs, traces, and Grafana configurations). To preserve volumes, use:

```bash
docker-compose down --volumes=false
```

### Accessing Services

- **Grafana UI:** http://localhost:3000 (admin/admin)
- **Prometheus UI:** http://localhost:9090
- **Loki API:** http://localhost:3100
- **Tempo API:** http://localhost:3200
