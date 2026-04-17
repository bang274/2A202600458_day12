#  Delivery Checklist — Day 12 Lab Submission

> **Student Name:** Tran Khanh Bang  
> **Student ID:** 2A202600458 
> **Date:** 17/4/2026

---

##  Submission Requirements

Submit a **GitHub repository** containing:

### 1. Mission Answers (40 points)

Create a file `MISSION_ANSWERS.md` with your answers to all exercises:

``` markdown
# Day 12 Lab - Mission Answers

## Part 1: Localhost vs Production

### Exercise 1.1: Anti-patterns found
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


## Part 2: Docker

### Exercise 2.1: Dockerfile questions
1. **Base image**: `python:3.11` - Full Python distribution (~1GB)
2. **Working directory**: `/app` - Set via `WORKDIR /app`
3. **Why COPY requirements.txt first**: Docker layer caching - if requirements.txt hasn't changed, it won't reinstall dependencies
4. **CMD vs ENTRYPOINT**:
   - `CMD`: Default command that can be overridden at runtime
   - `ENTRYPOINT`: Defines the actual executable, arguments are appended

### Exercise 2.3: Image size comparison
- Develop: 1.67GB
- Production: 262MB
- Difference: 12.5% (production/develop)

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
# Day 12 Lab - Mission Answers

## Part 1: Localhost vs Production

### Exercise 1.1: Anti-patterns found
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


## Part 2: Docker

### Exercise 2.1: Dockerfile questions
1. **Base image**: `python:3.11` - Full Python distribution (~1GB)
2. **Working directory**: `/app` - Set via `WORKDIR /app`
3. **Why COPY requirements.txt first**: Docker layer caching - if requirements.txt hasn't changed, it won't reinstall dependencies
4. **CMD vs ENTRYPOINT**:
   - `CMD`: Default command that can be overridden at runtime
   - `ENTRYPOINT`: Defines the actual executable, arguments are appended

### Exercise 2.3: Image size comparison
- Develop: 1.67GB
- Production: 262MB
- Difference: 12.5% (production/develop)

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
## Part 3: Cloud Deployment

### Exercise 3.1: Railway deployment
- URL: https://web-agent-production-d125.up.railway.app
- Screenshot: [Link to screenshot in repo]

## Part 4: API Security

### Exercise 4.1-4.3: Test results
[Paste your test outputs]

### Exercise 4.4: Cost guard implementation
[Explain your approach]

## Part 5: Scaling & Reliability

### Exercise 5.1-5.5: Implementation notes
[Your explanations and test results]
## Part 3: Cloud Deployment

### Exercise 3.1: Railway deployment
- URL: https://your-app.railway.app
- Screenshot: [Link to screenshot in repo]

## Part 4: API Security

### Exercise 4.1-4.3: Test results
[Paste your test outputs]

### Exercise 4.4: Cost guard implementation
[Explain your approach]

## Part 5: Scaling & Reliability

### Exercise 5.1-5.5: Implementation notes
[Your explanations and test results]
```

---

### 2. Full Source Code - Lab 06 Complete (60 points)

Your final production-ready agent with all files:

```
your-repo/
├── app/
│   ├── main.py              # Main application
│   ├── config.py            # Configuration
│   ├── auth.py              # Authentication
│   ├── rate_limiter.py      # Rate limiting
│   └── cost_guard.py        # Cost protection
├── utils/
│   └── mock_llm.py          # Mock LLM (provided)
├── Dockerfile               # Multi-stage build
├── docker-compose.yml       # Full stack
├── requirements.txt         # Dependencies
├── .env.example             # Environment template
├── .dockerignore            # Docker ignore
├── railway.toml             # Railway config (or render.yaml)
└── README.md                # Setup instructions
```

**Requirements:**
-  All code runs without errors
-  Multi-stage Dockerfile (image < 500 MB)
-  API key authentication
-  Rate limiting (10 req/min)
-  Cost guard ($10/month)
-  Health + readiness checks
-  Graceful shutdown
-  Stateless design (Redis)
-  No hardcoded secrets

---

### 3. Service Domain Link

Create a file `DEPLOYMENT.md` with your deployed service information:

```markdown
# Deployment Information

## Public URL
https://your-agent.railway.app

## Platform
Railway / Render / Cloud Run

## Test Commands

### Health Check
```bash
curl https://your-agent.railway.app/health
# Expected: {"status": "ok"}
```

### API Test (with authentication)
```bash
curl -X POST https://your-agent.railway.app/ask \
  -H "X-API-Key: YOUR_KEY" \
  -H "Content-Type: application/json" \
  -d '{"user_id": "test", "question": "Hello"}'
```

## Environment Variables Set
- PORT
- REDIS_URL
- AGENT_API_KEY
- LOG_LEVEL

## Screenshots
- [Deployment dashboard](screenshots/dashboard.png)
- [Service running](screenshots/running.png)
- [Test results](screenshots/test.png)
```

##  Pre-Submission Checklist

- [ ] Repository is public (or instructor has access)
- [ ] `MISSION_ANSWERS.md` completed with all exercises
- [ ] `DEPLOYMENT.md` has working public URL
- [ ] All source code in `app/` directory
- [ ] `README.md` has clear setup instructions
- [ ] No `.env` file committed (only `.env.example`)
- [ ] No hardcoded secrets in code
- [ ] Public URL is accessible and working
- [ ] Screenshots included in `screenshots/` folder
- [ ] Repository has clear commit history

---

##  Self-Test

Before submitting, verify your deployment:

```bash
# 1. Health check
curl https://your-app.railway.app/health

# 2. Authentication required
curl https://your-app.railway.app/ask
# Should return 401

# 3. With API key works
curl -H "X-API-Key: YOUR_KEY" https://your-app.railway.app/ask \
  -X POST -d '{"user_id":"test","question":"Hello"}'
# Should return 200

# 4. Rate limiting
for i in {1..15}; do 
  curl -H "X-API-Key: YOUR_KEY" https://your-app.railway.app/ask \
    -X POST -d '{"user_id":"test","question":"test"}'; 
done
# Should eventually return 429
```

---

##  Submission

**Submit your GitHub repository URL:**

```
https://github.com/your-username/day12-agent-deployment
```

**Deadline:** 17/4/2026

---

##  Quick Tips

1.  Test your public URL from a different device
2.  Make sure repository is public or instructor has access
3.  Include screenshots of working deployment
4.  Write clear commit messages
5.  Test all commands in DEPLOYMENT.md work
6.  No secrets in code or commit history

---

##  Need Help?

- Check [TROUBLESHOOTING.md](TROUBLESHOOTING.md)
- Review [CODE_LAB.md](CODE_LAB.md)
- Ask in office hours
- Post in discussion forum

---

**Good luck! **
