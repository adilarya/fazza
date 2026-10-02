# Fazza (fazza.world)

A public leaderboard that ranks people by **verified** donations to vetted Palestine relief
charities. Donors give on the charity's own site, upload the receipt here, an admin approves it,
and only approved donations count toward rank. Alongside the global board, users create private
invite-only groups to compete with friends, family, or their community.

---

## ⚠️ Read this first — every session

**At the start of a session:**

1. Read this entire file. It is the single source of truth.
2. Read **Current state** below to learn what exists, what's next, and what's blocked.
3. Read **Decisions** before proposing anything — those questions are already settled.
4. Run `git log --oneline -10` and `git status` to confirm the file reflects reality. If it
   doesn't, fix this file first and tell the user it had drifted.

**Before reporting any task complete:**

1. Update **Current state** — move finished items into Built, set the new Next.
2. Append to **Decisions** if a non-obvious choice was made. Append only; never edit old entries.
3. Add to **Open questions** anything that needs the user and move resolved ones out.
4. Run `npm run check` (lint + typecheck + build). It must pass.

Updating this file is part of the task, not an optional extra. A change isn't done until
CLAUDE.md describes the world after it.

**The filename must stay `CLAUDE.md`, uppercase.** Claude Code only auto-loads that exact name on
Linux. It was previously `claude.md` and was silently never loaded.

### 🚫 Never touch git history — Adil only

**Only Adil runs `git add`, `git commit`, `git push`, or anything else that writes to the
repository or the remote.** This is absolute and has no exceptions — not "it's a tiny change", not
"the user seemed to want it committed", not after a successful build, not when asked to "finish
up". Do not stage files. Do not amend. Do not create branches, tags, stashes, or worktrees. Do not
`git restore`, `git checkout -- .`, `git reset`, or `git clean` — those destroy uncommitted work
that is the only copy.

Read-only git is fine and encouraged: `git status`, `git log`, `git diff`, `git show`.

When work is ready, **say so and stop.** List the files changed and let Adil commit. If Adil
explicitly asks for a commit in the moment, still don't run it — write out the exact command for
him to run instead.

---

## Current state

_Last updated: 2026-10-02_

**Built:** Nothing. The repo contains an empty `README.md` (the only committed file), `logo.png`,
and this file. No `package.json`, no `src/`, no `supabase/`. `logo.png` and `CLAUDE.md` are
untracked — committing them is Adil's to do.

**Infrastructure:** `fazza.world` is registered and managed through **Cloudflare** — DNS lives
there, so the Vercel deployment is pointed at it via Cloudflare DNS records. Nothing else exists:
no Supabase project, no Vercel project. Both need the user; don't attempt to provision accounts.

**Next:**

1. Scaffold the npm-workspaces monorepo: `apps/web` (Next.js), `packages/shared`, `supabase/`.
   Leave `apps/mobile` out until the website works. Move `logo.png` to
   `apps/web/public/brand/logo.png` as part of this.
2. First migration: `profiles`, `charities`, `donations` + RLS.
3. Auth (magic link), then `/submit`, then `/admin`, then `/leaderboard`.
   `donation_intents` and `/submit` prefill come with `/submit`; the reminder email can wait.
4. Groups come after the single-player loop works end to end.

**Blocked on the user:**

- The charities list (see Open questions). Do not put real charity names in seed data until
  the user supplies and vets them — use obviously fake placeholders.
- Supabase project creation and keys.
- Vercel project creation, and adding the Cloudflare DNS records it asks for.
- Charity outreach. Webhook verification is the intended end state and it needs charity
  cooperation, so this unblocks more than any amount of code.

---

## Decisions

Append-only, newest last. Each line is settled — do not reopen without asking.

