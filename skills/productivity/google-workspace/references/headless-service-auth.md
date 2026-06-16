# Headless / Agent Environment Service Auth

How to authenticate various cloud PaaS and developer services from a
non-interactive (no TTY) Hermes agent environment. These techniques let you
connect external services so the agent can deploy, manage databases, and
orchestrate infrastructure on your behalf.

## General Principles

1. **Device-code OAuth** (best): Many CLIs support a device-authorization flow
   where a URL + code is printed, the user visits it in their own browser, and
   the CLI polls for completion. Works well with `background=true, pty=true`.
   Examples: Vercel, GitHub CLI.

2. **Personal Access Token** (reliable): Generate a token on the provider's
   dashboard, pass it via `--token` CLI flag or env var. Bypasses all
   browser-based flow. Examples: Supabase, Netlify, GitHub.

3. **OAuth PKCE** (most powerful): Google Workspace's `setup.py` handles the
   full OAuth PKCE dance non-interactively — generates URL, user pastes back
   the redirect, script exchanges for a token. Fully headless-compatible.

4. **Connection string** (simplest): Databases need no auth flow — just a
   connection URL. Examples: Neon, Supabase direct DB access.

## Service-Specific Notes

### Vercel
```bash
# Device code flow — use pty=true so the spinner polling works
terminal(command="vercel login", background=True, notify_on_complete=True, pty=True)
# The CLI prints "Visit https://vercel.com/oauth/device?user_code=XXXX-XXXX"
# User visits, authorizes, CLI picks it up automatically.
# Verify: `vercel whoami`
```

### Supabase
```bash
# Supabase CLI does NOT support device-code flow in non-TTY.
# You MUST use a personal access token.
# Generate at: https://supabase.com/dashboard/account/tokens
supabase login --token $SUPABASE_ACCESS_TOKEN
# Or set env var:
export SUPABASE_ACCESS_TOKEN="sbp_xxxx"
supabase login
# Verify: `supabase projects list`
```

### Netlify
```bash
# netlify-cli is extremely heavy (hundreds of deps, OOM-prone on small VMs).
# Prefer the REST API with a personal access token.
# Generate at: https://app.netlify.com/user/applications/personal
export NETLIFY_TOKEN="nfp_xxxx"
curl -H "Authorization: Bearer $NETLIFY_TOKEN" https://api.netlify.com/api/v1/user
curl -H "Authorization: Bearer $NETLIFY_TOKEN" https://api.netlify.com/api/v1/sites
```

### Cloudflare (Wrangler)
```bash
# wrangler login generates a device-like OAuth URL but NEEDS the user to visit
# it quickly (default callback server times out). Works but time-sensitive.
# Alternative: Use `wrangler login` with pty=true in background,
# send the URL to the user, have them authorize immediately.
terminal(command="wrangler login", background=True, notify_on_complete=True, pty=True)
# Verify: `wrangler whoami`
# Faster alternative: API token via Wrangler config:
# https://dash.cloudflare.com/profile/api-tokens
```

### Neon (PostgreSQL)
```bash
# No auth flow needed. Just install psql and use the connection string:
psql "postgresql://user:password@ep-xxxx.us-east-1.aws.neon.tech/dbname?sslmode=require"
# Note: the Neon pooled connection string may require `channel_binding=require`.
# Save to .env as: DATABASE_URL=postgresql://...
```

### Google Workspace
```bash
# Full OAuth PKCE flow through google-workspace/setup.py.
# Completely headless — no TTY needed.
# See: SKILL.md Step-by-step setup instructions.
```

## Troubleshooting

| Symptom | Likely Cause | Fix |
|---------|-------------|-----|
| CLI prints URL but never completes auth | Device code expired (15min window) or CLI process was not running in background | Generate new code, run with `background=True, pty=True` |
| npm install hangs / gets OOM-killed | Package too heavy for VM memory (netlify-cli, vercel) | Clear npm cache first (`rm -rf ~/.npm/_cacache`), install one at a time, or use API tokens instead |
| "non-TTY environment" error | CLI requires interactive terminal | Use `--token` flag, `--no-browser`, or env var to bypass |
| "Inappropriate ioctl for device" | Process backgrounded without PTY | Use `pty=True` in background terminal calls |
| 401/403 on API | Token expired, wrong scopes, or malformed | Regenerate token, verify it has correct permissions |
| `channel_binding=require` errors on Neon | Pooled connection string dropped `channel_binding` | Always include `&channel_binding=require` in the connection URL |