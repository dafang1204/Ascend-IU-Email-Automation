# Ascend IU Email Automation

A fully automated alumni event outreach pipeline built for Ascend IU, eliminating manual email tracking by orchestrating a 5-stage communication lifecycle from initial invite to thank-you.

---

## Tech Stack

- **[n8n](https://n8n.io/)** — Self-hosted workflow automation engine
- **[Airtable](https://airtable.com/)** — Alumni database and state tracker
- **Gmail API** — Creates email drafts at each stage for human review
- **Google Sheets API** — Stores form responses, polled by n8n for new submissions
- **Google Forms** — RSVP form alumni fill out
- **Claude API** - Generates Personalized Emails 

---

## Workflow Architecture

The automation is split into three independently triggered workflows:

### 1. Initial Invite Phase (Schedule Trigger)
Runs on a schedule and sends the first round of invites to all approved alumni.

### 2. Yes & No Response Phase (Watch Form Responses)
Polls Google Sheets for new form submissions. When a response is detected:
- **Yes** → Creates a Path A confirmation draft
- **No** → Creates a Path B declination draft
- Updates Airtable stage accordingly

### 3. No Response Follow-Up Phase (Daily No Response Check)
Runs daily. If an alumnus has been in the Invited stage for 7+ days with no form response, a Path C reminder draft is created.

### 4. Update Email Phase (Schedule Trigger)
On the scheduled event date, finds all confirmed alumni and creates an event update email draft (Blast 2).

### 5. Thank You Phase (Schedule Trigger)
After Blast 2 has been sent, finds all alumni in the Update Sent stage and creates a thank-you email draft (Blast 3).

---

## How It Works

Each alumnus progresses through a defined set of stages tracked in Airtable:

```
Invited → Confirmed / Declined / Reminded → Update Sent → Thank You Sent
```

The workflow uses `filterByFormula` queries in Airtable to ensure each branch only processes records in the correct state, preventing duplicate emails. All emails are staged as Gmail drafts before sending, allowing for human review.

**Deduplication logic:**
- `Stage 2 Sent = FALSE` prevents the form response branch from processing the same person twice
- `Draft Created = TRUE` prevents blast emails from being re-created if the workflow runs again

---

## Airtable Schema

| Field | Type | Description |
|---|---|---|
| Name | Text | Alumni full name |
| Email | Text | Alumni email address |
| Status | Single Select | `Approved` — eligible for outreach |
| Stage | Single Select | Current stage in the pipeline |
| Scheduled Date | Date | Event date used to trigger Blast 2 & 3 |
| Stage 2 Sent | Checkbox | Whether a Yes/No response email has been created |
| Last Email Sent | Text | Tracks which email was last sent |
| Last Email Sent Date | Date | Used for 7-day inactivity check |
| Draft Created | Checkbox | Whether a Blast 1 draft has been created |

**Stage values:**

| Stage | Meaning |
|---|---|
| `Invited` | Initial invite sent, awaiting response |
| `Confirmed` | Alumni responded Yes |
| `Declined` | Alumni responded No |
| `Reminded` | No response after 7 days, follow-up sent |
| `Update Sent` | Event update email sent (Blast 2) |
| `Thank You Sent` | Thank-you email sent (Blast 3) |

---

## Setup

To replicate this project you will need:

1. **n8n** — self-hosted instance running locally or on a server
2. **Airtable** — create a base with the schema above
3. **Google Cloud** — enable the Gmail API and Google Sheets API, create OAuth credentials
4. **Google Forms** — create an RSVP form linked to a Google Sheet
5. **Claude API** - Generates Customized Emails

Configure the following credentials in n8n:
- Airtable Personal Access Token
- Google OAuth2 (for Gmail and Google Sheets)

Update the workflow nodes with your own Airtable Base ID, Table ID, Google Sheet ID, and Gmail address.

