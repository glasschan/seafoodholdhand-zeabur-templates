# Multica for Zeabur

One-click deploy [Multica](https://github.com/multica-ai/multica) on Zeabur.

[![Deploy on Zeabur](https://zeabur.com/button.svg)](https://seafoodholdhand.com/recommends/zeabur-multica/)

## Why Caddy?

Prebuilt frontend images can't bake runtime env vars like `NEXT_PUBLIC_WS_URL`. Caddy solves this by putting everything behind one domain — WebSocket, API, and web all share the same origin. No CORS issues, no cross-origin WebSocket problems.

## Login

1. Enter your email on the login page
2. Get verification code:
   - **With Resend:** code sent to your email (set `RESEND_API_KEY`)
   - **Without Resend:** check Backend service logs for `[DEV] Verification code for`

## After Deployment

```bash
# Install CLI
brew install multica-ai/tap/multica

# Configure — both URLs point to the same domain
multica setup self-host \
  --server-url https://your-multica-domain.zeabur.app \
  --app-url https://your-multica-domain.zeabur.app
```

## Create by SEAFOODHOLDHAND

[seafoodholdhand.com](https://seafoodholdhand.com)
