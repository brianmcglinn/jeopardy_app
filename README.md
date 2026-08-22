# jeopardy-receiver

Custom Chromecast receiver for the Jeopardy app. Single static HTML file — no
build step.

## Setup

1. **Fill in Supabase config** near the top of `index.html`:
   ```js
   const SUPABASE_URL = 'https://YOUR-PROJECT.supabase.co';
   const SUPABASE_ANON_KEY = 'YOUR-ANON-KEY';
   ```
   The client is already set to `db: { schema: 'jeopardy_app' }`, matching
   the sender. If you ever rename the schema, update it in both places —
   this file's `createClient` call and `jeopardy-sender/src/lib/supabase.ts`.
   Also make sure `jeopardy_app` is added to `PGRST_DB_SCHEMAS` on your
   self-hosted instance (see the top-level README) — without that, every
   request from this receiver will 404.
2. **Deploy to GitHub Pages** — same as draft-receiver/awards-receiver:
   ```bash
   git init
   git add .
   git commit -m "Initial receiver"
   git remote add origin https://github.com/<you>/jeopardy-receiver.git
   git push -u origin main
   ```
   Then enable Pages on the repo (Settings → Pages → deploy from `main`).
   Your receiver URL will be `https://<you>.github.io/jeopardy-receiver/`.

3. **Register a Custom Receiver app** in the
   [Google Cast Developer Console](https://cast.google.com/publish):
   - Application type: Custom Receiver
   - Receiver Application URL: your GitHub Pages URL from step 2
   - Copy the generated **App ID** — this goes into the sender app's
     cast-launcher config (`CAST_RECEIVER_APP_ID` in `jeopardy-sender`).
   - New receivers can take a few minutes to propagate; unpublished/dev-mode
     apps only cast to Chromecast devices registered to your Google account.

## Namespace

The receiver listens for a custom message on:
```
urn:x-cast:com.mcglinn.jeopardy
```
carrying `{ gameId: "<uuid>" }`. Change this constant in `index.html` if you'd
rather use a different namespace convention — just make sure the sender's
cast-launcher sends on the same one.

## What drives the screen

The receiver doesn't hold any game logic itself — it's a dumb display that
subscribes to Supabase Realtime on `game_state`, `game_clues`, `players`, and
`final_jeopardy_responses` for the given `gameId`, and re-renders based on
`game_state.phase`. All control happens from the sender app.
# jeopardy_app
