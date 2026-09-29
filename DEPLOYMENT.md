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
| Platform | Render |
| Public URL | https://day12-agent-p3pa.onrender.com |
| Deployment date | 2026-09-29 |
| Runtime | Docker web service with Render Key Value datastore |

Environment variable names configured for the service: `AGENT_API_KEY`, `REDIS_URL`, `RATE_LIMIT_PER_MINUTE`, `MONTHLY_BUDGET_USD`, and `LOG_LEVEL`. Secret values and the Redis connection string are stored only in Render environment settings and are not recorded here.

## Cloud Verification

| Request | Actual result |
|---|---|
| `GET /health` | HTTP 200, `{"status":"ok","service":"day12-agent","version":"1.0.0"}` |
| `GET /ready` | HTTP 200, `{"status":"ready","redis":true}` |
| Unauthenticated `POST /ask` | HTTP 401 |

The application does not define a `/` route; the resulting HTTP 404 at the root URL is expected.

## Screenshots

- `screenshots/health.png` — actual `/health` response.
- `screenshots/dashboard.png` — Render Dashboard showing `day12-agent` deployed, all services up and running, and `day12-redis` (Valkey) available.
