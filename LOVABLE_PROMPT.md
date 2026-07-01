# The Single Lovable Prompt

Copy everything inside the box below and paste it into Lovable as your first message. It builds a working v1. After it generates, connect Supabase and add the secrets listed at the bottom, then use the follow‑up prompts in [`PLAN.md`](./PLAN.md#8-roadmap-after-the-single-prompt).

> Tip: Lovable builds iteratively. This prompt is intentionally scoped so the first generation is stable; the real LinkedIn OAuth/publishing and cron jobs are best wired in as the immediate next steps (they're described here so Lovable stubs them correctly).

---

```
Build a full-stack web app called "TrendPilot" — an AI content engine that discovers trending tech topics, uses a team of parallel AI agents to produce verified, US-audience-ready LinkedIn posts and carousels, and schedules/publishes them to my LinkedIn profile to grow followers and attract high-value clients.

TECH STACK
- React + Vite + TypeScript, Tailwind CSS, shadcn/ui components, lucide-react icons. Clean, modern, professional SaaS dashboard UI with a left sidebar, dark mode support, and a polished, trustworthy look (this is used to win enterprise clients).
- Supabase for Auth (email/password), Postgres database, Storage (for generated images/slides), and Edge Functions for all AI agents and third-party integrations. Use Supabase Row Level Security so each user only sees their own data.

CONTEXT (my business)
- I sell services in 5 pillars: Web Development, AI Integration, AI Agents Development, Mobile App Development, and Stripe/Payments Integration.
- Target audience: USA-based founders, CTOs, and decision-makers ("big fish" clients).
- Goal: grow LinkedIn followers and book high-value clients within 30 days via consistent, high-quality, verified content.

PAGES / FEATURES (build all of these with working navigation)
1. Dashboard: KPI cards (followers, weekly follower delta, impressions, posts published, consistency streak, leads in pipeline), a mini calendar of upcoming posts, and a "Today's suggested actions" list.
2. Trends: a board of trending items grouped by pillar. Each card shows title, source, a relevance score, pillar tag, and buttons "Generate content" and "Save". A "Refresh trends" button. Filters by pillar and source.
3. Content Studio: shows the AI generation pipeline. When I click "Generate content" on a trend, run the multi-agent pipeline and show a live status list of each agent (pending → running → done/failed). Output a draft post with hook, body, CTA, hashtags, plus a verifier report panel (claims checked, flags, US-audience fit, readability).
4. Carousel Editor: a slide-by-slide editor (6–8 slides) with title, body text per slide, and a generated/branded image per slide. Preview as a swipeable carousel and export slides as PNG/PDF (render slides client-side to images).
5. Approvals Queue: list of drafts with status needs_review. Inline edit, view verifier flags, then Approve / Reject. Approving opens a schedule dialog with an AI-suggested best time slot.
6. Calendar & Schedule: month/week calendar of scheduled + published posts. A "Schedule rules" panel to set cadence (posts per day cap), active days, and preferred US time slots (default: Tue–Thu, 7:30–9:30 AM ET and 12:00–1:00 PM ET; default 1 post/day + 2–3 carousels/week).
7. Analytics: charts for followers over time, impressions, engagement by post, and best-performing pillar/format. Include a quick "Log today's metrics" form and a CSV import for LinkedIn's native analytics export (since personal-profile analytics via API are limited).
8. Leads CRM: a Kanban board (New → Conversation → Call → Proposal → Won → Lost) with cards (name, company, source, notes) to track inbound clients.
9. Settings: brand voice, tech stack, ideal client profile, timezone, AI model provider selection, and an "Integrations" section with a "Connect LinkedIn" button and a "Connect AI provider" area.

THE AI AGENT TEAM (implement as one Supabase Edge Function `run-content-pipeline` that calls an LLM API; run independent agents in parallel with Promise.all; log each agent's status/latency to an `agent_runs` table so the UI can show live progress)
- Trend Scout: fetch and rank fresh items across my 5 pillars from public sources (Hacker News Algolia API, Dev.to API, GitHub Trending, Reddit, Product Hunt, RSS). Dedupe and score by recency + engagement + relevance to my stack.
- Angle Strategist: turn a trend into 2–3 post angles aimed at USA founders/CTOs (pain → insight → proof → soft CTA).
- Writer: draft the LinkedIn post — strong 2-line hook, scannable body with whitespace, one clear CTA, 3–5 hashtags, US English tone. (runs in parallel with Carousel Designer)
- Carousel Designer: produce slide-by-slide copy + layout for a swipeable document post. (parallel with Writer)
- Visual Agent: generate a cover image and per-slide visuals (call an image API if a key is present, otherwise render branded HTML slide templates to PNG).
- Verifier / Fact-Checker: check claims, remove unverifiable stats, flag ToS-risky wording, confirm US-audience fit and readability; return pass/fail + suggested fixes stored in a verifier_report JSON.
- Growth Editor: final polish for the LinkedIn algorithm (hook first, whitespace, a comment-bait question, a searchable keyword) and set recommended post type + best time slot.
The pipeline's final output is a Draft (post text + assets + metadata + verifier_report) saved with status "needs_review" so it appears in the Approvals Queue.

DATABASE TABLES (with RLS by user_id)
profiles, pillars, trend_sources, trends, drafts, carousels, slides, posts, schedule_rules, metrics, leads, agent_runs, integrations (stores LinkedIn OAuth tokens + selected AI provider). Choose sensible columns matching the features above.

LINKEDIN PUBLISHING (compliant — do NOT use browser automation/scraping; that violates LinkedIn's terms)
- Add a "Connect LinkedIn" OAuth 2.0 flow using scopes: openid, profile, email, w_member_social. Store tokens in the `integrations` table.
- Add an Edge Function `publish-post` that publishes an approved post (text + optional images/document carousel) to my LinkedIn profile using the official LinkedIn Posts API with the stored access token.
- Add an Edge Function `publish-due-posts` intended to be run on a schedule (pg_cron) that finds posts whose scheduled_at is due and publishes them, then records the returned post URN/URL and sets status "published".
- Keep a human-in-the-loop by default: nothing publishes without me Approving it. Include a Settings toggle "Auto-publish approved queue" (default OFF).
- If LinkedIn isn't connected yet, still let me create/approve/schedule posts and clearly show "LinkedIn not connected" states. Stub the API calls cleanly so they're easy to complete once credentials are added.

SECRETS (reference these; I will add them in Supabase)
- LLM: use OPENAI_API_KEY (allow  GEMINI_API_KEY or OPEN_ROUTER as alternatives selectable in Settings).
- Optional image gen: GEMINI_API_KEY or OPENAI_API_KEY.
- LinkedIn: Use agentic browser to open Linkedin and Loggedin and It should autometically open visit, check visually which is good and post accordingly

SEED DATA
Seed the 5 pillars and a handful of realistic sample trends and sample drafts per pillar so the dashboard, trends board, and approvals queue look populated on first load.

DELIVERABLE FOR THIS FIRST BUILD
A working app with all pages, navigation, database + RLS, seeded sample data, the `run-content-pipeline` edge function wired to an LLM key with the 7 agents (parallel where possible) and live per-agent status, the carousel editor with client-side PNG/PDF export, the approvals + scheduling flow, analytics with manual/CSV logging, the leads CRM, and stubbed-but-structured LinkedIn OAuth + publish functions ready to complete. Prioritize a clean, professional, responsive UI.
```

---

## After it generates — do these 3 things
1. **Connect Supabase** (Lovable will prompt you) and add the secrets above.
2. **Create your LinkedIn app** (see [`PLAN.md` §7](./PLAN.md#7-integrations--secrets-needed-add-in-lovablesupabase-secrets)) and paste the redirect URL Lovable gives you.
3. Run the **follow‑up prompts** in [`PLAN.md` §8](./PLAN.md#8-roadmap-after-the-single-prompt) to finish live agent streaming, real LinkedIn posting, and cron scheduling.
