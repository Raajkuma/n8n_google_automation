# Automated Email Notification Workflow with n8n, Google Sheets & Ollama LLM

An automated n8n workflow designed to fetch student assessment data from Google Sheets, utilize a local LLM via **Ollama** (`qwen3-vl:2B-BF16`) to draft personalized and professional email updates, parse the generated content, and dispatch emails automatically using **Gmail**.

---

## 📌 Overview

This workflow automates the communication pipeline for student assessment results. It performs the following sequence:

1. **Triggering**: Initiated manually or scheduled via n8n.
2. **Data Extraction**: Pulls recipient and assessment data from a Google Sheet.
3. **AI Generation**: Feeds the row data into a local LLM via Ollama using a structured prompt to generate a JSON response containing the recipient email address, subject line, and concise body text.
4. **Data Parsing**: Converts the raw LLM output string into a structured JSON object using a JavaScript Code Node.
5. **Email Dispatch**: Sends the generated email via Gmail.

---

## 🛠️ Workflow Architecture

```
[ Manual Trigger ]
        │
        ▼
[ Google Sheets Node ] ── (Fetches Student Data: Name, Course, Score, Status, EmailID)
        │
        ▼
[ Basic LLM Chain ] ◄──── [ Ollama Chat Model ] (qwen3-vl:2B-BF16)
        │
        ▼
[ JavaScript Code Node ] ── (Parses LLM String output into clean JSON)
        │
        ▼
[ Gmail Node ] ─────────── (Sends Email to recipient)
```

---

## 📋 Prerequisites

Before importing and running this workflow, ensure you have the following setup:

1. **n8n Instance**: Running n8n (v1.0 or higher recommended).
2. **Ollama Instance**: Running locally or hosted on an accessible network endpoint.
   - **Model Required**: `qwen3-vl:2B-BF16` (or any compatible chat model such as `llama3` or `mistral`).
3. **Google Services Account / Credentials**:
   - **Google Sheets OAuth2 Credential**: Configured in n8n with access to read spreadsheet data.
   - **Gmail OAuth2 Credential**: Configured in n8n with permission to send emails (`https://www.googleapis.com/auth/gmail.send`).

---

## 🚀 Setup & Installation

### 1. Import the Workflow
1. Open your n8n Dashboard.
2. Go to **Workflows** > **Add Workflow**.
3. Click the **`⋮` (Menu)** button in the top right corner and select **Import from File**.
4. Select `Mail Automation (1).json`.

### 2. Configure Credentials

#### A. Google Sheets
- Double-click the **Get row(s) in sheet** node.
- Select your **Google Sheets OAuth2** credential.
- Update the **Document ID** (Spreadsheet URL/ID) and **Sheet Name** to point to your tracking sheet.
- Expected Sheet Headers:
  - `Name`
  - `Course`
  - `Score`
  - `Status`
  - `EmailID`

#### B. Ollama Model
- Double-click the **Ollama Chat Model** node.
- Select or create your **Ollama API** credential (host address, e.g., `http://localhost:11434`).
- Ensure the model `docker.io/ai/qwen3-vl:2B-BF16` (or your chosen model name) is pulled and running in your Ollama environment.

#### C. Gmail
- Double-click the **Send a message** node.
- Select your **Gmail OAuth2** credential.

---

## 🤖 LLM Prompt & Rules Logic

The workflow uses a **Basic LLM Chain** node driven by system-defined rules to ensure consistent outputs:

- **Input Fields**:
  - `Name`
  - `Course`
  - `Score`
  - `Status`
  - `EmailID`
- **Output Requirements**:
  - Valid, unformatted raw JSON object (no markdown code blocks).
  - Schema keys: `Subject`, `To`, `Message`.
  - Max body word limit: 50–100 words.
  - Strict tone and fallback guidelines for missing/invalid email addresses.

---

## 🧪 How to Test & Run

1. Open the workflow in n8n.
2. Ensure your Google Sheet contains valid sample records.
3. Click **Execute Workflow** at the bottom of the canvas.
4. Inspect the output of each node:
   - **Google Sheets**: Confirm student records are retrieved.
   - **Basic LLM Chain**: Confirm JSON string creation.
   - **Code in JavaScript**: Verify `JSON.parse()` extracts `Subject`, `To`, and `Message`.
   - **Send a message**: Check for successful Gmail delivery.

---

## ⚙️ Customization

- **Change Trigger**: Replace the `When clicking ‘Execute workflow’` manual trigger with a **Cron / Schedule Trigger** or a **Google Sheets Trigger** (e.g., *On Row Added*).
- **Change Model**: Modify the **Ollama Chat Model** parameters to swap out models or adjust the temperature (default is set to `0.7`).