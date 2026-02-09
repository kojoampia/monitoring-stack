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

## Accessing Services

All services are exposed on `127.0.0.1` to prevent exposing them to the network by default.

*   **Grafana**: [http://127.0.0.1:3000](http://127.0.0.1:3000)
*   **Loki**: Port `3100`
*   **Mimir**: Port `9009`
*   **Tempo**: Port `3200`
*   **OpenTelemetry Collector Endpoints**:
    *   gRPC: `127.0.0.1:4317`
    *   HTTP: `127.0.0.1:4318`