| Date | Decision | Why |
|---|---|---|
| 2026-10-02 | Fazza never touches money | Legal and trust. No payments, ever. |
| 2026-10-02 | Dark theme is the default look | Near-black ground, green as accent, echoing the logo's 3D extrusion. |
| 2026-10-02 | A native app is planned (React Native / Expo) | Shares TypeScript, types and the Supabase client with web. Screens are rebuilt, not reused. |
| 2026-10-02 | Business logic lives in Postgres functions + RLS, **not** Server Actions | Web and app call the same Supabase backend. One implementation, one security audit, no client can bypass a rule. Supersedes the old "prefer Server Actions" guidance. |
| 2026-10-02 | Monorepo from day one | Cheapest to set up before any code exists. |
| 2026-10-02 | Magic link is the only auth method | No passwords to leak. No third-party sign-in means Apple's guideline 4.8 never applies, so no Apple Sign In is required for the app. |
| 2026-10-02 | Private invite-only groups are in v1 | They are the viral loop, not a nice-to-have. |
| 2026-10-02 | An approved donation counts everywhere automatically | Global board and every group the user belongs to. Simplest mental model. |
| 2026-10-02 | Anonymity is a default plus per-scope override | A user can be visible to their family group and anonymous globally. |
| 2026-10-02 | Anonymous rows show "Anonymous" and do not link to a profile | Strongest privacy reading. No pseudonyms, no stable alias. |
| 2026-10-02 | Unique `@handle` chosen at signup, public profile at `/u/[handle]` | Short and shareable; display name stays separately editable. |
| 2026-10-02 | Each tab is its own page | The site is not one long scroll. Home is only the explainer for newcomers. |
| 2026-10-02 | Two different nav bars, signed out vs signed in | Signed-out users have nothing to show in Groups or Me. |
| 2026-10-02 | Leaderboard has exactly two views: All-time and Year | No month or week board. |
| 2026-10-02 | Leaderboard rows show rank, name, total only | Badges and detail live on the profile page. |
| 2026-10-02 | Year badges use tiered bands | Top 3 medals, top 10 gold, top 50 silver, top 100 bronze. Many people earn something, so many people keep playing. |
| 2026-10-02 | Rejected donations are fixed by editing the same row | No second row is created, so an honest retry never trips the duplicate-receipt constraint. |
| 2026-10-02 | Receipts are kept as long as the donation counts | They are the evidence behind a public rank. |
| 2026-10-02 | FX rate pre-filled from a rate table, admin can override | Accuracy with a human backstop. |
| 2026-10-02 | Rank ties broken by who reached the total first | Rewards the earlier giver and keeps ranks stable. |
| 2026-10-02 | The first admin is set by hand in a migration | Never promotable through the UI. |
| 2026-10-02 | Volunteering is a planned pillar, not v1 | Donations only for the first release. |
| 2026-10-02 | Verification is manual in v1, but the schema is built for all three paths | `verification_method` and a nullable `receipt_path` from day one, so email and webhook verification slot in without reworking the table. |
| 2026-10-02 | Platform webhooks are the preferred end state, DKIM email the fallback | Webhooks need charity cooperation but give instant, forgery-proof verification with no receipt at all. DKIM is stronger than an upload but needs a parser per charity. |
| 2026-10-02 | Track donate click-outs to prefill `/submit` and send one reminder | Peak motivation is at donation time; the current flow spends it all on re-typing. Prefill plus a nudge recovers donations that would otherwise never be submitted. |
| 2026-10-02 | `fazza.world` is on Cloudflare | DNS is managed there; Vercel is pointed at it with Cloudflare records. |
| 2026-10-02 | Rank stays ordered by USD total | A charity effectiveness rating (1–5) will later appear beside the money, but it does not reorder the board. |
| 2026-10-02 | Charity effectiveness rating 1–5 is planned, not v1 | Shown on the charity page and beside the amount on the leaderboard. Mechanics undecided — see Open questions. |
| 2026-10-02 | In a group, each member starts at 0 from the moment they join | Scoring cutoff is `greatest(group_members.joined_at, groups.starts_at)`. No retroactive credit for donations made before joining. |
| 2026-10-02 | Admins may approve their own donations in v1 | Accepted risk while review is manual and the admin is the operator. **Once review is automated, self-approval must be blocked.** |
| 2026-10-02 | Account deletion is a full hard delete | Rank, donations, receipts and all user data go. |
| 2026-10-02 | Handles are permanent | Chosen once at signup, never changeable, so `/u/[handle]` links never break and freed handles can't be squatted. |
| 2026-10-02 | Year boards are browsable by year, and placements freeze | The Year tab has a year picker; `year_placements` is written at year end so a late approval can't rewrite a past podium. |
| 2026-10-02 | There is a minimum donation amount, and a submission cap | Nothing else stops someone flooding the review queue with one-cent receipts. |
| 2026-10-02 | Account deletion releases the transaction IDs it held | The duplicate-receipt check does not apply to orphaned receipts. A surviving hash table was considered and rejected: it would still be pseudonymised personal data and would undercut "everything goes". |
| 2026-10-02 | Impact score = `Σ(amount_usd_cents × charity rating)` | Per-donation weighting. A $1 gift to a 5★ charity adds 5, so it cannot multiply an unrelated $10,000 total the way a flat average across charities would. |
| 2026-10-02 | A donor's displayed rating is `impact_score ÷ total_usd_cents` | An amount-weighted average, shown as one 1–5 figure. |
| 2026-10-02 | **Two separate boards: Money and Impact** | Supersedes "Rank stays ordered by USD total". One ranked by raw USD, one by impact score, so neither metric has to compromise. |
| 2026-10-02 | Ratings are applied **live**, never frozen onto the donation | Re-rating a charity recomputes every score that involves it, including past ones. Accepted knowingly: it means a re-rating reshuffles historical impact standings. |
| 2026-10-02 | `year_placements` still freezes rank and tier | Badges are awarded at year end and stay awarded, even though the live score behind them can move. A badge already earned is never taken back. |
| 2026-10-02 | `/leaderboard` is Money/Impact as primary tabs, All-time/Year as a secondary row | They are two different questions, not four lists. Keeps the year picker out of the main tab strip and survives mobile. |
| 2026-10-02 | Ratings are derived from external evaluators and cited | Not Fazza's own judgement. A public claim that one relief charity is less effective than another needs a source the user can check. |
| 2026-10-02 | Year badges are awarded on both boards, separately | A 2026 Money gold and a 2026 Impact gold are different badges, so `year_placements` is keyed by board as well as year. |
| 2026-10-02 | Charities with no evaluator coverage get an internal provisional rating | Otherwise good organisations that nobody has rated could never be listed at all. Researched with Claude's help, entered by an admin, and always labelled as Fazza's own estimate rather than an external score. |
| 2026-10-02 | An external rating always supersedes an internal one | When an evaluator finally covers a charity, that rating replaces the estimate. Because ratings are live, every impact score involving that charity recomputes automatically. |
| 2026-10-02 | External and provisional ratings render differently | Filled stars plus the evaluator's name versus outlined stars marked "provisional · Fazza estimate" with a link to the reasoning. Distinguishable at a glance, not only in small print. |
| 2026-10-02 | Rating history is kept internally, never shown on the charity page | The page shows only the current rating and its method. History exists for audit and to generate change notifications. |
| 2026-10-02 | Donors materially affected by a rating change are emailed an explanation | The message explains what changed and why, not that the person fell. |
| 2026-10-02 | Leaderboards animate — rows move, numbers roll | Rank changes are the product's feedback loop; they should be felt, not just rendered. Must honour `prefers-reduced-motion`. |
| 2026-10-02 | Rating a charity follows a documented admin runbook | See Admin runbook. Keeps a contestable judgement consistent and auditable. |
| 2026-10-02 | **Only Adil runs git write commands** | No agent stages, commits, pushes, branches, resets or cleans. Read-only git only. |

