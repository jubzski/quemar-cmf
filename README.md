# QueMar CMF Tracker

A two-view web app for tracking Church Ministerial Fund (CMF) contributions, remittances, CBAP returns, and cash flow for the QueMar (Quezon Marinduque) Region Council, CBAP.

## Pages

- **`index.html`** — Church View. Public, read-only dashboard optimized for mobile (bottom tab nav, single-column stat cards, touch-friendly tables). Shows dashboard, churches, reports, returns, and cash flow.
- **`admin.html`** — Admin Portal. Desktop-optimized (top nav bar, multi-column grid layout) for treasurers to log in and manage contributions, remittances, CBAP returns, cash flow entries, and church list.
  - Default login: `admin` / `quemar2024` (change this in **Manage → Change Admin Password** after first login).

Both pages link to each other: the church view has a small 🔐 link in the header to reach the admin portal, and the admin portal has a "View Public Site" link back to the church view.

## Data

Both pages read/write the same [Supabase](https://supabase.com) project (see `SUPABASE_URL`/`SUPABASE_ANON_KEY` near the top of each `<script>` block). If Supabase is unreachable (e.g. opened via `file://`, or offline), the app falls back to browser `localStorage` so it still works standalone.

## Running locally

No build step — it's static HTML/CSS/JS. Serve the folder with any static file server, e.g.:

```bash
python3 -m http.server 8080
```

or via the npm script (same server used in production, see below):

```bash
npm install
PORT=8080 npm start
```

Then open `http://localhost:8080/index.html` (church view) or `http://localhost:8080/admin.html` (admin).

## Deploying to Railway

The repo is set up to deploy as-is:

- `package.json` installs [`serve`](https://www.npmjs.com/package/serve) and runs it on Railway's `$PORT`.
- `serve.json` disables `serve`'s default clean-URL redirects, so `/admin.html` and `/index.html` are served at their real paths (matching the links already in the pages) instead of 301-redirecting to `/admin` / `/index`.
- `railway.json` pins the Nixpacks builder and start command explicitly.

Steps:

1. Push this repo to GitHub (already done if you're reading this from the repo).
2. On [railway.app](https://railway.app), **New Project → Deploy from GitHub repo**, and authorize/select `jubzski/quemar-cmf`.
3. Pick the branch to deploy. Railway will detect `package.json`, run `npm install`, then `npm start`.
4. Once deployed, Railway assigns a `*.up.railway.app` domain automatically (or add a custom domain under the service's **Settings → Networking**). The public site is the domain root (`index.html`); the admin portal is `/admin.html` on the same domain.
5. No environment variables are required — the Supabase URL/anon key are already embedded in the HTML (see the Security note below for what that means).

## Security note

Admin login is a client-side check (password compared in the browser). This is convenient for a small static site but is **not real access control** — anyone can view source or edit `localStorage`. Treat the admin portal as a light deterrent, not a security boundary. For real protection, put it behind Supabase Auth/RLS or a server-side gate.
