Job‑Hunter N8N Workflow – Simple README & Setup Guide

---
📋 Overview

This N8N workflow automates a daily job‑search pipeline:

1. Fetch jobs from two sources (RapidAPI JSearch & Apify LinkedIn scraper) on a schedule.
2. Normalize & deduplicate the results.
3. For each job, tailor your master resume using Google’s Gemini LLM.
4. Save the tailored resume as a text file in Google Drive.
5. Notify you via Telegram with an Apply Now button and a Mark as Applied button.

Everything runs locally (self‑hosted N8N) and only uses free‑tier APIs (Gemini, Apify, RapidAPI, Telegram).

---
🔑 Required API Keys & Credentials

Service: Gemini (Google Generative Language)
What you need: API key (passed via HTTP Header x-goog-api-key)
Where to get it: https://makersuite.google.com/app/apikey (or https://ai.google.dev/gemini-api)
────────────────────────────────────────
Service: Telegram Bot
What you need: Bot token & your chat ID
Where to get it: Create a bot with @BotFather (https://t.me/BotFather) → get token. Get chat ID by
messaging @userinfobot (https://t.me/userinfobot) or sending a message to the bot and checking
https://api.telegram.org/bot<token>/getUpdates.
────────────────────────────────────────
Service: Apify
What you need: API token (for the LinkedIn‑job‑scraper actor)
Where to get it: https://console.apify.com/account/integrations (API token)
────────────────────────────────────────
Service: RapidAPI (JSearch)
What you need: API key (header x-rapidapi-key)
Where to get it: Sign up at https://rapidapi.com → search “JSearch” → subscribe to the free plan → copy the
key.
────────────────────────────────────────
Service: Google Drive (for master resume & output)
What you need: OAuth credentials or Service Account (already referenced in the workflow)
Where to get it: In N8N, create a Google Drive credential (OAuth2 or Service Account) and note its ID – the
 workflow expects credentials named Google Drive account (ID aYfa0k6BnRn9j3HS in the example). You can
reuse or create your own and update the node IDs.

▎ Tip: Store all secrets in N8N’s Credentials manager (never hard‑code them in node fields). The workflow already uses credential placeholders; just create matching credentials.

---
🛠️ Step‑by‑Step Setup

1. Install & Run N8N Locally

# Using Docker (recommended for quick start)
docker run -it --rm \
  -p 5678:5678 \
  -v ~/.n8n:/home/node/.n8n \
  n8nio/n8n
Open http://localhost:5678 and follow the wizard to create an admin user.

2. Import the Workflow

1. Save the JSON you just read (the file Job-workflow-automation.json) to your computer.
2. In N8N UI → Workflows → Import → select the file.
3. The workflow appears as “Job Hunting - Mark as Applied Confirmation”.

3. Configure Credentials

Node (as seen in workflow): Header Auth account (used by Gemini)
Credential name to create: Header Auth account
Type: HTTP Header Auth → Add header x-goog-api-key with your Gemini API key.
────────────────────────────────────────
Node (as seen in workflow): Telegram account
Credential name to create: Telegram account
Type: Telegram → Bot Token from BotFather.
────────────────────────────────────────
Node (as seen in workflow): Google Drive account
Credential name to create: Google Drive account
Type: Google Drive → OAuth2 (or Service Account) – grant access to Drive.
────────────────────────────────────────
Node (as seen in workflow): (Optional) If you prefer to keep the Apify token inside the node, you can leave
 it as‑is (the token is already in the JSON). For better security, move it to an HTTP Header Auth
credential and update the Apify node to use it.
Credential name to create:
Type:

▎ After creating each credential, open the corresponding node, select the credential from the dropdown, and Save the workflow.

4. Adjust Hard‑coded Values (if needed)

┌─────────────────────────┬───────────────────────────────────────────────────────────────────────────┐
│          Node           │                          What to check / change                           │
├─────────────────────────┼───────────────────────────────────────────────────────────────────────────┤
│ Fetch New Job Listings  │ The x-rapidapi-key header currently holds a demo key. Replace with your   │
│ (RapidAPI)              │ own RapidAPI key (or move to an HTTP Header Auth credential).             │
├─────────────────────────┼───────────────────────────────────────────────────────────────────────────┤
│ Fetch LinkedIn Jobs     │ The Apify token apify_api_E0LimLBL4QjKjwifXnrsfbRXzRq4Zl3ll3IB is         │
│ (Apify)                 │ embedded. Replace with your own Apify API token (or use a credential).    │
├─────────────────────────┼───────────────────────────────────────────────────────────────────────────┤
│                         │ The Google Drive file URL points to a sample doc. Change to the URL of    │
│ Get Master Resume       │ your master resume (Google Doc) – keep the same node settings (download   │
│                         │ as plain text).                                                           │
├─────────────────────────┼───────────────────────────────────────────────────────────────────────────┤
│ Create file from text   │ The folderId points to a sample Drive folder. Replace with the ID of the  │
│                         │ folder where you want tailored resumes saved.                             │
├─────────────────────────┼───────────────────────────────────────────────────────────────────────────┤
│ Send Apply Prompt &     │                                                                           │
│ Notify - Session        │ The chatId is set to 1075335869. Replace with your Telegram chat ID       │
│ Reached / Notify -      │ (integer).                                                                │
│ Submission Failed       │                                                                           │
├─────────────────────────┼───────────────────────────────────────────────────────────────────────────┤
│                         │ Cron expressions are set for 30 11 * * * (11:30 AM) and 0 16 * * *        │
│ Schedule nodes          │ (4:00 PM) in Asia/Kolkata timezone. Adjust timezone or times if needed    │
│                         │ via the node’s Timezone field.                                            │
└─────────────────────────┴───────────────────────────────────────────────────────────────────────────┘

5. Test the Workflow

1. Click Execute Workflow (top‑right).
2. Monitor the execution:
  - It will fetch jobs, process the first batch, and send you a Telegram message with an Apply Now link.
  - If Gemini returns a 429 (rate‑limit), you’ll get a “Session limit reached” Telegram notice and the workflow stops for the day.

3. Check your Google Drive folder – a .txt file named <Company> - <Job Title> Resume.txt should appear with the tailored resume.

6. Activate Automatic Scheduling

- Toggle the workflow Active switch (top‑left) to ON.
- The two Schedule triggers will now run automatically at the specified times each day.

---
📖 How It Works (Plain English)

┌───────────────┬─────────────────────────────────────────────────────────────────────────────────────┐
│     Step      │                                    What Happens                                     │
├───────────────┼─────────────────────────────────────────────────────────────────────────────────────┤
│ 1️⃣ Schedule   │ At 11:30 AM (JSearch) and 4:00 PM (Apify) the workflow wakes up.                    │
│ Trigger       │                                                                                     │
├───────────────┼─────────────────────────────────────────────────────────────────────────────────────┤
│               │ Calls RapidAPI JSearch (keyword “junior performance marketing fresher jobs in       │
│ 2️⃣ Fetch Jobs │ india”) or Apify LinkedIn scraper (Location: Chennai, India, last 24 h, easy‑apply, │
│               │  max 20).                                                                           │
├───────────────┼─────────────────────────────────────────────────────────────────────────────────────┤
│ 3️⃣ Normalize  │ Code nodes convert each API’s response into a common JSON shape: {job_id, title,    │
│               │ company, description, job_url, source}.                                             │
├───────────────┼─────────────────────────────────────────────────────────────────────────────────────┤
│ 4️⃣ Merge &    │ Both sources are combined, then duplicates (same company + title) are removed.      │
│ Dedup         │                                                                                     │
├───────────────┼─────────────────────────────────────────────────────────────────────────────────────┤
│ 5️⃣ Loop Over  │ For each unique job:                                                                │
│ Jobs          │                                                                                     │
├───────────────┼─────────────────────────────────────────────────────────────────────────────────────┤
│ 5a            │ Download your master resume from Google Drive (as plain text).                      │
├───────────────┼─────────────────────────────────────────────────────────────────────────────────────┤
│ 5b            │ Build a prompt for Gemini: “Tailor this resume for the job below…”.                 │
├───────────────┼─────────────────────────────────────────────────────────────────────────────────────┤
│ 5c            │ Call Gemini 2.5‑Flash (free tier) to get a tailored resume.                         │
├───────────────┼─────────────────────────────────────────────────────────────────────────────────────┤
│ 5d            │ If Gemini returns HTTP 429 (daily free‑tier limit), send a Telegram warning and     │
│               │ stop today’s run.                                                                   │
├───────────────┼─────────────────────────────────────────────────────────────────────────────────────┤
│ 5e            │ Extract the tailored text from Gemini’s response.                                   │
├───────────────┼─────────────────────────────────────────────────────────────────────────────────────┤
│ 5f            │ Save that text as a new .txt file in your Google Drive folder (named after company  │
│               │ & title).                                                                           │
├───────────────┼─────────────────────────────────────────────────────────────────────────────────────┤
│               │ Send you a Telegram message with: <br>• Job title & company <br>• Apply Now button  │
│ 5g            │ (links to the job posting) <br>• Mark as Applied button (callback data for your own │
│               │  tracking).                                                                         │
├───────────────┼─────────────────────────────────────────────────────────────────────────────────────┤
│ 6️⃣ End        │ If saving the file fails, you get a failure notice via Telegram.                    │
└───────────────┴─────────────────────────────────────────────────────────────────────────────────────┘

---
📄 Ready‑to‑Copy README (Markdown)

Create a file called README.md in the same folder as the workflow JSON and paste the following:

# Job Hunting Automation (N8N)

A fully local, free‑tier N8N workflow that:
- scrapes junior performance‑marketing jobs from RapidAPI JSearch & Apify LinkedIn,
- tailors your master resume per job using Google Gemini,
- saves the tailored resume to Google Drive,
- notifies you on Telegram with an Apply Now button.

## 🚀 Quick Start

1. **Install N8N** (Docker recommended):
   ```bash
   docker run -it --rm -p 5678:5678 -v ~/.n8n:/home/node/.n8n n8nio/n8n
2. Import Job-workflow-automation.json via the N8N UI → Workflows → Import.
3. Create Credentials (matching the names used in the workflow):
  - Header Auth account → HTTP Header Auth → x-goog-api-key: <YOUR_GEMINI_KEY>
  - Telegram account → Telegram Bot Token from @BotFather (https://t.me/BotFather)
  - Google Drive account → OAuth2 (or Service Account) – grant Drive access.
  - (Optional) Move the Apify & RapidAPI keys into HTTP Header Auth credentials for better security.
4. Edit Hard‑coded values (if needed):
  - Replace the Apify token in the Fetch LinkedIn Jobs (Apify) node.
  - Replace the RapidAPI key in the Fetch New Job Listings node.
  - Set your master‑resume Google Doc URL in Get Master Resume.
  - Set your target Drive folder ID in Create file from text.
  - Replace the Telegram chatId with your own numeric ID.
  - Adjust schedule timezones/times if you’re not in Asia/Kolkata.
5. Test: Click Execute Workflow. You should receive a Telegram message with an Apply Now button.
6. Activate: Toggle the workflow to ON – it will now run at 11:30 AM and 4:00 PM daily (Asia/Kolkata).

🔑 API Keys & Where to Get Them

┌────────────┬─────────────────────────┬──────────────────────────────────────────────────────────────┐
│  Service   │         Purpose         │                             Link                             │
├────────────┼─────────────────────────┼──────────────────────────────────────────────────────────────┤
│ Gemini API │ LLM for resume          │ https://makersuite.google.com/app/apikey                     │
│            │ tailoring               │                                                              │
├────────────┼─────────────────────────┼──────────────────────────────────────────────────────────────┤
│ Telegram   │                         │ Create via @BotFather (https://t.me/BotFather); get chat ID  │
│ Bot        │ Bot token & chat ID     │ from @userinfobot (https://t.me/userinfobot) or              │
│            │                         │ https://api.telegram.org/bot<token>/getUpdates               │
├────────────┼─────────────────────────┼──────────────────────────────────────────────────────────────┤
│            │ Token for               │                                                              │
│ Apify      │ LinkedIn‑job‑scraper    │ https://console.apify.com/account/integrations               │
│            │ actor                   │                                                              │
├────────────┼─────────────────────────┼──────────────────────────────────────────────────────────────┤
│ RapidAPI   │ Key for job search      │ Sign up at https://rapidapi.com → search “JSearch” →         │
│ (JSearch)  │ endpoint                │ subscribe → copy key                                         │
├────────────┼─────────────────────────┼──────────────────────────────────────────────────────────────┤
│ Google     │ Read master resume /    │ Set up via N8N Google Drive credential (OAuth2 or Service    │
│ Drive      │ write tailored resumes  │ Account)                                                     │
└────────────┴─────────────────────────┴──────────────────────────────────────────────────────────────┘

📝 Notes

- The workflow runs entirely self‑hosted; no cloud‑only services are required.
- Free tiers: Gemini 2.5‑Flash (generous daily limit), Apify $5/mo credit (covers the scraper usage at this volume), RapidAPI free tier (enough for a few requests per day), Telegram is free.
- If you hit the Gemini daily quota, the workflow will notify you via Telegram and pause until the next day.
- Feel free to adjust the search keywords, location, or schedule to match your job‑hunting preferences.

Happy hunting! 🎯

---

### 🎉 You’re all set!
Run the workflow, watch your Telegram for tailored‑resume prompts, and apply with one click. If you need any tweaks (different job titles, locations, or notification style), just edit the corresponding nodes and re‑save the workflow.

*Happy automating!*