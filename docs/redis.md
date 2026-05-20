# Redis Cheatsheet

```bash
docker compose -f redis.yaml up -d
```

---

## 1. Connect

```bash
# Enter CLI
docker exec -it redis redis-cli

# Connect with password
docker exec -it redis redis-cli -a yourpassword

# Ping test
PING                       -- returns PONG
```

---

## 2. Key Basics

```bash
SET name "John"
GET name
DEL name
EXISTS name                -- 1 = exists, 0 = not
KEYS *                     -- list all keys (avoid in prod)
KEYS user:*                -- list keys by pattern
TYPE name                  -- get type of key
RENAME name fullname
```

---

## 3. Expiry (TTL)

```bash
SET token "abc123" EX 3600       -- expires in 3600 seconds
SET token "abc123" PX 60000      -- expires in ms
EXPIRE name 60                   -- set expiry on existing key
TTL name                         -- time left in seconds (-1 = no expiry, -2 = gone)
PERSIST name                     -- remove expiry
```

---

## 4. String

```bash
SET counter 0
INCR counter               -- increment by 1
INCRBY counter 5           -- increment by 5
DECR counter
DECRBY counter 5
APPEND name " Doe"
STRLEN name
MSET a 1 b 2 c 3           -- set multiple
MGET a b c                 -- get multiple
```

---

## 5. Hash (Object-like)

```bash
HSET user:1 name "John" email "john@example.com" age 25
HGET user:1 name
HMGET user:1 name email
HGETALL user:1             -- all fields + values
HDEL user:1 age
HEXISTS user:1 name        -- 1 or 0
HKEYS user:1               -- all field names
HVALS user:1               -- all values
HLEN user:1                -- field count
HINCRBY user:1 age 1
```

---

## 6. List (Queue / Stack)

```bash
LPUSH queue "a"            -- push to left
RPUSH queue "b"            -- push to right
LPOP queue                 -- pop from left
RPOP queue                 -- pop from right
LRANGE queue 0 -1          -- get all items
LLEN queue                 -- length
LINDEX queue 0             -- get by index
LSET queue 0 "updated"
LREM queue 1 "a"           -- remove 1 occurrence of "a"

# Blocking pop (used in job queues)
BLPOP queue 5              -- wait up to 5s
```

---

## 7. Set (Unique values)

```bash
SADD tags "nodejs" "redis" "docker"
SREM tags "docker"
SMEMBERS tags              -- all members
SISMEMBER tags "redis"     -- 1 or 0
SCARD tags                 -- count
SUNION tags othertags      -- union
SINTER tags othertags      -- intersection
SDIFF tags othertags       -- difference
```

---

## 8. Sorted Set (Leaderboard / Priority)

```bash
ZADD leaderboard 100 "Alice"
ZADD leaderboard 200 "Bob"
ZRANGE leaderboard 0 -1 WITHSCORES     -- asc
ZREVRANGE leaderboard 0 -1 WITHSCORES  -- desc
ZSCORE leaderboard "Alice"
ZRANK leaderboard "Alice"              -- rank (0-based, asc)
ZREVRANK leaderboard "Alice"           -- rank (desc)
ZREM leaderboard "Alice"
ZINCRBY leaderboard 50 "Alice"
ZCARD leaderboard                      -- count
```

---

## 9. Pub/Sub

```bash
# Terminal 1 — Subscribe
SUBSCRIBE notifications

# Terminal 2 — Publish
PUBLISH notifications "Hello!"

PSUBSCRIBE user:*          -- pattern subscribe
UNSUBSCRIBE notifications
```

---

## 10. Transactions

```bash
MULTI
SET a 1
SET b 2
INCR counter
EXEC               -- run all

DISCARD            -- cancel transaction
```

---

## 11. Flush / Delete

```bash
DEL key1 key2              -- delete specific keys
FLUSHDB                    -- delete all keys in current db
FLUSHALL                   -- delete all keys in all databases
```

---

## 12. Database

```bash
SELECT 0                   -- switch to db 0 (default)
SELECT 1                   -- switch to db 1
DBSIZE                     -- number of keys in current db
MOVE key 1                 -- move key to db 1
```

---

## 13. Server Info

```bash
INFO                       -- full server info
INFO memory                -- memory usage
INFO replication           -- replication status
MONITOR                    -- live command stream (debug)
CONFIG GET maxmemory
CONFIG SET maxmemory 256mb
SLOWLOG GET                -- slow query log
```

---

## 14. Common Patterns

```bash
# Cache with expiry
SET cache:user:1 "{...json...}" EX 300

# Session token
SET session:abc123 "user_id:42" EX 86400

# Rate limit counter
INCR ratelimit:ip:192.168.1.1
EXPIRE ratelimit:ip:192.168.1.1 60

# Distributed lock
SET lock:resource "1" NX EX 30    -- NX = only if not exists

# Job queue
RPUSH jobs "task1"
BLPOP jobs 0                       -- worker blocks until job arrives
```

---

## 15. Key Naming Convention

```
user:1                     -- single resource
user:1:sessions            -- nested
cache:users:list           -- cache prefix
lock:payment:42            -- distributed lock
ratelimit:ip:1.2.3.4       -- rate limit
```

---

## 16. Data Types Summary

| Type        | Use Case                          |
|-------------|-----------------------------------|
| String      | Cache, counters, tokens, flags    |
| Hash        | Objects, user profiles            |
| List        | Queues, feeds, stacks             |
| Set         | Unique tags, memberships          |
| Sorted Set  | Leaderboards, priority queues     |
| Pub/Sub     | Real-time events, notifications   |