---

## Open questions

Ask the user; don't invent answers.

- **Which charities?** The user will supply the vetted list. Until then, placeholders only.
- **Group limits.** Max members per group, max groups a user can create or join. Unset.
- **Volunteering model.** Planned, but how hours are proven and scored is undecided. Keep the
  schema free of assumptions that would block it.
- **Admin review SLA.** How fast is a receipt expected to be reviewed? Affects the copy on `/me`.
- **Minimum donation amount.** Agreed there must be one; the value isn't set. Suggest a ~$1 USD
  equivalent plus a per-day submission cap.
- **Which charities will cooperate?** The user intends to reach out. Cooperation unlocks campaign
  pages and webhook verification; without it, Fazza is just a public referrer.
- **Reminder email cadence.** One nudge is decided. Whether there's ever a second, and how users
  opt out, is not.

---

## The money rule

Fazza holds no funds, takes no cut, and processes no payments. Every "Donate" button links out to
the charity's own donation page.

**Never add payment processing, wallets, Stripe, crypto, or any path for money to pass through this
site.** This is not a technical preference. It is what keeps Fazza out of charity-regulation and
money-transmission territory. If a request seems to require it, stop and ask.

---

## Stack

- **Monorepo** with npm workspaces. Package manager is npm.
- **Web:** Next.js (App Router, TypeScript, Server Components by default) on Vercel
- **Mobile (later):** React Native via Expo
- **Backend:** Supabase — Postgres, Auth, Storage, Row Level Security
- **Styling:** Tailwind CSS (web). Mobile does not share Tailwind — share tokens, not classes.

