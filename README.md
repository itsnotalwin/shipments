# Shipment Tracker — Production PWA

A shipment & warehouse tracking app built as a single-page PWA.
Deployable to GitHub Pages in minutes. Backed by Supabase (free tier)
for multi-user logins, shared data, and automaticbackups.

---

## Repository structure

```
index.html          ← entire app (HTML + CSS + JS)
manifest.json       ← PWA manifest
service-worker.js   ← offline caching
supabase-schema.sql ← run once in Supabase SQL Editor to set up the DB
README.md
icons/
  favicon.ico
  apple-touch-icon.png
  icon-16.png  icon-32.png  icon-48.png  icon-72.png
  icon-96.png  icon-128.png icon-144.png icon-152.png
  icon-167.png icon-180.png icon-192.png icon-256.png
  icon-384.png icon-512.png
  icon-maskable-512.png
```

---

## Step 1 — Create a free Supabase project

1. Go to https://supabase.com and sign up (no credit card required).
2. Click **New Project** → choose a name and a region close to you
   (e.g. `eu-west-2` or `af-south-1`).
3. Wait ~2 minutes for provisioning.

---

## Step 2 — Run the database schema

1. In your Supabase project, click **SQL Editor** → **New Query**.
2. Paste the entire contents of `supabase-schema.sql` and click **Run**.
3. You should see "Success. No rows returned."

This creates the `shipments`, `warehouse_items`, `profiles`,
`organizations`, and `geocode_cache` tables, enables Row Level Security
(so users only ever see their own organisation's data), and sets up a
trigger that automatically creates an org + profile when someone signs up.

---

## Step 3 — Add your Supabase credentials to index.html

Open `index.html` and find these two lines near the top of the `<script>` block:

```js
const SUPABASE_URL = 'https://YOUR-PROJECT.supabase.co';
const SUPABASE_ANON_KEY = 'YOUR-ANON-PUBLIC-KEY';
```

Replace the placeholder values with your real credentials:

- **Project URL**: Supabase dashboard → Settings → API → Project URL
- **Anon public key**: same page → Project API keys → `anon` `public`

> These values are safe to commit — the anon key is public by design.
> Row Level Security in the database enforces all access control.

---

## Step 4 — Deploy to GitHub Pages

1. Create a new GitHub repo (e.g. `shipment-tracker`).
2. Push all files to the `main` branch root.
3. In the repo: **Settings → Pages → Source** → Branch: `main`, Folder: `/ (root)`.
4. GitHub will publish at `https://<username>.github.io/<repo>/`.
5. PWA install prompts require HTTPS — GitHub Pages provides this automatically.

> **Important:** after pushing, bump `CACHE_NAME` in `service-worker.js`
> (e.g. `v3`, `v4`…) with every future deployment so returning users
> get the new version instead of a cached old copy.

---

## Installing the app

| Platform | How |
|----------|-----|
| **Chrome / Edge (desktop)** | Address bar install icon, or menu → "Install Shipment Tracker" |
| **Android (Chrome)** | Menu → "Add to Home screen" / "Install app" |
| **iOS (Safari)** | Share → "Add to Home Screen" (runs full-screen with icon) |

---

## Inviting teammates to the same workspace

The first person to sign up at your company becomes the **admin** of a
new organisation. To add teammates:

1. Each teammate visits the app and creates their own account.
2. After they've signed up, find their new `org_id` by running this in
   the Supabase SQL Editor:
   ```sql
   select id, org_id from profiles where email = 'teammate@company.com';
   ```
3. Move them into your org:
   ```sql
   update profiles
   set org_id = '<your-org-id>'
   where email = 'teammate@company.com';
   ```

They'll now see the same shipments and warehouse data as you.
A built-in invite-link flow can be added as a future enhancement.

---

## Data & privacy

- All data is stored in your own Supabase project — Anthropic and the
  app author have no access to it.
- Row Level Security ensures each organisation's data is completely
  isolated from other organisations.
- Supabase free tier includes daily automated backups.
- The geocode cache (city → lat/lng) is shared within your org so the
  same city is never geocoded twice.

---

## Running without a backend (local-only mode)

If `SUPABASE_URL` still contains `YOUR-PROJECT`, the app skips the
login screen entirely and falls back to `localStorage` (the original
behaviour). Useful for local development or single-user setups where
cloud sync is not needed.

---

## Bumping the service worker after updates

Edit `service-worker.js`, line 3:

```js
const CACHE_NAME = 'shipment-tracker-v3'; // increment each deploy
```

Commit and push. Within ~24 hours (or on next visit + refresh) all
users will receive the updated app.
