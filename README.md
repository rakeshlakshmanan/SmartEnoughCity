# SmartEnoughCity

Transport and mobility platform for the Greater Dublin Area. It pulls live and static data from 15+ public transport APIs into PostgreSQL/PostGIS, runs forecasting models on it, and gives city planners dashboards for buses, trains, trams, Dublin Bikes, cars, EV charging, disruptions, and events.

Team project for the Advanced Software Engineering module at Trinity College Dublin (Oct 2025 - Mar 2026). This is my fork of the team's repository.

## Architecture

| Service | Stack | What it does |
|---|---|---|
| `backend/hermes` | Java 17, Spring Boot 3.5, Spring Security, Keycloak | Main REST API for indicators, disruptions, recommendations, notifications, and user management |
| `backend/data_handler` | Python 3.12, SQLAlchemy, GeoAlchemy2 | Ingests bus (GTFS, GTFS-RT), Irish Rail, Luas, Dublin Bikes (GBFS), traffic and roadworks (TII), events, pedestrian, and population data into PostGIS |
| `backend/inference_engine` | Python 3.12, FastAPI, scikit-learn | Demand forecasts, utilisation indicators, and bike-station risk scores on a schedule |
| `frontend` | React 19, TypeScript, Vite, MUI, Leaflet, Recharts, TanStack Query | Dashboards and maps for each transport mode, plus public disruption and event pages |
| Keycloak | Keycloak 26 | Login, roles, and tokens |
| PostgreSQL | PostGIS | One schema per service |

Infrastructure runs on GKE. Terraform provisions the cluster, cert-manager, ingress-nginx, Istio, and network policies, and Helm deploys the services. Prometheus, Grafana, and Loki handle metrics and logs.

GitHub Actions runs linting, formatting, type checks, and tests for every service on each pull request, with a 75% coverage floor on the data handler. The CD pipeline builds the changed images, pushes them to GHCR, and deploys them to GKE with Helm.

## My contributions

- Built the Dublin Bikes module end to end: 17 Spring Boot endpoints for live stations, network summary, busiest and underused stations, rebalancing, hourly demand, demand classification, origin-destination pairs, risk scores, and coverage gaps with a proposal review flow
- Wrote the station risk engine in the inference engine: one logistic regression per station, trained on 30 days of snapshots, that predicts whether a station will be empty or full in 2 hours and rescores every 5 minutes
- Built the Cycle dashboard in React, with station demand, station risk, coverage-gap, and proposal panels
- Added the Keycloak authentication APIs and their tests
- Built the bus indicator APIs and tests and connected them to the frontend
- Added in-progress and completed statuses to disruption management
- Added the backend Helm chart and a log sanitiser, and fixed SpotBugs and Error Prone findings to get CI passing
- Worked with multiple Claude Code agents in parallel, guided by CLAUDE.md files and Claude Skills

## Running locally

Start PostGIS (port 55432) and Keycloak (port 8081):

```bash
docker compose up -d
```

Then start each service in its own terminal:

```bash
# Spring Boot API on port 8080
cd backend/hermes && ./gradlew bootRun --args='--spring.profiles.active=dev'

# Data ingestion
cd backend/data_handler && uv sync && uv run data-handler

# Inference engine on port 8000
cd backend/inference_engine && uv sync && uv run fastapi dev src/inference_engine/main.py

# Frontend
cd frontend && pnpm install && pnpm dev
```

Development settings for the Python services are in each service's `.env.development`.

## Checks

`./check.sh` runs the same lint, format, and test steps as CI. Use `--python`, `--java`, or `--frontend` to run one part.

## Team

Built by a team of students at Trinity College Dublin. See the contributors list for everyone who worked on it.
