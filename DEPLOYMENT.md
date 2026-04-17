# Deployment Information

## Public URL
Currently running locally at `http://localhost:8000`

## Platform
Docker Compose (Local Development) / Railway / Render / Cloud Run

## Test Commands

### Health Check
```bash
curl http://localhost:8000/health
# Expected: {"status":"healthy","timestamp":..., "service":"travelbuddy-agent","version":"1.0.0"}
```

### API Test (with authentication)
```bash
curl -X POST http://localhost:8000/ask \
  -H "X-API-Key: your-secret-api-key" \
  -H "Content-Type: application/json" \
  -d '{"question": "Tìm chuyến bay từ Hà Nội đến Đà Nẵng"}'
```

### Ready Check
```bash
curl http://localhost:8000/ready
```

## Environment Variables Set
- `PORT=8000`
- `REDIS_URL=redis://redis:6379`
- `AGENT_API_KEY=your-secret-api-key`
- `LOG_LEVEL=INFO`
- `RATE_LIMIT_PER_MINUTE=60`
- `MONTHLY_BUDGET_USD=100.0`
- `GITHUB_TOKEN=ghp_***` (GitHub PAT for Azure OpenAI)

## Deployment Files Location
`06-lab-complete/` directory contains:
- `Dockerfile` - Multi-stage Docker build
- `docker-compose.yml` - Service orchestration
- `app/` - Production agent application code (config.py, auth.py, rate_limiter.py, cost_guard.py, main.py, agent.py, tools.py)
- `.env` - Environment configuration
- `nginx.conf` - Nginx configuration (optional)

## Notes
- The agent uses LangGraph with ReAct pattern
- Tools available: search_flights, search_hotels, calculate_budget
- Rate limiting uses Redis sliding window
- Cost guard tracks monthly spending per user
- Deployed code is in `06-lab-complete/` directory