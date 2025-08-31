# Tokient Insight Stream - Backend Handoff (Loveable) — API Key Auth

This package adds **API key authentication** using a Bearer token in the `Authorization` header.

## Auth Model
- Header: `Authorization: Bearer <API_KEY>`
- Keys are stored in `api_keys` table with a `project_id` relationship.
- The server must verify the key, ensure it is `active`, and (if a `project_id` query/body is present) that the key is authorized for that project.
- Health endpoint is public; `/api/log` and `/api/stats` require auth.

## Env
```
REQUIRE_API_KEY=true
ALLOWED_ORIGINS=https://tokient-insight-stream.lovable.app
```
(See `.env.example` for full list.)

## Endpoints
- `GET /api/health` — public
- `POST /api/log` — requires `Authorization: Bearer <API_KEY>`
- `GET /api/stats?days=7&project_id=default` — requires `Authorization: Bearer <API_KEY>`

## Error Codes
- `401` — missing/invalid API key
- `403` — key valid but unauthorized for requested `project_id`

## Curl Examples
Replace `YOUR_API_KEY` and base URL as needed.

```bash
# Health (public)
curl -s https://tokient-insight-stream.lovable.app/api/health

# Log event (authorized)
curl -s -X POST https://tokient-insight-stream.lovable.app/api/log   -H "Authorization: Bearer YOUR_API_KEY"   -H "Content-Type: application/json"   -d '{
    "timestamp":"2025-08-31T12:00:00Z",
    "project_id":"default",
    "provider":"openai",
    "model":"gpt-4o",
    "tokens_in":800,
    "tokens_out":220,
    "cost_est":0.006,
    "status":"success"
  }'

# Stats (authorized)
curl -s "https://tokient-insight-stream.lovable.app/api/stats?days=7&project_id=default"   -H "Authorization: Bearer YOUR_API_KEY"
```

## DB Table (`api_keys`)
```
CREATE TABLE IF NOT EXISTS api_keys (
  key TEXT PRIMARY KEY,
  project_id TEXT NOT NULL,
  label TEXT,
  created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
  status TEXT DEFAULT 'active' -- 'active' | 'disabled'
);
```