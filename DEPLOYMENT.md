# Deployment Information

## Public URL
https://agent-production-2d63.up.railway.app

## Platform
Railway

## Test Commands

### Health Check
```bash
curl https://agent-production-2d63.up.railway.app/health
# Expected: {"status": "ok", "version": "1.0.0"}
```

### API Test (Authenticated)
```bash
# X-API-Key is set in Railway Variables
curl -X POST https://agent-production-2d63.up.railway.app/ask \
  -H "X-API-Key: YOUR_AGENT_API_KEY" \
  -H "Content-Type: application/json" \
  -d '{"question": "How are you today?"}'
```

---

## Environment Variables Set
- `PORT`: (Managed by Railway)
- `REDIS_URL`: (Managed by Railway/Redis Plugin)
- `AGENT_API_KEY`: (Your secret key)
- `ENVIRONMENT`: production
- `DAILY_BUDGET_USD`: 5.0

## Screenshots
- [x] Railway Dashboard showing both Agent and Redis
- [x] Successful curl response from public URL
