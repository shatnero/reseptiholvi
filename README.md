# Recipe Box

A single-file recipe app: paste text, a link, or a photo, parse it into a
structured recipe with a local LLM via [LM Studio](https://lmstudio.ai), and
keep everything in Supabase. Share individual recipes with a plain link —
no login required for viewers.

No build step, no framework — `recipe-box.html` is the entire app.

## Setup

### 1. Supabase

Create a free project at [supabase.com](https://supabase.com), then run this
once in the **SQL editor**:

```sql
create table public.recipes (
  id uuid primary key default gen_random_uuid(),
  owner_id uuid not null default auth.uid(),
  title text, servings text,
  tags text[] default '{}',
  ingredients jsonb default '[]',
  instructions jsonb default '[]',
  source_type text, raw_input text,
  parsed boolean default true,
  shared boolean default false,
  created_at timestamptz default now()
);
alter table public.recipes enable row level security;
create policy "owner full access" on public.recipes for all
  using (auth.uid() = owner_id) with check (auth.uid() = owner_id);
create policy "public reads shared" on public.recipes for select
  using (shared = true);
```

Then go to **Authentication → Users → Add user** and create the one login
you'll use — there's no public sign-up.

### 2. Configure the app

Open `recipe-box.html` and fill in, near the top of the `<script>`:

```js
const SUPABASE_URL = 'YOUR_SUPABASE_URL';       // Project Settings → API
const SUPABASE_ANON_KEY = 'YOUR_SUPABASE_ANON_KEY'; // the "anon public" key
```

The anon key is meant to be public — row-level security (above) is what
actually restricts writes to the one owner account, and reads to recipes
marked `shared`.

### 3. Deploy

Push this repo and turn on **GitHub Pages** (Settings → Pages → Deploy from
branch). That URL is the app.

### 4. LM Studio (for parsing)

LM Studio only runs on a computer (macOS/Windows/Linux) — there's no iOS
version. In LM Studio: **Developer → Server Settings** → turn on
**Enable CORS** and **Enable HTTPS**.

In the app's own **Settings** screen, set the server URL:
- `http://localhost:1234/v1` — if you're browsing from that same computer
- `https://<computer's LAN IP>:1234/v1` — if you're parsing from your phone
  over the same Wi-Fi (HTTPS is required here or the browser blocks the
  request as mixed content)

If LM Studio isn't reachable, use **Save for later** — the raw text/photo is
queued and you can parse it next time you're near the computer.

## Usage

- **Add a recipe** — paste text/a link, or upload a photo, then parse and
  review before saving.
- **Share** — toggle "Share" on a recipe to make it readable at
  `yoursite.com?r=<id>` with no login.
- **Save as PDF** — print view on any recipe (owner or shared) via the
  browser's print dialog.

## Notes

- Single Supabase user by design — this isn't multi-tenant.
- No image storage yet — photos are only used transiently to parse a recipe,
  not saved.
