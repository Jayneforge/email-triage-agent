# Email Triage Agent

An AI-powered inbox agent built in n8n that reads, classifies, labels, and acts on incoming email — without the owner opening it. Built as a initial piece improvement after a first automation project of cleaning a 25,000-unread inbox turned into an actual automation project.

Full technical breakdown, architecture diagram, and the debugging log are in [WORKFLOW.md](./WORKFLOW.md).

## What it does

- **Classifies every new email** using an LLM (category, priority, whether it needs action, whether it's a subscription/appointment revealing a future date) and applies the matching Gmail label automatically
- **Archives what's safe to archive** — receipts, newsletters, anything with no action required — and leaves everything else in the inbox
- **Flags genuinely important email via Telegram**, including a deliberate distinction: an email can look spam-like *and* still need a human decision (a phishing-style "confirm your address" email doesn't get silently archived just because it's suspicious)
- **Runs a weekly unsubscribe review** — every Friday, groups the week's promotional senders and sends one Telegram message per sender with Unsubscribe / Keep buttons. Tapping a button does the actual work (hits the unsubscribe link, or sends the unsubscribe request email) and the decision is remembered permanently — that sender is never asked about again
- **Tracks subscriptions and appointments independently of the sender** — the moment an email reveals a renewal or booking date, it's logged and a calendar event is created with two reminders (2 days before, morning of). If the company never sends a reminder, the system still knows the date. If they do send one later, it's matched and confirmed rather than duplicated

## Architecture

Four independently-triggered n8n workflows, coordinating through a shared Google Sheet rather than through direct workflow-to-workflow calls:

1. **Main Triage Agent** — Gmail-triggered, classifies and routes every new email
2. **Friday Digest Sender** — scheduled, aggregates the week's promo senders and sends the Telegram prompts
3. **Telegram Button Handler** — webhook-triggered, processes Unsubscribe/Keep taps
4. **Daily Subscription Reminders** — scheduled, checks tracked dates and fires reminders independently of any inbound email

See [WORKFLOW.md](./WORKFLOW.md) for the full diagram and node-by-node breakdown.

## Stack

- **n8n** (self-hosted) — orchestration
- **Groq (`openai/gpt-oss-120b`)** — email classification via HTTP Request node, OpenAI-compatible chat completions API
- **Gmail** — trigger, labeling, archiving, sending
- **Google Sheets** — shared state across all four workflows (processed log, promo mentions, sender preferences, subscriptions tracker)
- **Google Calendar** — subscription/appointment reminders
- **Telegram** — urgent alerts, weekly digest, interactive button handling

## Why no full auto-unsubscribe

Clicking unsubscribe links programmatically on unverified senders can confirm to spammers that an address is actively read — often making things worse, not better. This system queues candidates for a one-tap human decision instead of acting unilaterally, which also means it's a design choice worth defending in an interview, not just a limitation.

## Repo contents

- `WORKFLOW.md` — architecture diagram, node breakdown, classification schema, bugs found and fixed during build
- `/workflows` — exported n8n workflow JSON for all four flows
