# ZERO

Minimalist habit & fitness tracker. PWA. No frameworks, no build tools — one HTML file.

## Stack

- Vanilla HTML + CSS + JS
- [Supabase](https://supabase.com) — auth + database
- [Lucide Icons](https://lucide.dev) — via CDN
- [Claude API](https://anthropic.com) — AI coach (optional)

## Features

| Tab | What it does |
|-----|-------------|
| **Today** | Daily checklist (creatine, workout, weight, sleep, calories) + streak counter |
| **Log** | Body metrics (weight, sleep, calories, macros) + workout sets logger |
| **Stats** | 30-day averages, 14-day weight chart, recent workouts |
| **AI** | Chat with a Claude-powered fitness coach using your own API key |

## Running locally

No build step. Open `index.html` in a browser or serve it with any static server:

```bash
npx serve .
# or
python -m http.server 8080
```

## Supabase setup

Create a project at [supabase.com](https://supabase.com), then run this SQL in the editor:

```sql
create table profiles (
  id uuid references auth.users primary key,
  claude_api_key text,
  created_at timestamptz default now()
);

create table daily_logs (
  id uuid default gen_random_uuid() primary key,
  user_id uuid references auth.users not null,
  date date not null,
  weight numeric, sleep_hours numeric,
  calories int, protein int, carbs int, fat int,
  check_creatine bool default false,
  check_workout bool default false,
  check_weight bool default false,
  check_sleep bool default false,
  check_calories bool default false,
  unique(user_id, date)
);

create table workout_sessions (
  id uuid default gen_random_uuid() primary key,
  user_id uuid references auth.users not null,
  date date not null,
  exercise_name text not null,
  created_at timestamptz default now()
);

create table session_sets (
  id uuid default gen_random_uuid() primary key,
  session_id uuid references workout_sessions not null,
  set_number int, reps int, weight_kg numeric
);

create table ai_insights (
  id uuid default gen_random_uuid() primary key,
  user_id uuid references auth.users not null,
  date date not null,
  type text, content text,
  created_at timestamptz default now()
);

-- Enable RLS
alter table profiles enable row level security;
alter table daily_logs enable row level security;
alter table workout_sessions enable row level security;
alter table session_sets enable row level security;
alter table ai_insights enable row level security;

-- Policies
create policy "own profile" on profiles for all using (auth.uid() = id);
create policy "own logs" on daily_logs for all using (auth.uid() = user_id);
create policy "own sessions" on workout_sessions for all using (auth.uid() = user_id);
create policy "own sets" on session_sets for all using (
  session_id in (select id from workout_sessions where user_id = auth.uid())
);
create policy "own insights" on ai_insights for all using (auth.uid() = user_id);
```

Update the two constants at the top of `index.html` with your project credentials:

```js
const SUPABASE_URL = 'https://your-project.supabase.co'
const SUPABASE_ANON_KEY = 'your-anon-key'
```

## AI tab

The Claude API key is entered inside the app (AI tab → settings icon). It is stored in your `profiles` row in Supabase — never in localStorage or the source code. The key is only required when you send a message; login works without it.

## PWA

On mobile, use **Add to Home Screen** from your browser menu. The app runs fullscreen with no browser chrome.

## Color palette

| Token | Hex | Use |
|-------|-----|-----|
| Cream | `#E8DCC8` | Background |
| Card | `#D4C4A8` | Surfaces |
| Dark | `#1A1612` | Nav bar |
| Black | `#0D0B09` | Text, borders |
| Muted | `#6B5C47` | Secondary text |
| Success | `#3D2B1A` | Checked state |
