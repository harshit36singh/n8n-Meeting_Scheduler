# Setup

## 1. Import the workflow

In n8n: **Workflows → Import from File** and select `workflows/meeting-scheduler.json`. All nodes will show up needing credentials (see below) before you can activate it.

## 2. Google Calendar

1. In [Google Cloud Console](https://console.cloud.google.com/), create (or reuse) a project and enable the **Google Calendar API**.
2. Create an OAuth 2.0 Client ID (type: Web application) and add n8n's OAuth redirect URL (shown in the credential screen below) to **Authorized redirect URIs**.
3. In n8n, create a **Google Calendar OAuth2 API** credential using that client ID/secret, and connect your account.
4. Assign the credential to the **Get Busy Times** and **Book Meeting** nodes.
5. Both nodes default to the `primary` calendar. Change the `calendar` field on each if you want to check/book against a different calendar.

## 3. Gmail

1. Enable the **Gmail API** in the same (or another) Google Cloud project.
2. Create a **Gmail OAuth2** credential in n8n and connect the account you want sending mail.
3. Assign it to all five Gmail nodes: **Request Missing Info**, **Send No-Availability Email**, **Send Confirmation Email**, **Send 24h Reminder**, **Send 30min Reminder**, **Send Follow-Up Email**.
4. Prefer SMTP instead? Swap the Gmail nodes for n8n's **Send Email** node — the `sendTo`/`subject`/`message` fields map directly.

## 4. Inbound email (IMAP)

The **Meeting Request Email** trigger polls a mailbox with the IMAP node.

1. If using Gmail as the inbox, enable IMAP access in Gmail settings and create an app password (or use OAuth if your provider supports it).
2. Create an **IMAP** credential in n8n with the host/port/user/password for that mailbox.
3. Assign it to the **Meeting Request Email** node. Adjust the `customEmailConfig` search filter if you want to scope which messages count as meeting requests (e.g. by subject).

## 5. AI email parsing (optional but recommended)

The **Parse Email With AI** node turns a free-form email into structured fields. It uses n8n's OpenAI node:

1. Create an **OpenAI API** credential in n8n with your API key.
2. Assign it to **Parse Email With AI**. The default model is `gpt-4o-mini`; swap for any chat-capable model your credential has access to.
3. No AI credential? Replace this node with a simpler **Code** node that parses a structured email format (e.g. a template the sender fills in) instead of free text.

## 6. Form intake

The **Meeting Request Form** trigger needs no credential — once the workflow is active, n8n hosts the form at the URL shown on that node (Production URL after activation, Test URL while editing).

## 7. Tune the business logic

Open the **Find Available Slot** code node to adjust:

- `WORK_START_HOUR` / `WORK_END_HOUR` — the business hours slots are searched within (default 9–17).
- Weekend handling — currently skips Saturday/Sunday (`cursor.weekday <= 5`).
- `MAX_ALTERNATIVES` — how many backup slots to suggest by email when the exact request can't be met.
- The search looks up to 14 days ahead; change the `searchLimit`/`windowEnd` calculations in **Compute Search Window** and **Find Available Slot** together if you need a wider or narrower range.

## 8. Google Meet links (optional)

`Book Meeting` doesn't request conference data by default, to keep the imported JSON portable across n8n/Google API versions. To auto-generate a Google Meet link per booking, open the **Book Meeting** node and enable **Conference Data → Create Conference** in the UI — the **Prepare Confirmation Details** node already knows how to pick up `hangoutLink`/`conferenceData` if present.

## 9. Activate and test

1. Activate the workflow.
2. Submit the form (or send a test email to the connected inbox) with a request you know is inside your working hours.
3. Confirm the event appears on the calendar and the confirmation email arrives.
4. To test the reminder sequence without waiting hours/days, temporarily book a slot a few minutes out and shrink the `Wait` node offsets (e.g. `minutes: 2` instead of `hours: 24`) — then restore the originals afterward.
