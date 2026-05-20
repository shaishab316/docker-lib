# RedisInsight Cheatsheet

```bash
docker compose -f redisinsight.yaml up -d
```

→ [http://localhost:5540](http://localhost:5540)

No login required — opens directly in browser.

---

## Connect to Redis

| Field    | Value                |
|----------|----------------------|
| Host     | host.docker.internal |
| Port     | 6379                 |
| Password | your_redis_password  |