## Commands

```bash
npm run dev            # web dev server at localhost:3000
npm run build          # must pass before any commit
npm run lint
npm run typecheck      # tsc --noEmit across workspaces
npm run check          # lint + typecheck + build — run this before saying a task is done

npx supabase start     # local Supabase (needs Docker)
npx supabase migration new <name>
npx supabase db reset  # re-apply all migrations locally
npx supabase gen types typescript --local > packages/shared/src/database.types.ts
```

Regenerate types after **every** migration. `database.types.ts` is generated — never hand-edit it.

---

## Repo layout

```
apps/
  web/                    Next.js site
    src/app/              routes (see Pages below)
    src/components/
    src/lib/supabase/     server.ts, client.ts, middleware.ts
  mobile/                 Expo app — not scaffolded yet
packages/
  shared/
    src/database.types.ts generated by Supabase CLI, never hand-edit
    src/schemas/          zod schemas shared by web, mobile and server
    src/money.ts          currency formatting and minor-unit helpers
    src/tokens.ts         colors and spacing, consumed by both clients
supabase/
  migrations/             all schema changes, SQL only
  seed.sql                placeholder charities for local dev
logo.png                  at repo root for now — move to apps/web/public/brand/
```

Anything both clients need goes in `packages/shared`. If you find yourself about to write the same
validation rule twice, it belongs there — or in Postgres.

---

## Architecture

**All writes go through Postgres functions guarded by RLS.** The website and the app are both thin
clients calling the same Supabase backend. Do not put business rules in Next.js Server Actions —
the mobile app cannot call them, and duplicated rules drift apart and become security holes.

- Reads: RLS-protected tables and views, queried directly by the client.
- Writes: `security definer` SQL functions (`submit_donation`, `resubmit_donation`, `join_group`,
  `create_group`, `approve_donation`, `reject_donation`, `set_anonymity`).
- Server Components are still the default for rendering the web pages. The rule is about where
  *rules* live, not where HTML is produced.

Keep mobile viable with every decision: no web-only auth flows, no logic reachable only over HTTP
from Vercel, and magic-link redirects configured for deep links as well as URLs.

---

## Data model

Money is **always integers in minor units**, never floats. Ranking uses `amount_usd_cents`, fixed
at approval time, so ranks never drift when exchange rates move.

**`profiles`** — `id` (= `auth.users.id`), `handle` (unique, case-insensitive, reserved-words
blocked), `display_name`, `default_anonymous` (bool, default false), `global_anonymous` (bool,
**nullable** — null means inherit the default), `is_admin` (bool, default false), `created_at`

