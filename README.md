# Autonomous Email Triage & Subscription Intelligence Agent

> An enterprise-grade, event-driven n8n automation architecture designed to conquer unmanaged inbox chaos, secure calendar tracking independently of company reminder emails, and manage human-in-the-loop weekly unsubscribe prompts via Telegram.

---

## What this covers

* **Main Triage Agent (v2):** Polls unread mail, normalizes records, marks messages as read instantly to prevent processing loops, and utilizes Groq (Qwen 3) for multi-label categorization and priority routing.
* **Friday Digest Sender:** Runs every Friday evening to aggregate promotional senders, cross-reference permanent user preferences, and push a batch prompt with interactive action buttons to Telegram.
* **Telegram Button Handler:** Listens for real-time callback queries, executes unsubscription protocols, updates database states, and saves permanent user preferences.
* **Daily Subscription Reminders:** Fires daily at 6:00 AM to check active subscription and appointment tracker rows independently of company emails, sending timely alerts and Monday weekly schedules.

---

## System Architecture & Workflows

Instead of a monolithic script, this architecture uses **four loosely-coupled, independently-triggered n8n workflows** coordinating across a structured multi-tab Google Sheets database.