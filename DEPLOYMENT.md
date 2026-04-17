# Deployment Information

## Public URL
https://web-agent-production-d125.up.railway.app

## Platform
Railway (Cloud Deployment)

## Test Commands

### Health Check
```bash
curl https://web-agent-production-d125.up.railway.app/health
# Expected: {"status":"ok","uptime_seconds":..., "platform":"Railway", ...}
```

### API Test (with authentication)
```bash
curl -X POST https://web-agent-production-d125.up.railway.app/ask \
  -H "X-API-Key: your-secret-api-key" \
  -H "Content-Type: application/json" \
  -d '{"question": "Tìm chuyến bay từ Hà Nội đến Đà Nẵng"}'
```

## Environment Variables Set
- `PORT=8000`
- `REDIS_URL` (Railway provides managed Redis)
- `AGENT_API_KEY=your-secret-api-key`
- `LOG_LEVEL=INFO`
- `RATE_LIMIT_PER_MINUTE=60`
- `MONTHLY_BUDGET_USD=100.0`
- `GITHUB_TOKEN` (configured in Railway variables)

## Deployment Files Location
`06-lab-complete/` directory contains:
- `Dockerfile` - Multi-stage Docker build
- `docker-compose.yml` - Service orchestration
- `app/` - Production agent application code
- `railway.toml` - Railway deployment configuration

## Notes
- The agent uses LangGraph with ReAct pattern
- Tools available: search_flights, search_hotels, calculate_budget
- Rate limiting uses Redis sliding window
- Cost guard tracks monthly spending per user
- Deployed on Railway with automatic HTTPS