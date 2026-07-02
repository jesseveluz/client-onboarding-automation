# Client Onboarding Automation — Make.com Workflow

**Category:** Business Process Automation
**Tools:** Make.com, Google Forms, Google Sheets, DocuSign, Gmail, Slack (Incoming Webhooks)
**Type:** End-to-end no-code automation pipeline

This repo contains the exported blueprint(s) for a fully automated client onboarding system, along with a full write-up of the problem, design, and results below. Sensitive values (webhook URLs, sheet IDs, account-specific identifiers) have been redacted from the JSON files — see [Notes on the Blueprint Files](#notes-on-the-blueprint-files) at the bottom.

---

## Overview

Designed and built a fully automated client onboarding system that takes a new client from initial inquiry to signed contract and welcome email — with zero manual data entry. The system eliminates repetitive admin work, keeps a live, accurate record of every client's status, and ensures nothing falls through the cracks between "form submitted" and "contract signed."

---

## The Problem

Manual client onboarding is slow and error-prone. A typical process involves someone reading a form response, copying details into a spreadsheet by hand, manually creating and sending a contract, remembering to follow up once it's signed, and separately notifying the team — all steps that are easy to delay, forget, or fumble under a busy schedule. For a growing service business, this bottleneck directly limits how many new clients can be onboarded smoothly per month, and creates an inconsistent first impression at the exact moment it matters most.

**Goals for the automation:**
- Capture new client information the moment it's submitted, with no manual re-entry
- Automatically generate and send a contract for e-signature
- Track every client's status (pending vs. signed) in a live, shared spreadsheet
- Notify the team instantly when a new client comes in and when a contract is completed
- Send a professional welcome email to the client automatically, only once they've signed

---

## The Solution

Built as two interlocking Make.com scenarios, structured around the fact that contract signing is an asynchronous event that can happen anywhere from minutes to days after initial contact.

### Scenario 1: New Client Intake

| Step | Module | Function |
|---|---|---|
| 1 | Google Forms – Watch Responses | Triggers the instant a new client submits the intake form |
| 2 | Google Sheets – Add a Row | Logs client details (Name, Email, Company) into a live tracking sheet with Status set to "Pending" |
| 3 | DocuSign – Create and Send an Envelope from Template | Dynamically generates a contract from a pre-built template, auto-populating the client's name and role, and sends it for signature |
| 4 | Google Sheets – Update a Row | Writes the returned DocuSign Envelope ID back into the same row, creating a reliable unique key for tracking |
| 5 | Gmail – Send an Email | Notifies the company that a new client has come in and the contract is awaiting signature |

### Scenario 2: Contract Signed → Client Welcomed

| Step | Module | Function |
|---|---|---|
| 1 | DocuSign – Watch Envelope Recipient Status Changes | Instant webhook trigger fires the moment the client completes signing |
| 2 | Filter | Ensures the flow only proceeds when envelope status = Completed |
| 3 | Google Sheets – Search Rows | Locates the exact client record using the stored Envelope ID |
| 4 | Google Sheets – Update a Row | Updates Status from "Pending" to "Signed" |
| 5 | Gmail – Send an Email | Sends a personalized welcome email to the client, confirming next steps |
| 6 | HTTP (Slack Webhook) | Notifies the team in Slack that the contract is signed and the client is fully onboarded |

### Key Design Details

- **Template-driven contract generation:** Built a reusable DocuSign template with role-based recipients and auto-filling data-label fields, so the same template dynamically serves every new client without manual editing.
- **Auto-filled signing date:** Configured DocuSign's Date Signed tab to auto-capture the exact date of signature, removing manual date entry entirely.
- **Reliable record matching:** Used the DocuSign Envelope ID (rather than email alone) as the unique identifier linking a signed contract back to its spreadsheet row — avoiding mismatches in edge cases like repeat clients or resubmissions.
- **Slack integration without native app approval:** When the workspace blocked third-party app installation, solved this using a Slack Incoming Webhook combined with Make's HTTP module — achieving the same real-time notification without requiring elevated admin permissions.

---

## Tools & Technologies

- **Make.com** — automation orchestration across all steps
- **Google Forms** — client intake data collection
- **Google Sheets** — live client tracking database
- **DocuSign** — contract generation and e-signature
- **Gmail** — automated client-facing communication
- **Slack (Incoming Webhooks + HTTP module)** — internal team notifications

---

## Challenges & Solutions

**Challenge:** The two scenarios needed to stay decoupled, since contract signing happens asynchronously relative to form submission.
**Solution:** Designed as two independently triggered scenarios connected by a shared spreadsheet record (matched on Envelope ID) rather than a single linear flow — allowing each to run reliably on its own timeline.

**Challenge:** Slack blocked Make from being installed as an approved app.
**Solution:** Bypassed the restriction with a single-purpose Incoming Webhook, avoiding the need for broader admin approval while keeping full notification functionality.

---

## Results / Impact

- **Zero manual data entry** from client submission through contract completion
- **Real-time visibility** into every client's onboarding status via a live spreadsheet and Slack alerts
- **Consistent, professional client experience** — every client automatically receives a contract and welcome email without delay or human error
- **Fully reusable system** — onboarding a new client now requires no additional setup; the same automation scales to each new submission indefinitely

---

## Notes on the Blueprint Files

The `.json` files in this repo are exported Make.com blueprints for both scenarios. A few things worth knowing before reusing them:

- **Redacted fields:** Sensitive values — Slack webhook URLs, Google Sheet IDs, account-specific identifiers — have been replaced with placeholders (e.g., `"YOUR_WEBHOOK_URL_HERE"`). Replace these with your own values before importing.
- **No credentials included:** Make blueprints never store login credentials, API keys, or OAuth tokens. After importing, you'll need to reconnect Google, DocuSign, Gmail, and Slack to your own accounts.
- **How to import:** In Make, go to Scenarios → Create a new scenario → open the (•••) menu → Import Blueprint → select the `.json` file → reconnect your apps → test before activating.

---

*This project demonstrates end-to-end automation design: mapping a real business process, selecting the right tools for each step, handling asynchronous events correctly, and solving integration roadblocks (custom field mapping, platform permission restrictions) as they arose.*