**`charities`** — `id`, `name`, `slug`, `donate_url`, `logo_url`, `is_supported` (bool),
`receipt_notes` (what a valid receipt from this charity looks like), `effectiveness_rating`
(smallint 1–5), `created_at`

Rating provenance lives alongside it:

- `rating_origin` — `external` | `internal_estimate`
- `rating_source`, `rating_source_url` — the evaluator and the page a user can go and check.
  Required when `rating_origin = 'external'`, null otherwise.
- `rating_rationale` — why this number. Shown publicly. Required for an internal estimate.
- `rated_at`

> `effectiveness_rating` is **not null**, so there is no unrated case to handle in scoring, and a
> charity must be rated before `is_supported` can be set true.
>
> **Prefer an external evaluator.** Where one covers the charity, use its rating and cite it. Where
> none does, an admin enters a provisional `internal_estimate` — researched with Claude's help —
> so that charities nobody has evaluated can still be listed. An external rating, once available,
> always supersedes the estimate.
>
> **An internal estimate must never be presented as an external score.** It carries a visibly
> different label and shows its rationale. The point of sourcing ratings externally is that a
> public claim about a charity's effectiveness can be checked; an estimate that looks like an
> evaluator's verdict would quietly defeat that.
>
> Because ratings apply live rather than being copied onto donations, changing one immediately
> moves every impact score involving that charity — and therefore every affected donor's position
> on the Impact board. Treat re-rating as a significant action, not a routine edit.

**`donations`** — `id`, `user_id`, `charity_id`, `amount_minor` (int), `currency` (ISO 4217),
`amount_usd_cents` (int, set at approval), `transaction_id`, `donated_at`,
`status` (`pending` | `approved` | `rejected`), `reject_reason`, `reviewed_by`, `reviewed_at`,
`created_at`, `updated_at`, plus the verification columns:

- `verification_method` — `manual_receipt` | `email_dkim` | `platform_webhook`
- `receipt_path` (Storage key, **nullable**) — only `manual_receipt` rows have one
- `verification_ref` (text, nullable) — the email Message-ID, or the platform's own transaction id
- `auto_verified_at` (nullable) — set when a machine approved it rather than a person

Only `manual_receipt` is implemented in v1. The other two values exist so adding them later is a
code change, not a painful migration. `reviewed_by` stays null on an auto-verified row.

> **Unique constraint on `(charity_id, transaction_id)`.** This is the core anti-cheat rule.
> Never drop or weaken it. Because a rejection is fixed by editing the existing row rather than
> inserting a new one, an honest retry never collides with it.
>
> Known edge case: if someone submits a transaction ID that isn't theirs and is rejected, the real
> donor is locked out of that ID. The escape hatch is an admin voiding the row. Build that into
> `/admin` when the case first comes up — don't pre-build it.
>
> **Account deletion releases transaction IDs, by design.** A hard delete removes the rows holding
> them, so receipts a deleted account had claimed become submittable again. This is accepted: the
> check exists to stop one receipt counting twice on a live board, and once the account and its
> rank are gone there is nothing left to double-count. A surviving hash table was considered and
> rejected — it would still be pseudonymised personal data tied to a transaction, which would
> undercut the promise that deletion removes everything.

**`charity_rating_history`** — `id`, `charity_id`, `rating`, `rating_origin`, `rating_source`,
`rating_source_url`, `rating_rationale`, `effective_from`, `changed_by`. **Admin-only — never
exposed publicly or on the charity page.** It exists to audit a contestable judgement and to work
out who to notify when a rating moves.

**`donation_intents`** — `id`, `user_id`, `charity_id`, `clicked_at`, `donation_id` (nullable),
`nudged_at` (nullable). Written when a signed-in user clicks through to a charity's donate page.
Powers `/submit` prefill and the single reminder email. This is first-party only — no third-party
analytics, no tracking pixels, and nothing recorded for signed-out visitors.

