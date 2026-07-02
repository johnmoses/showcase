# Production Infrastructure on $20/month

**System:** Life Reports (10K users)  
**Stack:** AWS Lightsail · Docker Compose · GitHub Actions · Nginx · fail2ban

## The Constraint

Single $20/month Lightsail instance. Every config decision was made to stay within that budget without sacrificing reliability or security.

## Docker Compose — Memory-Bounded Services

```yaml
services:
  db:
    image: postgres:15
    mem_limit: 512m
    environment:
      POSTGRES_SHARED_BUFFERS: 256MB
      POSTGRES_MAX_CONNECTIONS: 50      # tuned for single instance
    ports:
      - "127.0.0.1:5432:5432"           # never exposed publicly

  backend:
    mem_limit: 384m
    healthcheck:
      test: ["CMD", "curl", "-f", "http://localhost:8000/health"]
      start_period: 30s                 # avoids false restarts on cold boot

  web:
    mem_limit: 384m

  autoheal:
    image: willfarrell/autoheal        # auto-restarts unhealthy containers
    volumes:
      - /var/run/docker.sock:/var/run/docker.sock
```

## Server Hardening (5 steps)

```bash
# 1. SSH — key-only, no root, max 3 attempts
sed -i 's/#PasswordAuthentication yes/PasswordAuthentication no/' /etc/ssh/sshd_config
sed -i 's/PermitRootLogin yes/PermitRootLogin no/' /etc/ssh/sshd_config

# 2. fail2ban — SSH 2h ban, nginx 10min ban
# 3. Unattended security upgrades
# 4. Kernel hardening — SYN flood protection, anti-spoofing
echo "net.ipv4.tcp_syncookies = 1" >> /etc/sysctl.conf
echo "net.ipv4.conf.all.rp_filter = 1" >> /etc/sysctl.conf

# 5. File permissions
chmod 600 /etc/ssl/private/*.key
```

## Rate Limiter (Python — sliding window)

```python
class RateLimiter:
    def is_allowed(self, client_ip: str) -> bool:
        now = time.time()
        client_requests = self.requests[client_ip]  # deque per IP

        # Evict requests outside the window
        while client_requests and client_requests[0] <= now - self.window_seconds:
            client_requests.popleft()

        if len(client_requests) >= self.max_requests:
            return False

        client_requests.append(now)
        return True

general_limiter = RateLimiter(max_requests=5000, window_seconds=60)
auth_limiter    = RateLimiter(max_requests=1000, window_seconds=60)

# Real IP extraction behind Nginx proxy
client_ip = request.headers.get("X-Forwarded-For", request.client.host).split(",")[0].strip()
```

## CI/CD (GitHub Actions)

```yaml
- name: Deploy to Lightsail
  run: |
    ssh $LIGHTSAIL_HOST "
      cd /app &&
      git pull &&
      docker compose pull &&
      docker compose up -d --remove-orphans
    "
```

## Cost Breakdown

| Resource | Monthly Cost |
|----------|-------------|
| Lightsail instance (2GB RAM) | $10 |
| Static IP | $0 (free with instance) |
| SSL (Let's Encrypt) | $0 |
| Backups | $2 |
| **Total** | **~$12-20** |
