# Unleash Cheatsheet

```bash
docker compose -f unleash.yaml up -d
```

→ [http://localhost:4242](http://localhost:4242)

> ⚠️ Your original `unleash.yaml` pointed `DATABASE_URL` to `localhost:5432`.
> Inside a container, `localhost` means the container itself — not your host
> or another container. Use the fixed compose file below (adds a `db`
> service and points to it by service name `db`).

---

## 1. First Login

| Field    | Value                    |
|----------|---------------------------|
| URL      | http://localhost:4242     |
| Username | `admin`                   |
| Password | `unleash4all` (forced change on first login) |

---

## 2. Core Concepts

```
Project      -- top-level grouping of flags (e.g. "default")
Feature Flag -- the on/off switch you check in code
Environment  -- dev / staging / production (each flag has a status per env)
Strategy     -- the rule that decides WHO the flag is on for
API Token    -- used by your app (client) or frontend to fetch flag state
```

---

## 3. Creating a Flag (Dashboard)

```
1. Open project → "New feature flag"
2. Name it, e.g. "new-checkout"
3. Leave OFF by default
4. Add a strategy under the environment you want to control:
   - Standard      -- simple on/off for everyone
   - Gradual rollout -- % of users (e.g. 10%, 50%, 100%)
   - UserIDs        -- target specific user IDs
   - IPs            -- target specific IPs
   - Custom constraints -- e.g. country = "BD", plan = "pro"
```

---

## 6. Strategy Types

| Strategy         | Use Case                                      |
|------------------|------------------------------------------------|
| Standard         | Simple flip, on for everyone or no one          |
| Gradual rollout  | % based rollout (10% → 50% → 100%)              |
| UserIDs          | Enable for specific accounts (e.g. your team)   |
| IPs              | Enable for specific networks/offices            |
| Constraints      | Target by custom fields (country, plan, etc.)   |

---

## 7. Backend SDK (Node.js)

```bash
npm install unleash-client
```

```js
const { initialize, isEnabled, destroy } = require("unleash-client");

const unleash = initialize({
  url: "http://localhost:4242/api/",
  appName: "my-app",
  customHeaders: {
    Authorization: "default:development.unleash-insecure-api-token"
  },
});

unleash.on("ready", () => {
  const enabled = isEnabled("new-checkout", { userId: "42" });
  console.log("new-checkout enabled:", enabled);
});

// on shutdown
// destroy();
```

---

## 8. Passing Context (for targeting rules)

```js
isEnabled("new-checkout", {
  userId: user.id,
  sessionId: session.id,
  remoteAddress: req.ip,
  properties: {
    country: "BD",
    plan: "pro",
  },
});
```

---

## 9. Exposing Flags to Frontend (via your own API)

```js
// backend route
app.get("/api/flags", (req, res) => {
  res.json({
    newCheckout: isEnabled("new-checkout", { userId: req.user.id }),
  });
});
```

```js
// frontend
const flags = await fetch("/api/flags").then(r => r.json());

if (flags.newCheckout) {
  renderNewCheckout();
} else {
  renderOldCheckout();
}
```

---

## 10. API Tokens (types)

```
Client token    -- used by backend SDK to read all flag rules
Frontend token  -- used by frontend SDK, more restricted (safe to expose to browser)
Admin token     -- full API access, used for automation/scripts
```

---

## 11. Useful Env Vars

```yaml
DATABASE_URL: "postgresql://user:pass@host:5432/unleash"
DATABASE_SSL: "false"                  # disable for local dev
INIT_FRONTEND_API_TOKENS: "default:development.<token>"
INIT_CLIENT_API_TOKENS: "default:development.<token>"
LOG_LEVEL: "warn"                       # error | warn | info | debug
```

---

## 12. Common Workflow (rollout pattern)

```
1. Create flag, OFF by default
2. Deploy code behind isEnabled() check   -- nothing changes yet
3. Enable for your team (UserIDs strategy)
4. Enable gradual rollout: 10%
5. Watch error tracking / metrics
6. Bump to 50%, then 100%
7. If broken at any point -> flip back to 0% or OFF instantly
8. Once stable at 100% -> remove flag + old code from codebase
```

---

## 13. Cleanup

```bash
docker compose -f unleash.yaml down        # stop containers
docker compose -f unleash.yaml down -v     # stop + delete volume (wipes DB)
```