**`groups`** — `id`, `name`, `join_code` (unique, unguessable), `created_by`, `starts_at`
(nullable), `ends_at` (nullable), `goal_usd_cents` (nullable), `created_at`

**`group_members`** — `group_id`, `user_id`, `role` (`owner` | `member`), `is_anonymous` (bool,
**nullable** — null means inherit the profile default), `joined_at`

> `joined_at` is a **scoring cutoff, not just a timestamp.** Everyone starts a group at zero, so a
> donation only counts toward a group if it was made after that member joined. This is the one
> place where "an approved donation counts everywhere automatically" is qualified — it always
> counts globally, but inside a group only from the moment you were in it.

**`fx_rates`** — `date`, `currency`, `usd_rate`, `source`, `fetched_at`. Pulled daily; the admin
can override the pre-filled value at approval. Store whichever rate was actually used.

**`year_placements`** — `user_id`, `year`, `board` (`money` | `impact`), `rank`, `tier`,
`score`, `finalized_at`. Primary key is `(user_id, year, board)` — each board awards its own
badges. Frozen at year end so neither a late approval nor a charity re-rating can rewrite a past
podium. **An awarded badge is never revoked**, even if today's live score would place the holder
differently.

### Views

- `leaderboard_alltime`, `leaderboard_year` — global **Money** boards, ordered by USD total
- `impact_alltime`, `impact_year` — global **Impact** boards, ordered by
  `Σ(amount_usd_cents × charities.effectiveness_rating)`, with the displayed 1–5 figure being that
  score divided by the USD total
- `group_leaderboard` — per-group. A donation counts for a member only when:

```
donated_at >= greatest(group_members.joined_at, coalesce(groups.starts_at, '-infinity'))
and donated_at <= coalesce(groups.ends_at, 'infinity')
```

Every board view returns `display_name` **or** the literal `"Anonymous"`. A view must **never**
expose `user_id` or `handle` for a row that resolved to anonymous — that's what makes the row
unlinkable to a profile.

### Anonymity resolution

```
global board:   coalesce(profiles.global_anonymous, profiles.default_anonymous)
group board:    coalesce(group_members.is_anonymous, profiles.default_anonymous)
```

Nullable means "inherit". Changing the default must immediately affect every scope that hasn't
been explicitly overridden. Anonymity changes take effect on the leaderboard immediately.

### Badge tiers

Top 3 get gold, silver and bronze medals. Then top 10 is gold tier, top 50 silver, top 100 bronze.
Badges appear on profile pages only — never in a leaderboard row.

---

## Security rules

- **RLS on every table.** Users read and insert only their own donations, and can never write
  `status`, `amount_usd_cents`, or any review field. Only admins (checked via `profiles.is_admin`
  inside the policy, never in client code) approve or reject.
- **The receipts bucket is private.** Users upload to `receipts/{user_id}/...` and read only their
  own files. Admins read through short-lived signed URLs. Receipts contain names, emails and
  payment details — never make the bucket public, and never put a receipt URL in a shared page.
- **`SUPABASE_SERVICE_ROLE_KEY` is server-only.** Never import it into a client component, never
  bundle it into the Expo app, never prefix it with `NEXT_PUBLIC_`.
- Validate uploads server-side: PDF, PNG or JPG only, max 5 MB. Check the actual file, not the
  extension.
- Validate every input with zod **on the server**, regardless of what the client checked. Share
  the schema from `packages/shared/src/schemas`.
- Rate-limit receipt submissions per user.
- Group `join_code` must be unguessable — random, not sequential, not derived from the name.
- Never commit `.env*`. Required variables are listed in `.env.example`.

---

## Pages and navigation

Each tab is its own route. The site is not one long scroll.

```
SIGNED OUT   فزعة   LEADERBOARDS   CHARITIES   LEARN MORE        [ SIGN IN ]
SIGNED IN    فزعة   LEADERBOARDS   CHARITIES   GROUPS   ME       [ SUBMIT ]
```

