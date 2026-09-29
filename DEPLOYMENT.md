# CP5 Deployment Record

## Student

| Field | Value |
|---|---|
| Name | Vũ Bá Anh |
| Student ID (mã học viên) | 2A202602893 |
| Repository | https://github.com/anhvb702/K4-L3B-DAY12-VuBaAnh-2A202602893-CloudServicesAndDeployment |

## Deployment

| Field | Value |
|---|---|
| Platform | Local fallback: Docker Compose (Railway trial expired; Render dashboard unavailable in this session) |
| Public URL | Not deployed publicly; local service: http://localhost:8001 |
| Deployment date | 2026-09-29 |
| Runtime | Docker Compose agent and Redis; host port 8001 maps to container port 8000 |

The local container receives `AGENT_API_KEY` and `REDIS_URL`; the remaining settings use the app defaults `RATE_LIMIT_PER_MINUTE=10`, `MONTHLY_BUDGET_USD=10.0`, and `LOG_LEVEL=INFO`. `AGENT_API_KEY` is supplied from the ignored local `.env`; no secret value is recorded here. Compose sets `REDIS_URL=redis://redis:6379/0`.

## Verification

| Request | Actual result |
|---|---|
| `GET http://localhost:8001/health` | HTTP 200, `{"status":"ok","service":"day12-agent","version":"1.0.0"}` |
| `GET http://localhost:8001/ready` | HTTP 200, `{"status":"ready","redis":true}` |
| Unauthenticated `POST /ask` | HTTP 401, `{"detail":"invalid or missing API key"}` |
| Authenticated `POST /ask` | HTTP 200; answer returned |

The app and Redis containers were running and healthy during these checks. `LOCAL_FALLBACK=true` is set in the local, ignored `.env` so CP5 tests exercise this Compose stack. This is the documented fallback and is not a public cloud deployment.

## Screenshots

- `screenshots/health.png` — actual local `/health` response.
- `screenshots/dashboard.png` — unavailable; this session has no browser/UI surface to capture a platform dashboard. The Compose status was verified with `docker compose ps`.
