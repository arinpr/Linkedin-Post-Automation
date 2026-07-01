# TrendPilot — Plan to Build with Lovable (Single Prompt)

**Product name (suggested):** TrendPilot
**One-liner:** An AI content engine that discovers trending tech topics, runs a team of parallel AI agents to produce verified, US‑audience‑ready LinkedIn posts + carousels, and schedules/publishes them to your LinkedIn to grow followers and attract high‑value clients.

This document is the **plan**. The ready‑to‑paste build prompt is in [`LOVABLE_PROMPT.md`](./LOVABLE_PROMPT.md).

---

## 1. Goal & success metrics

**Business goal:** Grow your LinkedIn following and land a few "big fish" clients within 30 days, positioned around your tech stack (Web Dev, AI Integration, AI Agents, Mobile App Dev, Stripe Integration).

**Product metrics the app tracks:**
- Followers gained (weekly delta)
- Post impressions, reactions, comments, shares
- Profile views & "who viewed" trend
- Inbound leads / DMs / booked calls (manually logged in a simple CRM board)
- Content published vs. scheduled (consistency streak)

**Target cadence (default, USA audience):** 1 text/insight post per day + 2–3 carousels per week, published in peak US windows (Tue–Thu, 7:30–9:30 AM and 12:00–1:00 PM ET). The app treats these as editable defaults and adapts using your own performance data.

---

## 2. Important reality check (read before building)

Lovable builds **full‑stack web apps** (React + Tailwind + shadcn/ui frontend, Supabase backend: Postgres, Auth, Storage, Edge Functions, Cron). It is excellent for the dashboard, the agent orchestration (via edge functions calling an LLM API), content/carousel generation, scheduling, and publishing through **official APIs**.

The part of your request that needs a compliant approach:

> "automatically visit to my LinkedIn profile and visually check, post content, scheduled content…"

