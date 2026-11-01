# Supabase setup — complete beginner guide (about 20 minutes)

**The idea in one line:** your website is the *shop front*. Supabase is the *warehouse* behind it:
it keeps the customers (login), the goods (listings), the documents (files) and the rule book (security).
Right now the shop front has no warehouse connected. These steps connect one.

You only need: a web browser, this project folder open in VS Code, and a free Supabase account.
Nothing here costs money.

> Supabase sometimes renames menu items. If a name below is slightly different on your screen,
> pick the closest one — the idea is the same.

---

## Step 1 — Create the Supabase project (the warehouse)

1. Go to **https://supabase.com** → **Start your project** → sign up (GitHub or email).
2. Click **New project**.
3. Fill in:

| Box | What to type |
|---|---|
| Name | `rems` |
| Database password | Click **Generate a password**, then **copy it into a notepad**. You will rarely need it, but you cannot see it again. |
| Region | **South Asia (Mumbai)** (closest to you = fastest) |
| Plan | Free |

4. Click **Create new project** and wait 1–2 minutes until the dashboard opens.

## Step 2 — Build the tables, rules and file folders (one paste)

*Analogy: this is like giving the warehouse its shelves, its locks and its rule book.*

1. In the left sidebar click **SQL Editor** → **New query**.
2. In VS Code open **`supabase/setup_all.sql`** → press **Ctrl + A** (select all) → **Ctrl + C** (copy).
3. Back in Supabase paste with **Ctrl + V** in the big box → click **Run** (bottom right).
   If it asks "run destructive query?" click **Run this query** (it is only creating things).
4. You should see **Success. No rows returned.** *(A note saying `pg_cron is not available` is fine.)*

**Check it worked:** left sidebar → **Table Editor** → you should see tables like `properties`, `documents`, `visit_requests`.
Left sidebar → **Storage** → you should see two folders (buckets): `property-photos` and `property-documents`.

## Step 3 — Add the demo accounts and sample listings

1. **SQL Editor** → **New query** (a fresh empty one).
2. In VS Code open **`supabase/seed.sql`** → Ctrl + A → Ctrl + C → paste into Supabase → **Run**.
3. Success again. This creates 5 demo users, all with password **`Demo@1234`**:

| Email | Role | Use it for |
|---|---|---|
| vedant@rems.demo | Seller + Buyer | list a property, publish visit slots |
| dharmil@rems.demo | Buyer + Seller | request site visits |
| priya.agent@rems.demo | Agent | list on behalf of an owner |
| **admin@rems.demo** | **Admin** | verify documents, approve/reject |
| rohan@rems.demo | Seller | second owner |

> Run `seed.sql` **only once**. Never run it on a real, public project (shared password).

## Step 4 — Two login settings

1. Left sidebar → **Authentication** → **Sign In / Providers** (may be called *Providers*) → click **Email**.
   Turn **Confirm email** **OFF** → **Save**.
   *Why: otherwise Supabase waits for people to click an email link before they can log in.*
2. **Authentication** → **URL Configuration**:
   - **Site URL:** `http://localhost:8080` (use the address your browser shows when the site runs — your screenshot shows `localhost:8080`).
   - **Redirect URLs** → Add URL: `http://localhost:8080/reset-password` → Save.

## Step 5 — Copy the two connection values

1. Click the **gear icon — Project Settings** → **API Keys** (or just **API**).
2. You need exactly **two** things:

| What | Looks like | Where |
|---|---|---|
| **Project URL** | `https://abcdxyz.supabase.co` | top of the API page (under *Data API* / *Project URL*) |
| **anon / publishable key** | very long text starting `eyJ…` or `sb_publishable_…` | *API Keys* page |

3. ⚠️ **Never copy the `service_role` / `secret` key** into the website. It is the master key to the warehouse.
   The anon / publishable key is designed to be public — the security rules from Step 2 protect your data.

## Step 6 — Tell the website where the warehouse is

1. In VS Code, in the project's **root folder** (where `package.json` is), find **`.env.example`**.
2. Make a copy and rename the copy to exactly **`.env.local`** (starts with a dot, no `.txt`).
3. Edit it to look like this (use **your** values, no quotes, no spaces around `=`):

```
VITE_SUPABASE_URL=https://YOUR-PROJECT-REF.supabase.co
VITE_SUPABASE_ANON_KEY=PASTE-THE-LONG-ANON-KEY-HERE
VITE_SHOW_DEMO_ACCOUNTS=true
VITE_DEMO_PASSWORD=Demo@1234
```

Leave `SUPABASE_SERVICE_ROLE_KEY=` empty. Save the file (Ctrl + S).

## Step 7 — Start the website

1. In VS Code: **Terminal → New Terminal**.
2. First time only: `npm install` (wait until it finishes).
3. Run: `npm run dev`
4. Open the address it prints (e.g. `http://localhost:8080`).

⚠️ **Any time you edit `.env.local`, stop the site (Ctrl + C) and run `npm run dev` again.**
The site only reads that file when it starts.

If the page says **"One setup step left"**, the site cannot see `.env.local` — see the table below.

## Step 8 — Try your demo script

Open **Sign in**. Use the one-click demo buttons (or type the email + `Demo@1234`).
Then follow your 10-step script: Vedant creates and submits a listing → Ananya (admin) verifies the 4 documents and approves →
the listing shows in **Explore**. For site visits: Vedant opens a listing's slots page and publishes times →
Dharmil requests a visit → Vedant confirms.

---

## Optional: reminders 24 hours before a visit

Without this, reminders still arrive, but only when a user opens the bell icon. To send them on time:
**Database → Extensions** → search **pg_cron** → switch **on**. Then **SQL Editor** → New query → paste → Run:

```sql
select cron.schedule('rems-process-reminders', '*/5 * * * *', 'select public.process_due_reminders()');
```

## If something goes wrong

| What you see | Why | Fix |
|---|---|---|
| "One setup step left" screen | Website can't find `.env.local` | File must be in the root folder, named exactly `.env.local`; both values filled; **restart** `npm run dev` |
| "Invalid login credentials" | `seed.sql` not run, or wrong password | Run Step 3 once; password is `Demo@1234` |
| "Email not confirmed" | Confirm email still on | Step 4 → turn it off |
| Red error `relation … does not exist` | Step 2 not run completely | Run `setup_all.sql` again on a **new empty** project |
| `seed.sql` fails | Rare: Supabase auth table differs | Register 2 accounts on the site's **Register** tab, then in SQL Editor run: `update profiles set roles = array['admin']::app_role[] where email = 'YOUR-ADMIN-EMAIL';` |
| Photo / document upload fails | Storage folders missing | Storage page must show both buckets; if not, re-run Step 2 |
| "View file" doesn't work on a *sample* document | Sample documents have no real file | Normal. Documents **you upload** open fine |
| Site feels slow | `npm run dev` is the slow development mode and the first page load compiles everything | Wait for the first load; close other heavy tabs. The heavy 3D effects were removed |

## Security rules to remember
- Put `.env.local` in `.gitignore` (already done) — never upload it to GitHub.
- Never paste the `service_role` key into the website or share it.
- Before a real launch: delete the demo accounts and don't use `seed.sql`.
