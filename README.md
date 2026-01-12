# Grafana OpenTelemetry Collector

This setup provides a Docker Compose configuration for running an OpenTelemetry Collector that exports metrics, traces, and logs from local services to Grafana Cloud.

## Prerequisites

- Docker and Docker Compose installed
- Grafana Cloud account with access to:
  - Metrics (Prometheus)
  - Traces (Tempo)
  - Logs (Loki)

## Setup

1. **Create your `.env` file:**
   ```bash
   cp env.example .env
   ```

2. **Fill in your Grafana Cloud credentials in `.env`:**
   
   To get your Grafana Cloud credentials:
   - Log in to [Grafana Cloud](https://grafana.com/auth/sign-in/)
   - Navigate to your stack (or create one if you don't have one)
   - Go to **Connections** → **Add new connection** or **Your stack** → **Details**
   
   For each service (Metrics, Traces, Logs):
   - **Metrics (Prometheus):**
     - Endpoint: `https://prometheus-<region>.grafana.net/api/prom/push`
     - Username: Your Prometheus instance ID
     - API Key: Your Prometheus API key
   
   - **Traces (Tempo):**
     - Endpoint: `https://tempo-<region>.grafana.net:443`
     - Username: Your Tempo instance ID
     - API Key: Your Tempo API key
   
   - **Logs (Loki):**
     - Endpoint: `https://logs-<region>.grafana.net/loki/api/v1/push`
     - Username: Your Loki instance ID
     - API Key: Your Loki API key
   
   Replace `<region>` with your Grafana Cloud region (e.g., `us-central1`, `eu-west1`, etc.)
   
   Update the `.env` file with your actual values.

4. **Start the collector:**
   ```bash
   docker-compose up -d
   ```

5. **Verify the collector is running:**
   ```bash
   docker-compose ps
   docker-compose logs otel-collector
   ```

## Network

The collector creates a Docker network named `local-otel-collector` that other services can connect to. Services can send telemetry data to the collector using:

- **gRPC endpoint:** `otel-collector:4317` (when connected to the network)
- **HTTP endpoint:** `otel-collector:4318` (when connected to the network)
- **Local access:** `localhost:4317` (gRPC) or `localhost:4318` (HTTP)

## Connecting Services

### Option 1: Connect service to the network

Add your service to the `local-otel-collector` network in your service's docker-compose:

```yaml
services:
  your-service:
    # ... your service config ...
    networks:
      - local-otel-collector

networks:
  local-otel-collector:
    external: true
```

Then configure your service to send telemetry to `otel-collector:4317` or `otel-collector:4318`.

### Option 2: Use localhost (for services running on host)

Configure your service to send telemetry to:
- `http://localhost:4317` (gRPC)
- `http://localhost:4318` (HTTP)

## Environment Variables for Applications

Set these in your application to send telemetry to the collector:

```bash
export OTEL_EXPORTER_OTLP_ENDPOINT=http://localhost:4318
export OTEL_EXPORTER_OTLP_PROTOCOL=http/protobuf
export OTEL_SERVICE_NAME=your-service-name
```

## Health Check

The collector exposes a health check endpoint at `http://localhost:13133`. You can check the collector status:

```bash
curl http://localhost:13133
```

## Debugging

- View collector logs: `docker-compose logs -f otel-collector`
- Check collector metrics: `http://localhost:13133/metrics`
- View zpages: `http://localhost:55679`

## Stopping the Collector

```bash
docker-compose down
```

To also remove the network:

```bash
docker-compose down --remove-orphans
```
