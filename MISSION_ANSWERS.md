# Day 12 Lab - Mission Answers

## Part 1: Localhost vs Production

### Exercise 1.1: Anti-patterns found
1. **Hardcoded Secrets**: The API key (`sk-xxx...`) was written directly in the code, which is a major security leak if pushed to Git.
2. **Hardcoded Configuration**: Settings like `PORT = 8000` or `DEBUG = True` were fixed in the script, making it difficult to change environments without editing code.
3. **Improper Logging**: Using `print()` instead of proper logging modules (`logging`) makes monitoring in production impossible.
4. **Lack of Dependency Management**: No `requirements.txt` or `.env` file was used initially to manage the environment.

### Exercise 1.3: Comparison table
| Feature | Develop | Production | Why Important? |
|---------|---------|------------|----------------|
| Config  | Hardcoded / .env | Environment Variables | Security & Flexibility |
| Secrets | Plain text code | Secret Managers / Vault | Prevents data breaches |
| Logging | print() to console | Structured (JSON) logs | Searchability in cloud |
| Debug   | True (verbose) | False | Performance & Security |

---

## Part 2: Docker

### Exercise 2.1: Dockerfile questions
1. **Base image**: `python:3.9-slim` (chosen for small footprint).
2. **Working directory**: `/app` (standard practice to isolate app files).
3. **Non-root user**: `appuser` (security best practice to limit permissions).
4. **Multi-stage build**: Uses a "builder" stage to compile and a "final" stage to run, stripping out build tools.

### Exercise 2.3: Image size comparison
- **Develop (Single Stage)**: ~413.8 MB
- **Production (Multi Stage)**: ~56.7 MB
- **Difference**: ~86.3% reduction
- **Benefit**: Faster deployments, lower storage costs, and smaller attack surface.

### Exercise 2.4: Docker Compose Stack
- **Services started**: `agent` (API), `redis` (Cache/Rate Limit), `qdrant` (Vector DB), `nginx` (Load Balancer).
- **Communication**: Services talk to each other via a private Docker network using service names (e.g., `redis://redis:6379`).

---

## Part 3: Cloud Deployment

### Exercise 3.1: Railway deployment
- **Public URL**: `https://agent-production-2d63.up.railway.app`
- **Screenshot**: 03-cloud-deployment/railway/railway.png

### Exercise 3.2: Render Blueprint (Infrastructure as Code)
- **Difference from Railway CLI**: `render.yaml` allows defining the entire stack (Web + Redis) in one file.
- **rootDir requirement**: Critical for monorepos so Render knows where to find the `Dockerfile` and `requirements.txt`.

---

## Part 4: API Security

### Exercise 4.1: API Key Protection
- **Status 401**: Returned when `X-API-Key` header is missing.
- **Status 403**: Returned when `X-API-Key` is incorrect.
- **Status 200**: Returned when the correct key is provided.

### Exercise 4.3: Rate Limiting
- **Algorithm**: Sliding Window Counter (accurate 60s windows).
- **Student Limit**: 10 requests per minute.
- **Admin Bypass**: Roles like `admin` are mapped to a higher-tier limiter (100 req/min).

### Exercise 4.4: Cost Guard implementation
- **Approach**: Calculates estimated USD cost per token based on real LLM pricing (e.g. $0.15 per 1M tokens).
- **Protection**: If the user's daily spend hits $1.00, it blocks further requests with a `402 Payment Required` status.

---

## Part 5: Scaling & Reliability

### Exercise 5.1: Health & Readiness Checks
- **Liveness (/health)**: Confirms the container is running and monitors memory/uptime. Platform restarts if this fails.
- **Ready (/ready)**: Confirms the agent is ready for traffic. Returns 503 if the model is still loading or during shutdown, preventing users from seeing errors.

### Exercise 5.2: Graceful shutdown
- **Logic**: Caught SIGTERM signal, set `_is_ready = False` to stop receiving new traffic, and waited for in-flight requests to finish (tracking counter).
- **Result**: Allowed current long-running tasks to finish without dropping the connection.

### Exercise 5.3 & 5.4: Scaling and Stateless Design
- **Verification**: Ran 3 agent instances behind an Nginx Load Balancer using `docker compose --scale agent=3`.
- **Statelessness**: Verified that even when different instances handled sequential requests (`instance-9dfba5`, `instance-1cf624`, etc.), the conversation history was preserved because state was moved to **Redis** instead of local memory.
- **Load Balancing**: Observed Nginx distributing requests across all healthy instances using Round-Robin balancing.