Submit is a filled green button, not a text tab — it's the one action the site exists to produce.
On mobile this becomes a bottom bar with Submit as the centre button.

| Route | Purpose |
|---|---|
| `/` | Explainer for newcomers only. Pitch, how it works in brief, a **Learn more** link. Not a leaderboard. |
| `/learn` | The trust story in full: human verification, vetted charities, and that Fazza never touches the money. |
| `/leaderboard` | **Money** and **Impact** as primary tab-style views; **All-time** (default) and **Year** as a secondary row, with a year picker for past years. Rows: rank, name, total — plus the 1–5 figure on the Impact board. |
| `/charities` | Flat list of supported charities, each linking out to its own donation page. No per-charity detail pages, no individual causes. Unsupported charities show as "coming soon". |
| `/groups` | The user's groups, plus create and join. Signed out, this is a real page explaining groups with a sign-in call to action — not a redirect. |
| `/groups/[code]` | A group's board and progress toward its goal. Non-members see a teaser only: name, goal, member count. Names and amounts are members-only. |
| `/submit` | Charity, amount, currency, transaction ID, date, receipt upload. |
| `/me` | Own rank, donation history with statuses, rejection reasons with **Fix and resubmit**, handle, display name, and anonymity settings per scope. |
| `/u/[handle]` | Public profile: verified total, year badges. Never reachable from an anonymous row. |
| `/admin` | Review queue. Admins only, enforced in RLS and not merely by hiding the link. |

People will land on `/groups` and `/me` from shared links while signed out. Those pages need a
genuine signed-out state.

---

## How donations work

**Fazza is a scoreboard bolted onto other people's donation pages.** The user gives on the
charity's own site with their own card; Fazza only ever records *proof* that it happened. Nothing
below changes that — these are all ways of checking a donation, never of taking one.

### The v1 flow (manual)

1. Signed in, the user clicks Donate on `/charities`. A `donation_intents` row records the charity
   and time, then they're sent to the charity's own page.
2. They donate there and the charity emails them a receipt.
3. They return to `/submit`. **Charity and date are already filled in** from the intent — they only
   enter amount, currency and transaction ID, and upload the receipt.
4. The row is created as `pending`. A duplicate `(charity_id, transaction_id)` returns a clear
   "this receipt has already been submitted" message.
5. An admin opens `/admin` and views the receipt beside the entered fields. The USD value is
   pre-filled from `fx_rates` and can be overridden. They approve, or reject with a reason.
6. On rejection the reason appears on `/me` with the fields editable. The user corrects and
   resubmits **the same row**, which returns to `pending`.
7. Only approved donations appear on any leaderboard.

If an intent has no matching donation after ~24 hours, send **one** reminder email with a link
straight to the prefilled form. One only — this is a charity site, not a growth funnel.

### Verification roadmap — do not build ahead of this

The weakness of v1 is that an uploaded image proves little: screenshots are trivially edited, and
nothing shows the uploader is the donor. The duplicate-receipt constraint stops a receipt being
claimed *twice*; it does not prove it's genuine. Manual review is the only real defence, and it
does not scale. Two better paths exist, in this order of preference:

1. **Platform webhooks** (`platform_webhook`). Many Palestine charities run on LaunchGood,
   Givebutter or Classy. A Fazza campaign page on one of those reports donations by webhook —
   instant, forgery-proof, no receipt at all. Needs charity cooperation, which is why outreach
   matters more than code here.
2. **Forwarded email with DKIM** (`email_dkim`). The user forwards the charity's receipt email;
   a valid DKIM signature from the charity's domain is cryptographic proof the charity really sent
   it, to *their* address. Much stronger than an upload, but forwarding can break signatures and
   each charity's email needs its own parser. Best used as a fast path that falls back to the
   manual queue.

**Build neither until asked.** Also do not build PDF parsing or OCR — guessing at an image is the
weakest possible verification and it would sit on the critical path for every submission.

---

## Admin runbook — rating a charity

