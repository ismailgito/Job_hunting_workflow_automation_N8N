# Job Hunting Automation via n8n

**Repository:** [github.com/ismailgito/Job_hunting_workflow_automation_N8N](https://github.com/ismailgito/Job_hunting_workflow_automation_N8N)

This document details a fully local, free-tier n8n workflow designed to automate the daily job-search pipeline for junior performance marketing roles.

## Overview

* Automatically fetches job listings from two primary sources: RapidAPI JSearch and Apify LinkedIn scraper.
* Normalizes and deduplicates the gathered job results.
* Utilizes Google Gemini LLM to customize a master resume for each unique job description.
* Saves the tailored resume as a text file directly to Google Drive.
* Sends a notification to Telegram containing direct action buttons for applying and tracking.

## Required API Keys and Credentials

* **Gemini (Google Generative Language)**
  * Requirement: API key passed via the HTTP header `x-goog-api-key`.
  * Provisioning: Obtain via makersuite.google.com/app/apikey or ai.google.dev/gemini-api.

* **Telegram Bot**
  * Requirement: Bot token and personal chat ID.
  * Provisioning: Generate a bot via @BotFather to receive a token. Retrieve the chat ID using @userinfobot or by inspecting the bot updates endpoint.

* **Apify**
  * Requirement: API token dedicated to the LinkedIn job-scraper actor.
  * Provisioning: Access via console.apify.com/account/integrations.

* **RapidAPI (JSearch)**
  * Requirement: API key passed via the header `x-rapidapi-key`.
  * Provisioning: Register at rapidapi.com, subscribe to the free basic plan for JSearch, and copy the provided key.

* **Google Drive**
  * Requirement: OAuth2 credentials or a Service Account configured within n8n.
  * Provisioning: Establish a credential named "Google Drive account" in the n8n credential manager to allow secure access.

## Step-by-Step Setup

* **Install and Run n8n Locally**
  * Deploy using Docker via the command line to map local storage and expose the appropriate network ports.
  * Access the user interface at localhost to complete the initial admin account setup.

* **Import the Workflow**
  * Download the workflow JSON file to your local machine.
  * Navigate to the n8n interface, select Workflows, choose Import, and upload the file.

* **Configure Credentials**
  * Map the Header Auth account to your Gemini API key.
  * Link the Telegram account node to your specific Bot Token.
  * Authenticate the Google Drive account node using OAuth2 or Service Account details.
  * Ensure all credentials are saved properly within the centralized n8n manager rather than hard-coded.

* **Adjust Environment-Specific Values**
  * Update the RapidAPI node with your specific API key.
  * Replace the placeholder Apify token with your personal token.
  * Swap the sample Google Drive file URL with the actual URL of your master resume.
  * Configure the output node with the correct destination folder ID in Google Drive.
  * Replace placeholder chat IDs in the Telegram nodes with your authentic chat ID.
  * Modify the cron expressions to align with your preferred timezone and execution schedule.

* **Execute System Testing**
  * Select the Execute Workflow option in the top-right corner of the interface.
  * Verify that job data is fetched, processed, and that a sample notification arrives on Telegram.
  * Confirm that a tailored text file appears in the designated Google Drive directory.

* **Activate the Automation**
  * Toggle the active switch in the top-left corner to enable automatic, scheduled triggers.

## Operational Workflow

* **Schedule Trigger:** The system activates automatically based on the predefined cron schedules.
* **Data Ingestion:** The workflow calls the JSearch and Apify APIs to gather recent job postings based on target keywords and locations.
* **Data Normalization:** Custom code blocks transform varying API payloads into a standardized JSON schema containing the job ID, title, company, description, URL, and source.
* **Deduplication:** The workflow merges the inputs and filters out duplicate records sharing identical company names and titles.
* **Processing Loop:** For each unique job record, the system performs the following actions:
  * Downloads the master resume as plain text from Google Drive.
  * Formulates a specialized contextual prompt instructing the LLM to align the resume with the specific job description.
  * Submits the request to the Gemini free-tier model.
  * Gracefully handles rate-limiting scenarios by halting operations and sending an alert if daily quotas are reached.
  * Exports the refined resume text into a new text file labeled with the company and job title.
  * Dispatches a Telegram notification containing the job details alongside interactive tracking buttons.

## Known Limitations and Free-Tier Constraints

* **Gemini API (gemini-2.5-flash)**
  * The precise rate limits depend heavily on the project usage tier. Live statuses can be monitored via the Google AI Studio rate-limit dashboard.
  * Resets occur daily at midnight Pacific Time.
  * If paid tiers are required later, costs accrue based on input and output token volume.

* **JSearch via RapidAPI**
  * Limited to 200 requests per month on the free basic plan.
  * Resets occur on a monthly billing cycle rather than daily.
  * This represents a key volume bottleneck if multiple daily checks are scheduled.

* **Apify Platform**
  * Provides a five dollar platform credit per month on the free tier.
  * Computed charges apply per actor execution and compute unit. Unused credits do not roll over to subsequent months.

* **Telegram Bot API**
  * Standard operations allow approximately one message per second per individual chat, which easily accommodates this workflow.

* **Google Drive API**
  * Features high request ceilings, meaning storage capacity is the primary constraint.

* **Identified Optimization Areas**
  * Quota handling does not currently differentiate between short-term rate limits and complete daily exhaustion.
  * The JSearch node lacks a graceful degradation fallback if the monthly quota is met prematurely.
  * The deduplication logic evaluates entries within a single run but does not cross-reference against historical records from previous days.
