<div align="center">

<img src="./assets/banner.svg" alt="RWA Complaint and Vendor Coordination Agent" width="100%" />

<br/>

![n8n](https://img.shields.io/badge/n8n-self--hosted-EA4B71?style=for-the-badge&logo=n8n&logoColor=white)
![Groq](https://img.shields.io/badge/Groq-llama--3.3--70b-F55036?style=for-the-badge)
![Telegram](https://img.shields.io/badge/Telegram-bot-26A5E4?style=for-the-badge&logo=telegram&logoColor=white)
![Google Sheets](https://img.shields.io/badge/Google%20Sheets-ledger-34A853?style=for-the-badge&logo=googlesheets&logoColor=white)
![Status](https://img.shields.io/badge/status-3%20of%204%20workflows%20built-5CF2B0?style=for-the-badge&labelColor=0b1511)

**An AI agent that runs the complaint desk of an Indian apartment society, so the secretary doesn't have to.**

[How it works](#how-it-works) · [The four workflows](#the-four-workflows) · [Status](#honest-status) · [Run it](#run-it-yourself) · [Engineering notes](#engineering-notes)

</div>

---

## The problem

Every residential society (RWA) in India runs on the same broken loop. A resident complains in a 300-person WhatsApp group. The message scrolls away. The secretary phones a plumber from memory. Nobody knows if it got fixed until the same resident complains again, louder.

There is no ticket, no record, no owner, and no closure.

## The fix

Residents message one bot. The agent does the rest, and the secretary stays in control of anything that leaves the building.

<div align="center">
<img src="./assets/telegram-flow.svg" alt="Resident, secretary and ledger views" width="100%" />
</div>

| Before | After |
|---|---|
| Complaint buried in a group chat | Logged as a ticket with ID, flat, category, urgency |
| Secretary hunts for the right vendor's number | Vendor message drafted from a per-category list |
| Vendor contacted with no oversight | Secretary approves in Telegram before anything goes out |
| Resident never hears back | Resident is notified automatically when it's resolved |
| No history | Every ticket lives in a Google Sheet |

## How it works

<div align="center">
<img src="./assets/architecture.svg" alt="System architecture" width="100%" />
</div>

One n8n canvas, separate chains per workflow, one Google Sheet as the system of record.

## The four workflows

Built one at a time, each tested end to end before starting the next.

### 1. Complaint Intake  `built`
Resident messages the bot. Groq classifies the complaint into a category and urgency. The agent resolves the resident from a Residents tab, creates a ticket, appends it to the Sheet, and confirms back to the resident.

### 2. Vendor Dispatch  `built`
A new row in the Complaints sheet triggers the chain. Groq drafts a vendor message using the vendor list for that category. The secretary gets it on Telegram to approve. On approval they receive a `wa.me` link that opens WhatsApp with the message ready to send.

### 3. Status Tracking  `built`
The secretary sends `/resolve <ticket_id>`. The Sheet updates and the resident gets an automatic resolution message.

### 4. Notice Broadcast  `planned`
Secretary drafts a notice through the bot, the agent formats it, broadcasts it to residents, and tracks acknowledgments in the Sheet.

## Design principles

- **Human in the loop.** Nothing is sent to a vendor without the secretary's approval.
- **One source of truth.** A single Google Sheet with `Complaints` and `Residents` tabs. Anyone on the committee can open it without a login to some new tool.
- **Free-tier friendly.** Groq, Sheets, Telegram and self-hosted n8n. No per-seat SaaS bill for a society that is deciding whether to pay for a parking sticker.
- **Portable.** Config lives in one node. Swap the Sheet, the vendor list and the bot token, and it runs for a different society.

## Honest status

This is a working build, not a deployed product. Being clear about that:

| Piece | State |
|---|---|
| Workflows 1, 2, 3 | Built and tested end to end |
| Workflow 4 (notice broadcast) | Not started |
| Vendor messaging | Manual send via `wa.me` link. No WhatsApp Business API yet |
| Real-society pilot | None yet. Looking for an RWA secretary to trial it |
| Usage numbers | None to report, so none are claimed |

## Stack

| Layer | Tool |
|---|---|
| Orchestration | n8n (self-hosted, Docker) |
| LLM | Groq, `llama-3.3-70b-versatile` |
| Messaging | Telegram Bot API |
| Data | Google Sheets |
| Local webhook exposure | ngrok (static domain) |

## Run it yourself

1. Run n8n locally with Docker and expose it with an ngrok domain so Telegram can reach your webhook.
2. Create a Telegram bot with BotFather and add the token as an n8n credential.
3. Create a Google Sheet with two tabs: `Complaints` (ticket_id, flat, category, urgency, status, timestamp, chat_id, description) and `Residents`.
4. Add your Groq API key as an n8n credential.
5. Edit the **Config** node in each workflow: sheet ID, secretary chat ID, vendor list.
6. Activate the workflows and message your bot.

> Workflow exports are not published in this repo. Reach out if you want to see it running.

## Engineering notes

Real problems hit while building this, kept because they cost hours.

- **Use `$json['field']`, not `.item` selectors.** `.item` breaks on multi-item executions.
- **An AI Agent node drops every upstream field.** Add a Merge node (Combine by Position) after each one to reattach context.
- **Groq wraps JSON in markdown fences.** Strip them before `JSON.parse()`:
  ```js
  JSON.parse(text.replace(/```json|```/g, '').trim())
  ```
- **Telegram webhooks need a stable public URL.** A static ngrok domain beats re-registering the webhook every restart.

## Roadmap

- [x] Complaint intake
- [x] Vendor dispatch with secretary approval
- [x] Resolution tracking
- [ ] Notice broadcast with acknowledgment tracking
- [ ] WhatsApp Business API for vendor messaging
- [ ] Pilot with a real society

---

<div align="center">

Built by **Samhita** · [Edit Theory](https://github.com/Samhita1008) · AI automation for real businesses

</div>
