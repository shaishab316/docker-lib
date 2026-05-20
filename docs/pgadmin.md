# pgAdmin Cheatsheet

```bash
docker compose -f pgadmin.yaml up -d
```

→ [http://localhost:5050](http://localhost:5050)

| Field | Value |
|-------|-------|
| Email | admin@admin.com |
| Pass  | admin |


---

## Connect to PostgreSQL

| Field    | Value                |
|----------|----------------------|
| Host     | host.docker.internal |
| Port     | 5432                 |
| Username | your_pg_user         |
| Password | your_pg_password     |