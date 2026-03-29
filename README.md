# FluxNova Engine Server

Local Camunda 7 BPM engine server that provides a FluxNova-compatible `engine-rest` API for workflow orchestration.

## Overview

FluxNova BPM is built on Camunda 7. Because the FluxNova Spring Boot starters (`org.finos.fluxnova.bpm.springboot`) are not published to Maven Central, this project uses the Camunda 7 starters directly — the REST API surface is identical.

The engine exposes the full Camunda REST API at `/engine-rest` and includes the Cockpit and Tasklist web UIs for visualising and managing BPMN process instances.

## Tech Stack

- Java 21, Spring Boot 3.4.4
- Camunda 7.22 (`camunda-bpm-spring-boot-starter-webapp` + `-rest`)
- H2 in-memory DB for local dev; Aurora MySQL for production (env var override)

## Running Locally

### Prerequisites

- Java 21
- Maven 3.9+

### Build and Run

```bash
mvn clean install
java -jar target/fluxnova-server-0.0.1-SNAPSHOT.jar
```

The engine starts on port `8090`. Access the web UIs at `http://localhost:8090` with credentials `demo / demo`.

| URL | Description |
|-----|-------------|
| `http://localhost:8090` | Camunda Cockpit / Tasklist |
| `http://localhost:8090/engine-rest/engine` | API health check |

### Deploying a BPMN Process

```bash
curl -X POST http://localhost:8090/engine-rest/deployment/create \
  -F "deployment-name=my-process" \
  -F "deploy-changed-only=true" \
  -F "data=@my-process.bpmn"
```

> **BPMN requirements:**
> - Add `camunda:historyTimeToLive="P90D"` to the `<process>` element — deployment fails without it
> - Include a full `<bpmndi:BPMNDiagram>` section with shape/edge coordinates — Cockpit cannot render the diagram without it

### Active Processes

| Process Key | Description |
|-------------|-------------|
| `trip-planning-process` | Start → Review Destination → Set Travel Dates → Family Approval → End |

## Configuration

Local configuration is in `src/main/resources/application.yaml`. All values can be overridden with environment variables at runtime:

| Environment Variable | Description |
|---|---|
| `SPRING_DATASOURCE_URL` | JDBC URL (defaults to H2 in-memory) |
| `SPRING_DATASOURCE_USERNAME` | DB username |
| `SPRING_DATASOURCE_PASSWORD` | DB password |
| `CAMUNDA_BPM_ADMIN_USER_ID` | Admin username (default: `demo`) |
| `CAMUNDA_BPM_ADMIN_USER_PASSWORD` | Admin password (default: `demo`) |

## AWS Deployment

Infrastructure is managed by the [fluxnova-cdk](https://github.com/sports4him12/fluxnova-cdk) project. In production the engine runs as an internal ECS Fargate service (no public access) backed by Aurora MySQL. The trip planner service reaches it via CloudMap DNS at `fluxnova-engine.fluxnova.internal:8090`.
