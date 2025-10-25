# Compliant Blue/Green Deployment with Nginx

**Strictly follows the HNG Stage 2 DevOps requirements:**
- ✅ Nginx in front of two identical Node.js services
- ✅ Blue (active) and Green (backup) with automatic failover
- ✅ Zero failed client requests during failover
- ✅ Proper header forwarding (X-App-Pool, X-Release-Id)
- ✅ Required ports: 8080 (Nginx), 8081 (Blue), 8082 (Green)
- ✅ Full parameterization via .env file

## Quick Start

1. **Setup environment:**
   ```bash
   cp .env.example .env
   # Edit .env with your Docker images
   ```

2. **Start services:**
   ```bash
   docker compose up -d
   ```

3. **Verify baseline (Blue active):**
   ```bash
   curl -i http://localhost:8080/version
   # Expected: X-App-Pool: blue, X-Release-Id: <RELEASE_ID_BLUE>
   ```

## Testing Failover

1. **Induce chaos on Blue:**
   ```bash
   curl -X POST "http://localhost:8081/chaos/start?mode=error"
   ```

2. **Verify automatic switch to Green:**
   ```bash
   curl -i http://localhost:8080/version
   # Expected: X-App-Pool: green, X-Release-Id: <RELEASE_ID_GREEN>
   ```

3. **Stop chaos:**
   ```bash
   curl -X POST "http://localhost:8081/chaos/stop"
   ```

## Manual Pool Toggle

Edit `.env` file:
```env
ACTIVE_POOL=green  # Switch to green as primary
```

Then restart Nginx:
```bash
docker compose up -d nginx
```

## Architecture

- **Nginx**: Load balancer with backup upstream configuration
- **Blue App**: Primary service (when ACTIVE_POOL=blue)
- **Green App**: Backup service, becomes primary when ACTIVE_POOL=green
- **Failover**: max_fails=1, fail_timeout=5s for quick detection
- **Retry**: proxy_next_upstream ensures zero client 5xx errors

## Requirements Compliance

✅ **Docker Compose orchestration**  
✅ **Nginx templating with envsubst**  
✅ **Proper port exposure (8080, 8081, 8082)**  
✅ **Header forwarding unchanged**  
✅ **Primary/backup upstream configuration**  
✅ **Tight timeouts and retry policy**  
✅ **Full .env parameterization**  

## Cleanup
```bash
docker compose down -v
```