A rating is a public claim that one relief charity is more effective than another. Follow this
every time; don't improvise.

1. **Look for an external evaluator first.** Charity Navigator, GiveWell, and the relevant national
   regulator. If one covers the charity, use its rating, record `rating_source` and
   `rating_source_url`, and stop here.
2. **Only if none covers it**, produce a provisional `internal_estimate`. Gather what actually
   exists: published financials, overhead ratio, audit reports, registration status, how plainly
   the organisation reports where money went.
3. **Write the `rating_rationale`** — what evidence was used and what was missing. It is shown
   publicly. If the honest answer is "very little is published", say that; a low-confidence rating
   stated plainly is better than a confident-looking number with nothing behind it.
4. **Set `rating_origin = 'internal_estimate'`** so the UI marks it provisional. Never record an
   estimate as `external`.
5. **Re-check provisional ratings when revisiting a charity.** The fallback only works if someone
   notices once an evaluator finally publishes — otherwise estimates sit stale for years.
6. **On any change, write a `charity_rating_history` row**, then notify donors whose impact score
   moved materially. The email explains what changed and why. It never tells someone they dropped.

Claude may be used to research steps 1–3. Be clear about what that is: a review of what others have
published, not an independent evaluation. It informs the admin's rating; it is not the rating.

---

## Product rules

- **Anonymity is a first-class feature, not an afterthought.** Many donors give privately on
  purpose, and some are at real risk if named. When anonymity and any other goal conflict,
  anonymity wins.
- Only charities with `is_supported = true` accept submissions.
- Leaderboards display USD. Users enter the original currency.
- Copy is warm and encouraging, never shaming. No "you're losing", no guilt, no dark patterns.
  Someone who gave $5 should feel good about appearing here.
- Competition is the mechanism, not the point. The product exists to move money to relief work.

---

## Brand

- **Dark by default.** Near-black ground (`#111111`), green (`#006C35`) as the accent and glow,
  white text. The logo's 3D extrusion is the cue for depth and shadow.
- Colors: green `#006C35` (primary), black `#111111`, white `#FFFFFF`. No other accent colors
  without asking. Define them once in `packages/shared/src/tokens.ts`.
- Latin type: **Unbounded** for headings, the wordmark, and spaced caps. A clean sans for body.
- Arabic: the logo is فزعة in Aref Ruqaa with a 3D extrusion. Use the files in
  `apps/web/public/brand/` (currently just `logo.png` at the repo root, pending scaffold).
  Never re-typeset the logo in code.
- Use logical CSS properties (`ms-`, `me-`, `ps-`, `pe-`, `start`, `end`) rather than left/right,
  so an Arabic RTL version can be added later without a rewrite.

### Motion

Leaderboards are animated, not static tables. When a board updates or the user switches tab, rows
**move** to their new positions rather than snapping, and totals **roll** from the old figure to
the new one. Watching yourself climb is the product's reward; a table that simply re-renders throws
that away.

- Animate position with transforms, keyed on a stable row id, so rows glide instead of repainting.
- Respect `prefers-reduced-motion` — fall back to a plain cross-fade. This is not optional.
- Never animate a page's first paint. Ranks animate on *change*, not on arrival, or every visit
  feels slow.

---

## Working style

- **v1 is:** auth, charities, submit, admin review, global leaderboard, profile, public profile
  pages, and groups. Ask before building anything outside that list.
- Schema changes go through a **new migration file**. Never edit an old migration, and never change
  schema through the Supabase dashboard.
- Keep every decision mobile-compatible — an Expo app is coming.
- Prefer Server Components for rendering; add client components only for interactivity. But put
  *rules* in Postgres, not in the web layer.
- Run `npm run check` after every change.
- Never run a git command that writes. Report what changed and let Adil commit.
- Don't add dependencies without saying why a dependency is needed.
- When something here turns out to be wrong, fix this file in the same change. A stale CLAUDE.md
  is worse than no CLAUDE.md, because the next session will trust it.