- **Do NOT** drive a headless browser that logs into LinkedIn and clicks around. Automated browsing/scraping/robotic posting violates the [LinkedIn User Agreement §8.2](https://www.linkedin.com/legal/user-agreement) and is the #1 cause of account restrictions/bans. That risks the exact asset (your profile) you're trying to grow.
- **Compliant path (what this plan uses):** LinkedIn's official **OAuth 2.0 + Posting API** with the `w_member_social` scope (from the "Share on LinkedIn" product) and `Sign In with LinkedIn using OpenID Connect`. This lets the app publish text + image/document (carousel) posts to *your* profile with your explicit authorization.
- **Human‑in‑the‑loop by default:** Because the goal is high‑value clients, every post gets an **Approve / Edit / Reject** step before it goes live (you can toggle "auto‑publish approved queue" once you trust it).
- **Analytics limits:** LinkedIn's API exposes rich analytics for *Company Pages*, but **personal profile** follower/impression analytics are limited. So the app supports (a) API metrics where available, and (b) a fast manual "log yesterday's numbers" entry, plus optional CSV import of LinkedIn's native analytics export.

The prompt is written so Lovable builds the compliant version, with browser‑automation explicitly out of scope.

---

## 3. Architecture

```
┌────────────────────────────────────────────────────────────────┐
│  React + Tailwind + shadcn/ui (Lovable frontend)                 │
│  Dashboard · Trends · Content Studio · Carousel Editor · Queue   │
│  Calendar · Approvals · Analytics · Leads CRM · Settings         │
└───────────────┬──────────────────────────────────────────────────┘
                │ Supabase JS (auth + data)
┌───────────────▼──────────────────────────────────────────────────┐
│  Supabase                                                          │
│  • Postgres (topics, trends, drafts, posts, schedule, leads)       │
│  • Auth (email/password)                                           │
│  • Storage (generated images / carousel slides / PDFs)             │
│  • Edge Functions (the agents + integrations)                      │
│  • Cron (pg_cron): trend refresh + publish-due-posts               │
└───┬───────────────┬───────────────┬───────────────┬───────────────┘
    │               │               │               │
 Trend sources   LLM API        Image/Carousel   LinkedIn API
 (HN, Dev.to,    (OpenAI/       (client render   (OAuth + Posts /
 Reddit, RSS,    Anthropic/     of slides →      w_member_social)
 GitHub, PH)     Gemini)        PNG/PDF)
```

**Why this shape:** each "agent" is a specialized LLM call (its own system prompt + tools) orchestrated by an edge function. Running them concurrently in one function (Promise.all) gives the "agents working in parallel" behavior without extra infra.

---

## 4. The agent team (parallel where possible)

Orchestrated by an edge function `run-content-pipeline`. For a chosen trend, agents run in this dependency order; independent steps run in parallel.

| # | Agent | Job | Runs |
|---|-------|-----|------|
| 1 | **Trend Scout** | Pull + rank fresh items from HN, Dev.to, Reddit, GitHub Trending, Product Hunt, RSS across your 5 pillars; dedupe; score by recency + engagement + relevance to your stack. | Scheduled (cron) + on‑demand |
| 2 | **Angle Strategist** | Turn a trend into 2–3 post angles tailored to attract **decision‑maker / founder** buyers in the USA (pain → insight → proof → soft CTA). | After a trend is picked |
| 3 | **Writer** | Draft the LinkedIn post: strong hook, scannable body, 1 clear CTA, 3–5 hashtags, US English/tone. | Parallel with Designer |
| 4 | **Carousel Designer** | Produce slide‑by‑slide copy + layout spec (title, 6–8 slides, CTA slide) for a swipeable document post. | Parallel with Writer |
| 5 | **Image/Visual Agent** | Generate cover image + per‑slide visuals (prompt an image API, or render branded HTML slides to PNG). | After Designer |
| 6 | **Verifier / Fact‑Checker** | Check claims, remove hallucinated stats, flag anything unverifiable, ensure no ToS‑risky language, confirm US‑audience fit and readability. Returns pass/fail + fixes. | After Writer/Designer |
| 7 | **Growth Editor** | Final polish for LinkedIn algorithm (hook in first 2 lines, whitespace, comment‑bait question, keyword for search), sets recommended post type + best time slot. | Last |

Output = a **Draft** record (text + assets + metadata + verifier report) that lands in the **Approvals** queue.

---

## 5. Data model (Postgres tables)

- `profiles` — user, brand voice, tech stack, target ICP (ideal client profile), timezone.
- `pillars` — the 5 topics (Web Dev, AI Integration, AI Agents, Mobile App, Stripe) + custom.
- `trend_sources` — enabled sources + config.
- `trends` — fetched items (title, url, source, score, pillar, fetched_at, raw_engagement).
- `drafts` — generated content (post_text, hashtags, angle, verifier_report jsonb, status: `generating|needs_review|approved|rejected`).
- `carousels` / `slides` — slide copy + image URLs (Storage).
- `posts` — approved content queued/published (scheduled_at, published_at, linkedin_post_urn, status).
- `schedule_rules` — cadence, days, time slots, per‑day cap.
- `metrics` — daily followers, impressions, reactions, comments, profile_views (API + manual).
- `leads` — simple CRM (name, company, source, stage: `new|conversation|call|proposal|won|lost`, notes).
- `agent_runs` — audit log of each pipeline run (per‑agent status, tokens, latency, errors).
- `integrations` — LinkedIn OAuth tokens (access/refresh, expiry), API provider selection.

---

## 6. Key user flows

1. **Onboarding:** enter brand voice, tech stack, target client profile, timezone → connect LinkedIn (OAuth) → pick pillars.
2. **Discover trends:** Trend Scout populates the Trends board (auto daily + manual refresh). You star the ones you like.
3. **Generate:** click "Generate content" on a trend → pipeline runs the agents (live status per agent) → draft + carousel + verifier report appear.
4. **Review/Approve:** edit inline, see the verifier's flags, Approve → schedule (auto‑suggested best slot) or Reject.
5. **Schedule & publish:** cron `publish-due-posts` posts approved items to LinkedIn at the scheduled time via the official API; status → published + link.
6. **Measure & learn:** Analytics dashboard + manual/API metric logging; the app recommends what topic/format is winning and adjusts suggested cadence.
7. **Convert:** Leads board to track inbound interest → calls → clients.

---

## 7. Integrations & secrets needed (add in Lovable/Supabase secrets)

| Purpose | Provider (pick one where noted) | Secret |
|---|---|---|
| LLM (agents) | OpenAI **or** Anthropic **or** Gemini | `OPENAI_API_KEY` / `ANTHROPIC_API_KEY` / `GEMINI_API_KEY` |
| Image generation (optional) | OpenAI Images / Replicate | `OPENAI_API_KEY` / `REPLICATE_API_TOKEN` |
| LinkedIn publishing | LinkedIn Developer App | `LINKEDIN_CLIENT_ID`, `LINKEDIN_CLIENT_SECRET`, redirect URL |
| Trend sources | HN/Dev.to/GitHub/Reddit/Product Hunt/RSS | mostly public; Reddit/PH may need app tokens |

**LinkedIn app setup (one‑time, you do this):**
1. Create an app at the LinkedIn Developer Portal.
2. Add products: **Sign In with LinkedIn using OpenID Connect** + **Share on LinkedIn**.
3. Request scopes: `openid`, `profile`, `email`, `w_member_social`.
4. Set the OAuth redirect URL to your app's callback (Lovable will give you the URL).

---

## 8. Roadmap after the single prompt

The single prompt gets you a working v1. Recommended follow‑up prompts in Lovable:
1. "Add live per‑agent progress streaming in the pipeline view."
2. "Add the LinkedIn OAuth flow + real posting edge function using `w_member_social`."
3. "Add pg_cron jobs for daily trend refresh and publish‑due‑posts."
4. "Add CSV import for LinkedIn analytics export and a weekly performance summary."
5. "Add the Leads CRM Kanban board."

---

## 9. Risks & mitigations

- **Account safety:** official API + human approval only; no browser bots. → protects your profile.
- **AI hallucination:** dedicated Verifier agent + your final approval. → protects credibility with big clients.
- **API limits/costs:** cache trends; batch LLM calls; let user pick model. 
- **Personal‑profile analytics gaps:** manual/CSV logging fallback.
- **"30‑day big client" expectation:** the app maximizes consistency, targeting, and quality — the strongest controllable levers — but outcomes also depend on offer and outreach, so a Leads board + CTA strategy is built in.

---

## 10. 30‑day growth playbook (baked into app defaults)

- **Week 1 — Authority:** daily insight posts on your 5 pillars; 1 carousel ("how we built X with Stripe/AI agents"). Optimize profile headline/CTA.
- **Week 2 — Proof:** case‑study carousels, before/after, mini‑tutorials; add 1 clear "DM me / book a call" CTA per 3 posts.
- **Week 3 — Demand:** "problem → solution" posts targeting founder pain (payments, AI adoption, shipping speed); comment‑bait questions.
- **Week 4 — Convert:** offer‑driven posts + testimonials/results; direct CTA; funnel warm engagers into the Leads board.
