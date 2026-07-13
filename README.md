# Job Hunting Automation (N8N)

A fully local, free-tier N8N workflow that:
- Scrapes junior performance-marketing jobs from RapidAPI JSearch & Apify LinkedIn
- Tailors your master resume per job using Google Gemini
- Saves the tailored resume to Google Drive
- Notifies you on Telegram with an Apply Now button

## 📋 Overview

This N8N workflow automates a daily job-search pipeline:

- Fetch jobs from two sources (RapidAPI JSearch & Apify LinkedIn scraper) on a schedule
- Normalize & deduplicate the results
- For each job, tailor your master resume using Google's Gemini LLM
- Save the tailored resume as a text file in Google Drive
- Notify you via Telegram with an **Apply Now** button and a **Mark as Applied** button

Everything runs locally (self-hosted N8N) and only uses free-tier APIs (Gemini, Apify, RapidAPI, Telegram).

## 🔑 Required API Keys & Credentials

- **Gemini (Google Generative Language)**
  - What you need: API key (passed via HTTP header `x-goog-api-key`)
  - Where to get it: [makersuite.google.com/app/apikey](https://makersuite.google.com/app/apikey) (or [ai.google.dev/gemini-api](https://ai.google.dev/gemini-api))

- **Telegram Bot**
  - What you need: Bot token & your chat ID
  - Where to get it: Create a bot with [@BotFather](https://t.me/BotFather) → get token. Get chat ID from [@userinfobot](https://t.me/userinfobot), or by messaging the bot and checking `https://api.telegram.org/bot<token>/getUpdates`

- **Apify**
  - What you need: API token (for the LinkedIn job-scraper actor)
  - Where to get it: [console.apify.com/account/integrations](https://console.apify.com/account/integrations)

- **RapidAPI (JSearch)**
  - What you need: API key (header `x-rapidapi-key`)
  - Where to get it: Sign up at [rapidapi.com](https://rapidapi.com) → search "JSearch" → subscribe to the free plan → copy the key

- **Google Drive** (for master resume & output)
  - What you need: OAuth credentials or Service Account (already referenced in the workflow)
  - Where to get it: In N8N, create a Google Drive credential (OAuth2 or Service Account). The workflow expects a credential named **Google Drive account**. You can reuse or create your own and update the node IDs.

> **Tip:** Store all secrets in N8N's Credentials manager (never hard-code them in node fields). The workflow already uses credential placeholders — just create matching credentials.

## 🛠️ Step-by-Step Setup

### 1. Install & Run N8N Locally

```bash
# Using Docker (recommended for quick start)
docker run -it --rm \
  -p 5678:5678 \
  -v ~/.n8n:/home/node/.n8n \
  n8nio/n8n
```

Open `http://localhost:5678` and follow the wizard to create an admin user.

### 2. Import the Workflow

- Save the workflow JSON (`Job-workflow-automation.json`) to your computer
- In N8N UI → **Workflows** → **Import** → select the file
- The workflow appears as "Job Hunting - Mark as Applied Confirmation"

### 3. Configure Credentials

- **Header Auth account** (used by Gemini) → Type: HTTP Header Auth → add header `x-goog-api-key` with your Gemini API key
- **Telegram account** → Type: Telegram → Bot Token from BotFather
- **Google Drive account** → Type: Google Drive → OAuth2 (or Service Account) — grant access to Drive
- *(Optional)* Apify token can stay embedded in the node, or for better security, move it to an HTTP Header Auth credential and update the Apify node to use it

> After creating each credential, open the corresponding node, select the credential from the dropdown, and save the workflow.

### 4. Adjust Hard-coded Values (if needed)

- **Fetch New Job Listings (RapidAPI)** — the `x-rapidapi-key` header currently holds a demo key. Replace with your own RapidAPI key (or move to a credential).
- **Fetch LinkedIn Jobs (Apify)** — the Apify token is embedded. Replace with your own Apify API token (or use a credential).
- **Get Master Resume** — the Google Drive file URL points to a sample doc. Change it to your master resume (Google Doc), keeping the same node settings (download as plain text).
- **Create file from text** — the `folderId` points to a sample Drive folder. Replace with the ID of the folder where tailored resumes should be saved.
- **Send Apply Prompt & Notify nodes** — the `chatId` is set to a placeholder. Replace with your own Telegram chat ID (integer).
- **Schedule nodes** — cron expressions are set for `30 11 * * *` (11:30 AM) and `0 16 * * *` (4:00 PM) in Asia/Kolkata timezone. Adjust timezone or times as needed.

### 5. Test the Workflow

- Click **Execute Workflow** (top-right)
- Monitor the execution — it will fetch jobs, process the first batch, and send you a Telegram message with an Apply Now link
- If Gemini returns a 429 (rate-limit), you'll get a "Session limit reached" Telegram notice and the workflow stops for the day
- Check your Google Drive folder — a `.txt` file named `<Company> - <Job Title> Resume.txt` should appear with the tailored resume

### 6. Activate Automatic Scheduling

- Toggle the workflow **Active** switch (top-left) to ON
- The two Schedule triggers will now run automatically at the specified times each day

## 📖 How It Works (Plain English)

- **Schedule Trigger** — at 11:30 AM (JSearch) and 4:00 PM (Apify), the workflow wakes up
- **Fetch Jobs** — calls RapidAPI JSearch (keyword "junior performance marketing fresher jobs in India") or Apify LinkedIn scraper (Location: Chennai, India, last 24h, easy-apply, max 20)
- **Normalize** — code nodes convert each API's response into a common JSON shape: `{job_id, title, company, description, job_url, source}`
- **Merge & Dedup** — both sources are combined, then duplicates (same company + title) are removed
- **Loop Over Jobs** — for each unique job:
  - Download your master resume from Google Drive (as plain text)
  - Build a prompt for Gemini: "Tailor this resume for the job below…"
  - Call Gemini 2.5-Flash (free tier) to get a tailored resume
  - If Gemini returns HTTP 429 (daily free-tier limit), send a Telegram warning and stop today's run
  - Extract the tailored text from Gemini's response
  - Save that text as a new `.txt` file in your Google Drive folder (named after company & title)
  - Send a Telegram message with: job title & company, an Apply Now button (links to the posting), and a Mark as Applied button (callback data for your own tracking)
- **End** — if saving the file fails, you get a failure notice via Telegram

## 🔑 API Keys & Where to Get Them

- **Gemini API** — LLM for resume tailoring → [makersuite.google.com/app/apikey](https://makersuite.google.com/app/apikey)
- **Telegram Bot** — Bot token & chat ID → Create via [@BotFather](https://t.me/BotFather); get chat ID from [@userinfobot](https://t.me/userinfobot) or `https://api.telegram.org/bot<token>/getUpdates`
- **Apify** — Token for LinkedIn-job-scraper actor → [console.apify.com/account/integrations](https://console.apify.com/account/integrations)
- **RapidAPI (JSearch)** — Key for job search endpoint → Sign up at [rapidapi.com](https://rapidapi.com) → search "JSearch" → subscribe → copy key
- **Google Drive** — Read master resume / write tailored resumes → Set up via N8N Google Drive credential (OAuth2 or Service Account)

## 📝 Notes

- The workflow runs entirely self-hosted; no cloud-only services are required
- Free tiers: Gemini 2.5-Flash (generous daily limit), Apify $5/mo credit (covers scraper usage at this volume), RapidAPI free tier (enough for a few requests/day), Telegram is free
- If you hit the Gemini daily quota, the workflow will notify you via Telegram and pause until the next day
- Feel free to adjust the search keywords, location, or schedule to match your job-hunting preferences

Happy hunting! 🎯

## 🎉 You're all set!

Run the workflow, watch your Telegram for tailored-resume prompts, and apply with one click. If you need tweaks (different job titles, locations, or notification style), just edit the corresponding nodes and re-save the workflow.

*Happy automating!*
