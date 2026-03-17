# LocalVex Media — n8n Workflow Templates

Production-ready n8n automation workflows for LocalVex Media's lead generation, client communication, and reputation management stack.

## Folder Structure

```
n8n-templates/
├── README.md                          ← You are here
└── workflows/
    ├── fb-lead-to-ghl.json            ← WF1: Facebook Lead → GHL Contact
    ├── welcome-sequence-trigger.json   ← WF2: 3-Email Welcome Drip Sequence
    ├── weekly-report-emailer.json      ← WF3: Monday Morning Performance Report
    ├── missed-call-text-back.json      ← WF4: Missed Call → Instant SMS + Reminder
    ├── review-request-sequence.json    ← WF5: Post-Service Review Request Automation
    └── credentials-template.json       ← Credential placeholders for all workflows
```

## Workflow Summaries

| # | File | Trigger | What It Does |
|---|------|---------|--------------|
| 1 | `fb-lead-to-ghl.json` | Facebook Webhook | Captures FB lead ad submissions → creates GHL contact → adds to pipeline → sends instant SMS |
| 2 | `welcome-sequence-trigger.json` | GHL Webhook (tag: new-lead) | Sends 3-email nurture sequence over 3 days → tags contact "sequence-complete" |
| 3 | `weekly-report-emailer.json` | Schedule (Monday 8 AM) | Pulls weekly stats from GHL → compiles report → emails to you + active clients |
| 4 | `missed-call-text-back.json` | GHL Webhook (missed call) | Detects business hours → sends appropriate SMS → moves pipeline stage → 30-min reminder |
| 5 | `review-request-sequence.json` | GHL Webhook (tag: service-complete) | Waits 2 hours → SMS review request → 2-day email follow-up → logs to Google Sheets |

## How to Import Workflows into n8n

1. Open your n8n instance (self-hosted or n8n.cloud)
2. Click **Workflows** in the left sidebar
3. Click **Add Workflow** (top right)
4. In the new workflow, click the **⋮** menu (three dots, top right)
5. Select **Import from File**
6. Choose the `.json` file from this folder
7. The workflow loads with all nodes and connections pre-configured
8. Configure credentials (see below), then **Activate** the workflow

## Required Credentials by Workflow

| Credential | WF1 | WF2 | WF3 | WF4 | WF5 | How to Get |
|------------|-----|-----|-----|-----|-----|------------|
| **GHL API Key** | ✅ | ✅ | ✅ | ✅ | ✅ | GHL → Settings → Business Profile → API Key |
| **SMTP Email** | ✅ | ✅ | ✅ | ✅ | ✅ | Gmail: Account → Security → App Passwords |
| **Facebook App** | ✅ | | | | | developers.facebook.com → Create App → Webhooks |
| **Google Sheets OAuth** | | | | | ✅ | Google Cloud Console → APIs → Sheets API → OAuth |
| **Google Review Link** | | | | | ✅ | Google Business Profile → Get more reviews |

### GHL IDs You'll Need

These aren't credentials but are referenced as placeholders in the workflow JSON:

| Placeholder | Where to Find It |
|-------------|-----------------|
| `{{GHL_API_KEY}}` | GHL → Settings → Business Profile → API Key |
| `{{GHL_PIPELINE_ID}}` | GHL → Settings → Pipelines → click pipeline → ID in URL |
| `{{GHL_STAGE_ID}}` | GHL → Settings → Pipelines → click stage → ID in URL |
| `{{GHL_ATTEMPTED_CONTACT_STAGE_ID}}` | Same as above, for "Attempted Contact" stage |
| `{{GHL_LOCATION_ID}}` | GHL → Settings → Business Profile → Location ID |
| `{{GOOGLE_SHEET_ID}}` | From the Google Sheets URL: `docs.google.com/spreadsheets/d/{THIS_PART}/edit` |
| `{{GOOGLE_REVIEW_LINK_PLACEHOLDER}}` | Google Business Profile → Home → Get more reviews |

## Quick-Start Guide (5 Steps to First Workflow Live)

### Step 1: Set Up n8n

If you don't have n8n running yet:

```bash
# Docker (recommended for self-hosted)
docker run -it --rm --name n8n -p 5678:5678 -v n8n_data:/home/node/.n8n n8nio/n8n

# Or sign up at https://n8n.cloud for hosted version
```

Open `http://localhost:5678` in your browser.

### Step 2: Add Your Credentials

1. Go to **Settings → Credentials** in n8n
2. Add **SMTP** credential:
   - Name: `Gmail SMTP`
   - Host: `smtp.gmail.com`, Port: `465`, Secure: `true`
   - User: your Gmail, Password: your App Password
3. Add **Header Auth** credential (for GHL):
   - Name: `GoHighLevel API`
   - Header Name: `Authorization`
   - Header Value: `Bearer YOUR_API_KEY`

### Step 3: Import Your First Workflow

Start with **Workflow 4 (Missed Call Text Back)** — it's the simplest and gives immediate value.

1. Click **Workflows → Add Workflow → ⋮ → Import from File**
2. Select `missed-call-text-back.json`
3. Find-and-replace `{{GHL_API_KEY}}` with your actual key in each HTTP Request node
4. Find-and-replace `{{GHL_PIPELINE_ID}}` and `{{GHL_ATTEMPTED_CONTACT_STAGE_ID}}`

### Step 4: Configure the GHL Webhook

1. **Activate** the workflow in n8n — copy the webhook URL shown
2. In GHL → Settings → Phone Numbers → your number
3. Set the Missed Call Webhook URL to the n8n URL
4. Save

### Step 5: Test It

1. Call your GHL phone number from a test phone
2. Don't answer — let it ring to voicemail
3. Check n8n → Executions to see the workflow fire
4. Verify the SMS was sent and the reminder email arrived

You're live! Repeat Steps 3–5 for each additional workflow.

## Troubleshooting

| Problem | Fix |
|---------|-----|
| Webhook not firing | Make sure the workflow is **Activated** (toggle in top right). Inactive workflows don't listen for webhooks. |
| GHL API returns 401 | API key is wrong or expired. Regenerate in GHL → Settings → Business Profile. |
| Emails not sending | Check SMTP credentials. For Gmail, you must use an App Password (not your regular password). Enable 2FA first. |
| Google Sheets "permission denied" | Re-authenticate the Google Sheets credential in n8n. Make sure the Sheet is accessible by the Google account you connected. |
| Wait nodes not resuming | n8n must stay running for Wait nodes to resume. If using Docker, ensure the container doesn't restart. Use `--restart unless-stopped`. |
| Facebook webhook verification fails | The webhook must respond to GET requests with the `hub.challenge` value. n8n's Webhook node handles this automatically when active. |

## Architecture Notes

- All workflows use **GoHighLevel REST API v1** (`https://rest.gohighlevel.com/v1/`)
- GHL rate limit: **100 requests/minute** — the Weekly Report workflow handles pagination
- Wait nodes require n8n to be running continuously (they're stored in n8n's internal DB)
- All SMS is sent through GHL's built-in phone system (Twilio under the hood) — no separate Twilio account needed
- Email sending uses direct SMTP — no email service API required (Gmail App Password works fine for low volume)
