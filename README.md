# Waymo Concierge

**An AI-native, account-grounded support agent for Waymo Pass subscribers.**

🔗 **Live demo:** https://waymo-concierge-234196044632.us-east1.run.app/

> A working prototype built for MGMT 275 (Product Management in Tech Companies), UCLA Anderson — Spring 2026. Demonstrates an AI agent that answers subscription questions using grounded tool calls, with deliberate guardrails around recommendations, refunds, and state changes.

---

## The problem

Waymo Pass is a tiered monthly subscription for autonomous rides (Around Town, Flex Commuter, Daily Commuter). The subscription model removed pricing uncertainty but created a new one: subscribers don't know where they are in their mileage cap, why they were charged an overage, or whether they're on the right tier. Today every one of those questions becomes a customer-support ticket with a median 14-hour resolution time — a trust failure for a brand whose entire positioning is trust.

## What this is

Waymo Concierge is an in-app chat agent that answers Waymo Pass account questions in seconds, grounded in the subscriber's actual account data rather than reasoned from training. It uses an LLM for language and reasoning but is tightly scoped: it does not give financial advice, does not recommend tiers, does not approve refunds, and does not act outside subscription support.

This repository contains a functional frontend prototype with a single hardcoded user, demonstrating the full agentic loop and the product's core design decisions.

## What it does

- Answers mileage and billing questions grounded in the subscriber's account ("How many miles do I have left?", "Why was I charged $42?")
- Surfaces tier-fit math — what each tier would have cost given the user's actual 90-day usage — **without recommending a tier**
- Walks subscribers through tier changes, pauses, and cancellations using a preview-then-confirm pattern
- Files refund *requests* for human review (never approves refunds itself)
- Escalates safety, accessibility, legal, and high-emotion conversations to a human
- Shows the tools the agent called for each response, so the agentic logic is visible

## Core design decisions

These are the decisions that define the product. They are deliberate and were the most-debated parts of the spec.

| Decision | Rationale |
|---|---|
| **Show the math, never recommend a tier** | A recommendation that fits past behavior ages badly when behavior changes. The agent surfaces usage and per-tier cost; the user decides. |
| **Refund = request, never approval** | Refunds touch retention strategy, fraud signals, and policy edge cases — not LLM territory. The agent files a request; a human approves. |
| **Preview → confirm for all state changes** | No tier change, pause, or cancellation happens without an explicit user confirmation on a preview screen. Enforced at the tool layer, not by the model's promise. |
| **Explicit human-handoff triggers** | Safety incidents, accessibility needs, legal threats, and repeated frustration route to a human rather than getting a "better" automated answer. |

## How it works

The agentic loop runs client-side:

1. User sends a message
2. The agent (LLM) decides which tools to call
3. Tools execute against the mocked backend and return the subscriber's data
4. The agent grounds its answer in the tool results
5. State-changing actions pause for an explicit user confirmation before committing

**Read-only tools:** `getSubscriberProfile`, `getRecentTransactions`, `getUsageHistory`, `getTierOptions`, `computeWhatIf`, `getPolicy`

**State-changing tools (preview/commit):** `previewTierChange` / `commitTierChange`, `previewPause` / `commitPause`, `previewCancel` / `commitCancel`, `fileRefundRequest`, `escalateToHuman`

## Sample data

The prototype simulates the experience for one hardcoded subscriber:

- **Joe Bruin** — Los Angeles, Flex Commuter tier ($149/mo, 200-mile cap)
- 142 of 200 miles used in the current cycle (May 1 – June 1, 2026)
- 90-day usage: 312 miles across 47 rides
- Recent transactions include a $42.10 overage and a $5.00 referral credit

All backend data is hardcoded — there is no database, no authentication, and no real API except the LLM.

## Tech stack

- React + TypeScript + Tailwind CSS
- Google Gemini API (agent reasoning and function calling)
- Fully client-side; deployed on Google Cloud Run
- Built using Google AI Studio's Build feature

## Repository contents

| File | What it is |
|---|---|
| `README.md` | This file |
| `Waymo_Concierge_Spec.md` | The Source of Truth spec — system prompt, tool schemas, refusal patterns, worked examples |
| `Waymo_Concierge_PR_FAQ.docx` | Narrative & PR-FAQ — the product justification |


## Running it

The primary way to use the prototype is the **live demo** linked at the top. If you exported the source from Google AI Studio and want to run it locally, follow the instructions in the AI Studio export (typically `npm install` then `npm run dev`).

## Project context

Built as the final project for MGMT 275, demonstrating an end-to-end AI product cycle: problem identification → product strategy → spec → functional prototype → evaluation. The accompanying spec, PR-FAQ, and eval set document the reasoning behind every design decision.

## Authors

Brian Palmigiano · Eylon Siman-Tov
UCLA Anderson — MGMT 275, Spring 2026

---

*This is an academic prototype. It is not affiliated with or endorsed by Waymo LLC. "Waymo" is used here as the subject of a product management case study.*
