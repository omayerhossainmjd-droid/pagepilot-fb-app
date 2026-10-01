# PagePilot

Facebook Page management and automation through Meta’s official Graph API. PagePilot never asks for a Facebook password, never drives Facebook in a browser, and never invents unsupported endpoints.

## What it does

- Facebook Login (Meta OAuth) after you sign in to PagePilot
- Page discovery, switching, and encrypted Page tokens
- Text / link / photo / video publishing where Graph supports it
- Scheduling with retries, cancel, and drafts
- Comment inbox: reply, hide, delete
- Keyword automations and moderation rules
- Insights that Meta actually returns
- Activity logs and in-app notifications

Unsupported actions (including automatic reactions) are disabled with a clear explanation.

## 1. Create a Meta Developer account

1. Open [Meta for Developers](https://developers.facebook.com/).
2. Register as a developer.

## 2. Create a Meta App

1. Create an app of type **Business**.
2. Add **Facebook Login** (Facebook Login for Business is preferred for Page tokens).
3. Add **Webhooks** if you want live comment events.

## 3. Configure Facebook Login

Valid OAuth redirect URI:

```
https://YOUR-DOMAIN/api/meta/callback
```

Local development:

```
http://localhost:8080/api/meta/callback
```

## 4. Products and API access

Enable the Pages API products your app needs, then request App Review for live mode.

## 5. Redirect URLs

Set `META_REDIRECT_URI` to the callback above. The app can also derive it from the request origin.

## 6. App ID and App Secret

Copy the App ID and App Secret from the Meta dashboard. You can:

- Set them as server environment variables, or
- Paste them in **Settings** after signing in (encrypted at rest).

Never put secrets in client-side code.

## 7. Permissions to request

Required for core Page management:

- `pages_show_list`
- `pages_read_engagement`
- `pages_read_user_content`
- `pages_manage_posts`
- `pages_manage_engagement`
- `pages_manage_metadata`
- `read_insights`

Optional:

- `publish_video`
- `business_management`

Your Facebook user must be able to perform `CREATE_CONTENT`, `MODERATE`, and/or `ANALYZE` on the Page.

## 8. Environment variables

Do not commit real credentials. Configure these on the server:

```
META_APP_ID=
META_APP_SECRET=
META_API_VERSION=v22.0
META_REDIRECT_URI=
META_WEBHOOK_VERIFY_TOKEN=
DATABASE_URL=
REDIS_URL=
ENCRYPTION_KEY=
```

Notes:

- `META_API_VERSION` is read by the compatibility layer so you can bump Graph versions without rewriting the app.
- `ENCRYPTION_KEY` should be 32 bytes (or any passphrase; it is hashed to 32 bytes). Required in production.
- `DATABASE_URL` is a Postgres connection string. The live preview falls back to an embedded database.
- `REDIS_URL` is reserved for a Redis/BullMQ worker. The app ships a Postgres-backed job queue with the same retry/delay semantics so scheduling works without Redis.
- Webhook callback: `POST /api/meta/webhook` (signature verified with the app secret). Verify token is `META_WEBHOOK_VERIFY_TOKEN`.

## 9. PostgreSQL

Provision Postgres and set `DATABASE_URL`. Schema lives in `migrations/*.sql` and is applied on boot / build.

## 10. Redis (optional)

The scheduler is a delayed job table (`jobs`) processed in-process. To run BullMQ in production instead, set `REDIS_URL` and swap the adapter in `src/lib/services/scheduler.server.ts`.

## 11. Run migrations

Migrations apply automatically in preview. On a Postgres deploy they run during `npm run build` via `npm run db:migrate`.

## 12. Start the application

```
npm install
npm run dev
```

Sign in with Google, X, or email, then open **Settings** to connect Facebook Login.

## Security

- OAuth `state` is stored server-side and expires
- Access tokens are AES-256-GCM encrypted and never returned to the browser
- Server functions are session-scoped with `authMiddleware`
- Graph calls go through a compatibility catalog (version, permission, support, rate limit, error mapping, audit log)
- Webhooks require `X-Hub-Signature-256`

## Policy

PagePilot will not:

- Collect Facebook passwords
- Use Selenium / Playwright / cookies to operate Facebook
- Fake likes, reactions, or engagement
- Bypass CAPTCHAs or rate limits
- Call unofficial endpoints
