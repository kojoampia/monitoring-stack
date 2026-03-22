# Monitoring Stack

This directory contains a Docker-based monitoring stack using the Grafana LGTM (Loki-Grafana-Tempo-Mimir) stack, with the addition of an OpenTelemetry Collector and Grafana Alloy.

## Overview

The stack is designed to provide a complete observability solution, covering logs, metrics, and traces. All services are configured to work together, and data is visualized in Grafana. By default, all ports are bound to `127.0.0.1` for security.

## Services

The stack is composed of the following services:

*   **Grafana**: The visualization layer of the stack. It's where you can create dashboards to view metrics, logs, and traces.
    *   Access: [http://127.0.0.1:3000](http://127.0.0.1:3000)

*   **Loki**: A log aggregation system designed to store and query logs from all your applications and infrastructure.
    *   Receives logs from the OpenTelemetry Collector and Alloy.

*   **Mimir**: A scalable long-term storage for Prometheus metrics.
    *   Receives metrics from the OpenTelemetry Collector.

*   **Tempo**: A high-volume, minimal-dependency distributed tracing backend.
    *   Receives traces from the OpenTelemetry Collector.

*   **OpenTelemetry Collector**: A vendor-agnostic way to receive, process, and export telemetry data. It's configured to receive OTLP data and export it to Loki, Tempo, and Mimir.
    *   OTLP gRPC endpoint: `127.0.0.1:4317`
    *   OTLP HTTP endpoint: `127.0.0.1:4318`

*   **Alloy**: A vendor-neutral telemetry collector from Grafana, based on the OpenTelemetry Collector. It can be used to collect logs, metrics, and traces.

## Getting Started

To run the stack, you need to have Docker and Docker Compose installed.

1.  Navigate to the `monitoring-stack` directory.
2.  Run the following command to start the stack in detached mode:

    ```sh
    docker-compose up -d
    ```

## .env Configuration

Create a `.env` file in this directory before starting the stack. The table below documents the expected variables.

| Variable | Required | Used By | Description | Example |
|---|---|---|---|---|
| MINIO_ROOT_USER | Yes | MinIO, minio-setup | MinIO admin username used during object storage bootstrap. | admin |
| MINIO_ROOT_PASSWORD | Yes | MinIO, minio-setup | MinIO admin password used during object storage bootstrap. | ChangeMeStrongPassword |
| S3_ENDPOINT | Yes | Loki, Mimir, Tempo, minio-setup | S3-compatible endpoint used as object storage backend for logs, metrics, and traces. | minio:9000 |
| S3_ACCESS_KEY | Yes | Loki, Mimir, Tempo | Access key for S3-compatible storage. Should match MinIO credentials in this setup. | admin |
| S3_SECRET_KEY | Yes | Loki, Mimir, Tempo | Secret key for S3-compatible storage. Should match MinIO credentials in this setup. | ChangeMeStrongPassword |
| MINIO_BUCKETS | Yes | minio-setup | Comma-separated list of buckets to create during startup. | loki,mimir,mimir-alertmanager,tempo |
| SMTP_USERNAME | Optional | Alerting integrations | SMTP account username for email notifications. | alerts@example.com |
| SMTP_PASSWORD | Optional | Alerting integrations | SMTP account password or app password. | app-password |
| TELEGRAM_BOT_TOKEN | Optional | Alerting integrations | Telegram bot token for Telegram notifications. | 123456:ABCDEF |
| TELEGRAM_CHAT_ID | Optional | Alerting integrations | Target Telegram chat ID for alerts. | -100123456789 |
| SMTP_MAIL_FROM | Optional | Alerting integrations | Sender identity used in outgoing alert emails. | "Monitoring"<alerts@example.com> |
| SMTP_MAIL_BASE | Optional | Alerting integrations | Base domain used in email-related templates or links. | monitoring.example.com |
| SMTP_MAIL_SERVER | Optional | Alerting integrations | SMTP relay host. | smtp-relay.gmail.com |
| SMTP_MAIL_PORT | Optional | Alerting integrations | SMTP relay port. | 465 |
| SMTP_MAIL_USER | Optional | Alerting integrations | SMTP login user. | alerts@example.com |
| SMTP_MAIL_PASSWORD | Optional | Alerting integrations | SMTP login password/app password. | app-password |
| JEDI_MAIL_PASSW | Optional | Alerting integrations | Custom variable used by local alerting/email wiring. | app-password |
| SPRING_DOCKER_COMPOSE_ENABLED | Optional | Application integration | Toggle used by Spring apps that may run alongside the stack. | false |

Minimal required `.env` for stack startup:

```env
MINIO_ROOT_USER=admin
MINIO_ROOT_PASSWORD=ChangeMeStrongPassword

S3_ENDPOINT=minio:9000
S3_ACCESS_KEY=admin
S3_SECRET_KEY=ChangeMeStrongPassword

MINIO_BUCKETS=loki,mimir,mimir-alertmanager,tempo
```

## Accessing Services

All services are exposed on `127.0.0.1` to prevent exposing them to the network by default.

*   **Grafana**: [http://127.0.0.1:3000](http://127.0.0.1:3000)
*   **Loki**: Port `3100`
*   **Mimir**: Port `9009`
*   **Tempo**: Port `3200`
*   **OpenTelemetry Collector Endpoints**:
    *   gRPC: `127.0.0.1:4317`
    *   HTTP: `127.0.0.1:4318`
