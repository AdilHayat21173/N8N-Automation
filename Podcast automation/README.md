# Newsletter Workflow (n8n)

## Summary

This n8n workflow automates the process of transforming new **Riverside.fm** podcast recordings into a polished newsletter package.  
It runs on a schedule, fetches the latest recording, extracts the transcript, and stores it in Google Sheets.  
AI agents then clean and summarize the transcript, extract insights and quotes, and draft a 400-word newsletter in **Kathryn Finney’s voice**.  

The workflow also generates a branded image using **Google Gemini**, uploads it to **Google Drive**, and finally sends a **Gmail approval email** with Approve/Disapprove buttons.  
Approved content is finalized and emailed to the team with the newsletter draft and embedded image.

---

## How It Works (Step-by-Step)

### 1) Trigger on a schedule
- **Node:** Schedule Trigger → every 12 hours.

### 2) Fetch Riverside recordings
- **Node:** HTTP Request → `GET https://platform.riverside.fm/api/v2/recordings` with `Accept: application/json` and `Authorization: BEARER …`.

### 3) Pick the newest recording
- **Node:** Code in JavaScript → sorts by `created_date` (desc) and exposes `transcript_txt_url` / `transcript_srt_url` if present.

### 4) Download the TXT transcript
- **Node:** HTTP Request1 → `URL = {{ $json.transcript_txt_url }}`; response format = file.

### 5) Extract plain text
- **Node:** Extract from File → operation = `text` to read the downloaded transcript.

### 6) Store the transcript
- **Node:** Append row in sheet1 (Google Sheets) → document `1sl0fq01ON8ksNc2xHgBc8_AnZviZSmPZYKnRJTslVYo`, Sheet1; maps `id` and `full transcript`.

### 7) Clean + summarize
- **Node:** AI Agent → remove timestamps/fillers; output 3–5 themes + 2–3 sentence intro (plain text).

### 8) Extract insights, emotions, quotes
- **Node:** AI Agent1 → 3 key insights, 2 emotional highlights, 2 short quotes (bullets).

### 9) Draft the newsletter (400 words)
- **Node:** AI Agent2 → Kathryn Finney’s voice; bolded headers, bullets, strong CTA.

### 10) Turn text into an image prompt
- **Node:** AI Agent3 → instructs a graphic, non-human scene prompt with palette/overlay text rules.

### 11) Generate the image
- **Node:** Generate an initial image (Google Gemini) → model `models/gemini-2.5-flash-image-preview`, binary output.

### 12) Resize and upload
- **Nodes:** Edit Image → 800×400  
- Upload file (Drive folder **Asanapic**)  
- Share file (anyone → reader) to embed.

### 13) Review & approval via Gmail
- **Node:** Approve & Disapprove Post → sends email with image + newsletter and YouTube/Apple links.  
- **Node:** If1 → routes approved vs revise paths.

### 14) Optional edits
- **Nodes:** Mail For Update Text / Mail for Update Image → request changes via custom form; classify replies as continue or revise.

### 15) Final emails
- **Nodes:** Final response / Final Mail after Editing → sends approved content with embedded Drive image.

---

## Configuration

- **Riverside Auth:** Replace inline Authorization headers with a Credential reference.  
- **Google Sheets:** Confirm the spreadsheet ID and that Sheet1 has columns `id` and `full transcript`.  
- **Drive Folder:** Uploads go to folder **Asanapic** (`1oHLnyIxBW1rkJWEFzHTqbLs0c7xIVeTw`).  
- **Sharing:** Files are shared as `anyone / reader` (optional `allowFileDiscovery: true` in the second path).  
- **Models:** OpenAI chat nodes use `gpt-4.1-mini`; Gemini nodes use `models/gemini-2.5-flash-image-preview`.

---

## Email Templates (built-in)

- **Approval email:** Draft + image + YT/Apple links; Approve / Disapprove buttons.  
- **Text/image update emails:** Include Drive preview button and capture reviewer feedback.

---

## Security Checklist

- Move all tokens/keys to **n8n Credentials**; never hardcode secrets in nodes.  
- Limit Drive sharing to the minimum necessary; prefer `reader` access and disable discovery unless needed.
