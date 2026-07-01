# Linkedin-Post-Automation — TrendPilot

Plan and a single copy‑paste prompt to build **TrendPilot** with [Lovable](https://lovable.dev): an AI content engine that discovers trending tech topics, runs a team of parallel AI agents to produce verified, US‑audience‑ready LinkedIn posts + carousels, and schedules/publishes them to your LinkedIn to grow followers and win high‑value clients.

## Contents
- [`PLAN.md`](./PLAN.md) — full plan: goal, architecture, the AI agent team, data model, integrations, compliance, and a 30‑day growth playbook.
- [`LOVABLE_PROMPT.md`](./LOVABLE_PROMPT.md) — the single prompt to paste into Lovable, plus setup steps.

## Quick start
1. Read `PLAN.md` (note the LinkedIn compliance section — the app uses LinkedIn's official API, **not** browser automation).
2. Copy the prompt from `LOVABLE_PROMPT.md` into Lovable.
3. Connect Supabase, add your secrets (LLM + LinkedIn), create a LinkedIn developer app, and run the follow‑up prompts.
