# Day 12 Lab - Mission Answers

## Part 1: Localhost vs Production

### Exercise 1.1: Anti-patterns found (in basic/app.py)

1. **API key hardcoded** - `OPENAI_API_KEY = "sk-hardcoded-fake-key-never-do-this"` - Secret exposed in code, will be leaked if pushed to GitHub
2. **Database URL hardcoded** - `DATABASE_URL = "postgresql://admin:password123@localhost:5432/mydb"` - Contains credentials
3. **Debug mode enabled** - `DEBUG = True` - Shows sensitive info in production
4. **No health check endpoint** - No `/health` or `/ready` endpoint to let platform know if service is alive
5. **Port hardcoded** - `port=8000` - Doesn't read from environment variable, won't work on Railway/Render
6. **Print logging instead of structured logging** - Uses `print()` which outputs plain text, not parseable JSON
7. **Secrets logged** - `print(f"[DEBUG] Using key: {OPENAI_API_KEY}")` - Logs secrets!
8. **Reload enabled in production** - `reload=True` - Can cause issues in production
9. **Host bound to localhost** - `host="localhost"` - Only accepts local connections, not accessible from container

### Exercise 1.3: Comparison table

| Feature | Basic (Develop) | Advanced (Production) | Why Important? |
|---------|----------------|---------------------|---------------|
| Config | Hardcoded values | Environment variables (12-Factor) | Secrets not exposed in code, flexible across environments |
| Health check | None | `/health` and `/ready` endpoints | Platform knows when to restart/ route traffic |
| Logging | `print()` | JSON structured logging | Easy parsing in log aggregators (Datadog, Loki) |
| Shutdown | Sudden (`reload=True`) | Graceful (SIGTERM + lifespan) | Complete in-flight requests before shutdown |
| Port binding | `localhost:8000` | Reads from `PORT` env var | Works on Railway/Render which inject PORT |
| CORS | Not configured | Configured via settings | Security - only allowed origins |
| Debug mode | Always on | `settings.debug` flag | Don't leak info in production |

---

## Part 2: Docker Containerization

### Exercise 2.1: Dockerfile questions

1. **Base image**: `python:3.11` - Full Python distribution (~1GB)
2. **Working directory**: `/app` - Set via `WORKDIR /app`
3. **Why COPY requirements.txt first**: Docker layer caching - if requirements.txt hasn't changed, it won't reinstall dependencies
4. **CMD vs ENTRYPOINT**:
   - `CMD`: Default command that can be overridden at runtime
   - `ENTRYPOINT`: Defines the actual executable, arguments are appended

### Exercise 2.3: Multi-stage build analysis

- **Stage 1 (builder)**: Installs all dependencies including build tools (gcc, libpq-dev) needed to compile packages
- **Stage 2 (runtime)**: Only copies what's needed to RUN, uses `python:3.11-slim` base
- **Why smaller**: Removes build tools, uses slim base image, doesn't include all dev dependencies

### Exercise 2.4: Docker Compose stack

**Services started**:
1. `agent` - FastAPI AI agent (2 replicas defined)
2. `redis` - Cache for session storage and rate limiting
3. `qdrant` - Vector database for RAG
4. `nginx` - Reverse proxy and load balancer

**Communication**:
- Nginx routes external traffic to agent replicas
- Agent connects to Redis and Qdrant via internal network
- All services in `internal` bridge network

---

## Part 3: Cloud Deployment

### Exercise 3.1: Railway deployment

**Steps**:
1. Install Railway CLI: `npm i -g @railway/cli`
2. Login: `railway login`
3. Init: `railway init`
4. Set variables: `railway variables set PORT=8000 AGENT_API_KEY=my-secret-key`
5. Deploy: `railway up`
6. Get URL: `railway domain`

**railway.toml vs render.yaml differences**:
- Railway uses `railway.toml` (Railway-specific config)
- Render uses `render.yaml` (Render-specific, defines services)
- Both define build and deploy settings but use different formats

### Exercise 3.2: Render deployment

**Differences**:
- Requires GitHub integration
- Uses Blueprint (render.yaml) for infrastructure as code
- Automatic deploy on Git push

---

## Part 4: API Security

### Exercise 4.1: API Key authentication (basic version)

**Questions from develop/app.py**:

- **Where is API key checked?**: In `verify_api_key()` function (line 39-54), uses FastAPI's `APIKeyHeader` security dependency
- **What happens with wrong key?**: Returns 403 Forbidden with "Invalid API key" message
- **How to rotate key?**: Change `AGENT_API_KEY` environment variable and restart

### Exercise 4.2: JWT authentication (advanced)

From `production/auth.py`:
1. User calls `/token` endpoint with username/password
2. Server validates and returns JWT token (with expiration)
3. User calls `/ask` with `Authorization: Bearer <token>`
4. Server verifies JWT signature and extracts user_id

### Exercise 4.3: Rate limiting

**From production/rate_limiter.py**:

- **Algorithm**: Sliding Window Counter
- **Limit**: 10 requests/minute (user tier), 100 requests/minute (admin tier)
- **How to bypass for admin**: Uses separate `rate_limiter_admin` instance with higher limit

### Exercise 4.4: Cost guard implementation

**From production/cost_guard.py**:

```python
def check_budget(user_id: str) -> None:
    # Check global budget first
    if global_cost >= global_daily_budget:
        raise 503
    
    # Check per-user budget
    if user_cost >= daily_budget:
        raise 402 Payment Required
    
    # Warn at 80%
```

- Uses in-memory tracking (would use Redis in production)
- Tracks input/output tokens separately
- Calculates cost based on token prices
- Daily reset at midnight UTC

---

## Part 5: Scaling & Reliability

### Exercise 5.1: Health checks

**Implementation in develop/app.py**:

- `/health` (liveness): Returns 200 if process is alive, includes uptime and version
- `/ready` (readiness): Returns 200 if ready, 503 if still starting up
- Includes middleware to track in-flight requests

### Exercise 5.2: Graceful shutdown

**From develop/app.py**:

```python
def handle_sigterm(signum, frame):
    logger.info(f"Received signal {signum} — uvicorn will handle graceful shutdown")

signal.signal(signal.SIGTERM, handle_sigterm)
```

- uvicorn catches SIGTERM and calls lifespan shutdown
- Waits for in-flight requests to complete (up to 30 seconds)
- Sets `_is_ready = False` to stop accepting new requests

### Exercise 5.3: Stateless design

**Why stateless matters**: When scaling to multiple instances, each instance has its own memory. Storing state in one instance's memory means other instances can't access it.

**Solution**: Store conversation history in Redis instead of in-memory dict:
```python
# Instead of:
history = conversation_history.get(user_id, [])

# Use:
history = r.lrange(f"history:{user_id}", 0, -1)
```

### Exercise 5.4: Load balancing

**From docker-compose.yml**:
- Uses Nginx as reverse proxy and load balancer
- `docker compose up --scale agent=3` creates 3 replicas
- Nginx uses round-robin by default
- Health checks ensure traffic only goes to healthy instances

### Exercise 5.5: Test stateless

**What test does**:
1. Calls API to create conversation
2. Kills random instance
3. Calls API again - conversation should still exist if using Redis

---

## Summary

This lab covered:
1. **Development vs Production** - 12-Factor App principles, environment-based config
2. **Docker** - Multi-stage builds, optimized images, Docker Compose orchestration
3. **Cloud Deployment** - Railway, Render, environment variable management
4. **API Security** - API keys, JWT, rate limiting, cost guards
5. **Scaling** - Health checks, graceful shutdown, stateless design, load balancing