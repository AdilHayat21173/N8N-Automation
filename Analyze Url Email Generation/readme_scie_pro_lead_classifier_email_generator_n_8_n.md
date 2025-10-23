# SciePro Lead Classifier & Email Generator — n8n Workflow

This repository contains an **n8n workflow** that:

1. Pulls the next prospect from a Google Sheet (one per run).
2. Crawls the prospect’s website and extracts basic content.
3. Classifies the organization into one of seven predefined tags.
4. Looks up a matching **email template** by tag from a second sheet.
5. Generates a personalized **subject + body** using LLMs.
6. Writes the **New Category** and the composed **Email Template** back to the prospect row in Sheet1.

> **Note:** This workflow does **not send emails**. It prepares ready‑to‑send copy and stores it in your Sheet.

---

## Contents
- [Architecture](#architecture)
- [Prerequisites](#prerequisites)
- [Google Sheets Structure](#google-sheets-structure)
- [How It Works (Node by Node)](#how-it-works-node-by-node)
- [Setup Instructions](#setup-instructions)
- [Customization](#customization)
- [Running & Monitoring](#running--monitoring)
- [Troubleshooting](#troubleshooting)
- [Security & Privacy](#security--privacy)
- [Limitations](#limitations)
- [FAQ](#faq)

---

## Architecture

```
Schedule Trigger (every 1 min)
   └─► Google Sheets: Get row(s) in sheet  [Sheet1; pick next prospect with Email Template == "="]
        └─► HTTP Request [GET Website]
             └─► HTML Extract [title, h1, meta, headings, body]
                  └─► AI Agent [Classify into one Tag + produce Summary]
                       └─► Google Sheets: Get row(s) in sheet1  [Sheet4; fetch template by Tag]
                            └─► AI Agent1 [Compose Subject + Body using Tag, Summary, Template]
                                 └─► Google Sheets: Update row in sheet [Sheet1; write New Category + Email Template]

(OpenAI Chat Model node provides the LLM to both agents.)
```

---

## Prerequisites

- **n8n** (self‑hosted or n8n Cloud)
- **Google Sheets OAuth2** credential with read/write access to the target spreadsheet
- **OpenAI API** credential
- Access to the Google Sheet: `SciePro Prospect List` (Document ID: `1xahAEayzGh8FchIVpjfN932y2OSIcBc_GX_v7EX0iY4`)

> You can rename the spreadsheet or move it; just update the nodes’ *Document ID* and *Sheet* settings accordingly.

---

## Google Sheets Structure

This workflow expects **two tabs** in the same spreadsheet:

### 1) `Sheet1` (gid=0) — Prospect List
Required columns (case‑sensitive, including spacing):

| Column                    | Purpose                                                                                     |
|---------------------------|---------------------------------------------------------------------------------------------|
| `Business Name`           | Prospect company/org name (optional but helpful)                                            |
| `Website`                 | Full URL to crawl (e.g., `https://example.com`)                                             |
| `Category`                | Existing category (optional)                                                                 |
| `Contact First Name`      | For personalizing the greeting                                                               |
| `Contact Last Name`       | Optional                                                                                    |
| `Title`                   | Optional                                                                                    |
| `Email`                   | **Unique identifier** used to match the row during update                                   |
| `Linkedin`                | Optional                                                                                    |
| `Email Template `         | **Must contain a trailing space** in the header. Set to a literal `=` for rows to process.  |
| `New Category`            | Will be written with the classifier’s tag                                                   |

> **Important:** The header **`Email Template ` (with a trailing space)** is intentional and must match exactly. The workflow filters on this column with value `=` to pick the next unprocessed row.

### 2) `Sheet4` (gid=2076602539) — Template Bank
This sheet stores **email templates keyed by Tag**. Recommended columns:

| Column  | Purpose                                                               |
|---------|-----------------------------------------------------------------------|
| `Tag`   | One of the 7 fixed tags (see below)                                   |
| `Email` | The **template text** the model should follow (structure/style/voice) |

**Allowed Tags (exactly one is chosen by the classifier):**
- Universities/Institutions
- Medical animation studios
- Legal animation studios
- 3D print companies
- App developers
- Medtech, pharma, and healthcare marketing teams
- Other/Unknown

**Example `Email` template value (stored in a single cell):**
```
Subject: {{PRODUCT/RELEVANCE ONE-LINER}}

Hi {{FirstName}},

{{Opening tailored to their org type}}. {{Value prop + proof}}.

If helpful, I can share a quick example relevant to {{their field}}.

Best,
{{Your Name}}
```
> The composer will inject the first name from `Contact First Name` and shape content using the Summary + Tag.

---

## How It Works (Node by Node)

Below is a concise mapping of the provided JSON to behavior.

### 1) **Schedule Trigger** (`n8n-nodes-base.scheduleTrigger`)
- **Interval:** every **1 minute**.
- Kicks off one iteration per run.

### 2) **Get row(s) in sheet** (`n8n-nodes-base.googleSheets` @ `Sheet1`, gid=0)
- **Filter:** `Email Template ` == `=`
- **Option:** `returnFirstMatch: true` ⇒ picks **one** row per run.

### 3) **HTTP Request1** (`n8n-nodes-base.httpRequest`)
- **URL:** `{{$json.Website}}` from the selected row.
- Follows redirects.

### 4) **HTML1** (`n8n-nodes-base.html`)
- **Operation:** `extractHtmlContent`
- **Selectors:** `title`, `h1`, `meta[name="description"]`, `meta[property="og:type"]`, `link[rel="canonical"]`, `nav a`, `h2,h3`, `script[src]`, `a[href]`, and main text from `main/article/body`.
- **Options:** trims values and cleans text.

### 5) **OpenAI Chat Model** (`@n8n/n8n-nodes-langchain.lmChatOpenAi`)
- **Model:** `gpt-4.1-mini` (provided to both agents below).

### 6) **AI Agent** (`@n8n/n8n-nodes-langchain.agent`)
- **Role:** Strict **lead classifier** for SciePro.
- **Input:** Title/H1/Meta/Body text from HTML node.
- **Output (JSON):** `{ "tag": "…", "summary": "…" }`
- **Decision Rules:** Fixed 7-tag schema (see list above).
- Output is validated by **Structured Output Parser**.

### 7) **Get row(s) in sheet1** (`n8n-nodes-base.googleSheets` @ `Sheet4`, gid=2076602539)
- **Filter:** `Tag == {{ $json.output.tag }}` from the classifier.
- Returns the **email template** to follow.

### 8) **AI Agent1** (`@n8n/n8n-nodes-langchain.agent`)
- **Role:** **Personalized Email Generator**.
- **Inputs:**
  - `Tag` (from classifier)
  - `Email` (template text from `Sheet4`)
  - `Summary` (from classifier)
  - `Contact First Name` (from original prospect row; used to personalize)
- **Output (JSON):** `{ "subject": "…", "body": "…" }`
- Validated by **Structured Output Parser1**.

### 9) **Update row in sheet** (`n8n-nodes-base.googleSheets` @ `Sheet1`, gid=0)
- **Matching column:** `Email` (acts as primary key)
- **Writes:**
  - `New Category` = `{{ $('Get row(s) in sheet1').item.json.Tag }}`
  - `Email Template ` =
    ```
    Subject: {{ $json.output.subject }}

    Body: {{ $json.output.body }}
    ```
  - Leaves other fields intact.

---

## Setup Instructions

1. **Import the Workflow**
   - In n8n, go to **Workflows ▶ Import from File/Clipboard** and paste the JSON.

2. **Attach Credentials**
   - **Google Sheets OAuth2:** Create/choose an account with access to the spreadsheet. Attach it to all Google Sheets nodes.
   - **OpenAI API:** Create/choose a credential and attach to the **OpenAI Chat Model** node.

3. **Verify Spreadsheet IDs & Tabs**
   - `Document ID`: `1xahAEayzGh8FchIVpjfN932y2OSIcBc_GX_v7EX0iY4`
   - `Sheet1` (gid=0) and `Sheet4` (gid=2076602539). Adjust if you rename/move tabs.

4. **Prepare Sheet Headers**
   - Ensure `Email Template ` (with a trailing space) exists on **Sheet1**.
   - Seed `Sheet4` with your email templates (one row per Tag).

5. **Mark Rows to Process**
   - In **Sheet1**, set `Email Template ` to a literal `=` for any row you want processed.

6. **Enable the Workflow**
   - Toggle **Active**. It will run every minute by default.

---

## Customization

- **Classification Tags:**
  - Update rules in the **AI Agent** system prompt to add/refine categories.

- **Email Templates:**
  - Add more rows to `Sheet4` per Tag. You can store multiple templates and extend selection logic (e.g., by locale/region) if desired.

- **Personalization Fields:**
  - Extend the **AI Agent1** prompt to use more columns (e.g., `Title`, `Business Name`). Remember to add those to the input mapping.

- **Run Frequency:**
  - Change the **Schedule Trigger** from 1 minute to another cadence.

- **Model Choice:**
  - Swap `gpt-4.1-mini` to your preferred model in the **OpenAI Chat Model** node.

- **Row Selection Policy:**
  - Currently picks the **first match**. For batching, disable `returnFirstMatch` and loop; or use a queue flag.

---

## Running & Monitoring

- Use **Executions** in n8n to view each run.
- Check the **Last Node** data to confirm the subject/body and the updated cells on **Sheet1**.
- If nothing happens, confirm that at least one row in **Sheet1** has `Email Template ` set to `=`.

---

## Troubleshooting

**No row selected**
- Ensure the header is exactly `Email Template ` (with trailing space) and the cell value is exactly `=`.

**HTTP fetch fails**
- Verify the `Website` URL includes `http(s)://` and is reachable.

**Classifier outputs invalid JSON**
- The Structured Output Parser should enforce JSON; if it still fails, reduce input text size or adjust the AI Agent’s system prompt for stricter formatting.

**Wrong Tag or generic “Other/Unknown”**
- Improve the **HTML extraction** coverage or tweak the classification rules with more examples/keywords.

**Email not personalized**
- Ensure `Contact First Name` is populated in **Sheet1**. Check AI Agent1’s system prompt mentions this field.

**Update to wrong row**
- The update matches on `Email`. Confirm `Email` values are unique and not blank.

**Rate limits / Costs**
- Runs every minute by default. Consider lowering frequency or adding a max‑runs guard to control LLM/API usage.

---

## Security & Privacy

- Website content is fetched over HTTP(S); avoid processing sensitive/internal URLs.
- Only store non‑sensitive summaries/classifications in Sheets.
- OpenAI inputs include website text and prospect metadata; ensure you have consent to process this data.

---

## Limitations

- Classification is heuristic and may miscategorize edge cases.
- Email generation quality depends on template quality and extracted context.
- Pages behind logins or heavy JS may yield sparse HTML extraction.

---

## FAQ

**Q: Does this send emails?**  
A: No. It composes and stores copy in `Email Template ` for a human to review/send via your CRM or email tool.

**Q: How do I process multiple rows per run?**  
A: Remove `returnFirstMatch`, iterate in a loop, or use a separate queue sheet.

**Q: Can I add more tags?**  
A: Yes—expand the rules in the classifier prompt and ensure `Sheet4` contains templates for each new tag.

**Q: Can I log versions?**  
A: Add a `Last Processed` timestamp and a `Version`/`Notes` column to Sheet1 and write to them in the update step.

---

### Quick Start (TL;DR)
1) Import the workflow  
2) Attach Google Sheets & OpenAI credentials  
3) Ensure `Sheet1` has the exact headers (incl. `Email Template ` with trailing space)  
4) Seed `Sheet4` with `Tag` + `Email` template rows  
5) Set `Email Template ` to `=` on any prospect you want processed  
6) Activate the workflow and watch executions

---

**Happy automating!**

