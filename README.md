# webx

A self-hosted web hosting platform. Register any domain, publish HTML, and serve it directly — no path prefixes, no tricks. Just `http://yourdomain.com:3000/`.

---

## Quick start

```bash
npm install

# With sudo — enables real domain routing via /etc/hosts (recommended for local dev)
sudo node server.js

# Without sudo — everything still works via fallback URLs
node server.js
```

Open the dashboard at **http://localhost:3000**

---

## How domain routing works

webx reads the **Host header** on every incoming HTTP request. When you register `mysite.com`, the server:

1. Appends `127.0.0.1 mysite.com # WEBX-MANAGED` to `/etc/hosts` *(requires sudo)*
2. From that point on, any browser on the same machine visiting `http://mysite.com:3000/` gets that site's HTML served directly
3. No `/view/` prefix. No redirect. The domain itself is the address.
4. When you delete the site, the `/etc/hosts` entry is automatically removed.

**Without sudo:** a fallback URL is provided — `http://localhost:3000/__site__/mysite.com` — that always works regardless of DNS.

**In production:** point a wildcard DNS record `*.yourdomain.com → your server IP`. Every registered domain will resolve globally with no `/etc/hosts` needed.

---

## Domain support

webx accepts **any domain you type** with two exceptions:

- **Reserved names** — a small hardcoded list of major brands is blocked to prevent impersonation: `google`, `facebook`, `apple`, `amazon`, `microsoft`, `netflix`, `twitter`, `instagram`, `youtube`, `github`, `reddit`, `wikipedia`, `cloudflare`, `openai`, `anthropic`, `stripe`, `paypal`
- **Already taken** — first come, first served

Everything else is fair game. Examples of valid domains:

```
hello.com
mysite.io
whatever.hell.apple.sea.eater.com
a.b.c.d.e.f.net
x.y.z
coolblog.ai
```

### Supported TLDs

Standard: `.com` `.net` `.org` `.io` `.co` `.ai`

Custom: `.hell` `.void` `.dark` `.abyss` `.inferno` `.chaos` `.null` `.exe` `.death` `.doom` `.fire` `.shadow` `.blood` `.soul` `.sin` `.curse` `.hex`

These are shown as quick-pick chips in the UI. Any other TLD you type manually works too.

---

## Features

### Site creation
- Write raw HTML directly in the browser editor
- Template, style, and script snippet buttons
- Live preview before publishing
- Real-time domain availability check with suggestions for taken names

### Visibility
- **Public** — anyone can visit the site
- **Private** — only the owner can view it; everyone else gets error 666

### Site lifespan
Sites can be set to expire automatically after a chosen amount of:
- **seconds**, **minutes**, **hours**, or **days**

Or left as **permanent** with no expiry. Expired sites return error 1944 to all visitors.

### Incognito mode
Start a temporary browsing session with:
- A randomized session fingerprint
- Zero logs stored server-side
- Automatic destruction after 1 hour, or on demand via the "End session" button

### Dashboard
- View all your registered sites
- Live countdown timers on expiring sites
- Visit, edit, or delete any site
- View counters per site

### Explore
Public directory listing all non-private, non-expired sites on the platform.

---

## Error codes

| Code | Name | When it appears |
|------|------|-----------------|
| 404 | Not Found | Domain is not registered |
| 666 | Private | Site is private and visitor is not the owner |
| 1944 | Expired | Site's lifespan has ended |
| 21 | Too Young | Access denied |
| 6969 | Too Wholesome | Access denied |
| 67 | Protocol Mismatch | Request rejected by server |

All error pages are custom-styled. Preview any of them from the **Error codes** tab in the dashboard.

---

## API reference

All routes live under `/api/`. Authenticated routes require an `x-session-id` header with the session token returned at login.

### Auth

| Method | Route | Body | Description |
|--------|-------|------|-------------|
| POST | `/api/auth/register` | `{ username, password }` | Create an account |
| POST | `/api/auth/login` | `{ username, password }` | Sign in |
| POST | `/api/auth/logout` | — | Sign out |

### Domains

| Method | Route | Description |
|--------|-------|-------------|
| GET | `/api/domain/check?domain=foo.com` | Check if a domain is available |
| GET | `/api/domains/tlds` | List all suggested TLDs |

### Sites

| Method | Route | Body | Description |
|--------|-------|------|-------------|
| POST | `/api/sites/create` | `{ domain, html, isPrivate, expiresIn, expiresUnit }` | Publish a new site |
| GET | `/api/sites/my` | — | List your own sites |
| GET | `/api/sites/all` | — | List all public sites |
| GET | `/api/sites/html/:domain` | — | Fetch your site's HTML source |
| PUT | `/api/sites/:domain` | `{ html?, isPrivate? }` | Update a site |
| DELETE | `/api/sites/:domain` | — | Delete a site |

**`expiresUnit`** accepts: `seconds`, `minutes`, `hours`, `days`. Omit both `expiresIn` and `expiresUnit` for a permanent site.

### Incognito

| Method | Route | Body | Description |
|--------|-------|------|-------------|
| POST | `/api/incognito/start` | — | Start a session. Returns `{ id, token, expiresAt }` |
| DELETE | `/api/incognito/:id` | `{ token }` | Destroy the session immediately |

### Special routes

| Route | Description |
|-------|-------------|
| `http://<domain>:3000/` | Real domain routing (sudo / production DNS) |
| `/__site__/<domain>` | Fallback viewer — always works from localhost |
| `/__error_preview_<code>` | Preview a specific error page |

---

## Configuration

All options are set via environment variables:

| Variable | Default | Description |
|----------|---------|-------------|
| `PORT` | `3000` | Port the server listens on |
| `ADMIN_HOST` | `localhost` | Hostname used for the admin dashboard |
| `HOSTS_FILE` | `/etc/hosts` | Path to the hosts file for DNS injection |

Example:

```bash
PORT=8080 ADMIN_HOST=myserver.local sudo node server.js
```

---

## Project structure

```
webx/
├── server.js      # Node.js + Express backend
├── index.html     # Full frontend — single file, no build step required
└── package.json
```

No build tools. No frontend framework. No external database required — data lives in memory by default (a server restart clears all sites). To add persistence, replace the `db` Maps in `server.js` with MongoDB, Redis, or SQLite.

---

## Production deployment

1. **Database** — swap the in-memory `db` object in `server.js` for a real database so sites survive restarts
2. **DNS** — add a wildcard DNS record `*.yourdomain.com → server IP` so all registered domains resolve automatically
3. **Reverse proxy** — put nginx or Caddy in front; run Node as a non-root user (no sudo needed)
4. **HTTPS** — terminate TLS at the proxy with a wildcard certificate (e.g. via Let's Encrypt)
5. **Process manager** — use PM2 or systemd to keep the server running
