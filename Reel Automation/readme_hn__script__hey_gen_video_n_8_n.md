# Hacker News → Script → HeyGen Video — n8n Workflow

This workflow turns trending AI/LLM news into a **vertical avatar video** with on‑screen text and a generated background image, then polls HeyGen until the render completes.

It combines:
- **Hacker News** scraping (via n8n HN tools)
- **OpenAI** for script + captions
- **Google Gemini** for background image generation
- **Google Drive** for temporary asset hosting
- **HeyGen** API for avatar video rendering

> ⚠️ **This workflow does not download or publish the final video.** It polls for completion status. You can add a final step to fetch the result URL from HeyGen.

---

## Architecture

```
Manual Trigger
  └─► AI Agent (HN orchestrator prompt)
        ├─► HN: Fetch Front Page (AI keyword)
        └─► HN: Fetch Article (with comments)
           (uses OpenAI: Write Script)
                 └─► AI Agent2 → generate short captions (JSON list)
                       └─► Split Out captions (one item per caption)
                             └─► AI Agent3 → caption → image prompt (JSON)
                                   └─► Google Gemini → generate vertical image (binary)
                                         └─► Code (JS) → normalize binary + file name
                                               └─► Google Drive → upload image
                                                     └─► HeyGen v2 /video/generate (POST)
                                                           └─► HeyGen v1 /video_status.get (GET)
                                                                 └─► IF status==completed → stop
                                                                 └─► ELSE Wait(60s) → poll again
```

---

## Prerequisites

- **n8n** (self‑hosted or Cloud)
- **Credentials**
  - OpenAI API (for `Write Script` + Agents)
  - Google Gemini (image generation)
  - Google Drive OAuth2 (file upload)
  - HeyGen API (HTTP Header Auth; add the required auth header per HeyGen docs)
- **Enabled nodes**: `n8n-nodes-base.hackerNewsTool`, `@n8n/n8n-nodes-langchain.*`, `n8n-nodes-base.httpRequest`, `n8n-nodes-base.googleDrive`, `n8n-nodes-base.code`, `n8n-nodes-base.wait`, `n8n-nodes-base.if`, `@n8n/n8n-nodes-langchain.googleGemini`

> Set environment variables such as `OPENAI_API_KEY`, `GEMINI_API_KEY`, and your HeyGen API key in n8n credentials. Use **Header Auth** to attach HeyGen auth to both HTTP Request nodes.

---

## Data Flow & Node Details

### 1) **When clicking ‘Execute workflow’** (Manual Trigger)
Starts a single end‑to‑end run.

### 2) **AI Agent** (HN Orchestrator)
Prompt (condensed):
- Fetch top 10 HN stories in last 24h related to AI/LLMs
- Pick the most likely **to go viral**
- Fetch article + HN comments
- Write a ~20s (spoken ≈30s) script with stats, 6th‑grade reading level, balanced view
- Minimal commas early; end with: `Hit follow to stay ahead in AI!`
- **Output only** the final script text (no notes)

**Tools wired in:**
- **Fetch HN Front Page**: resource `all`, keyword `AI`, tags `front_page`
- **Fetch HN Article**: by `articleId` (provided via `$fromAI(...)` from the agent)
- **OpenAI: Write Script**: model `gpt-4.1`

### 3) **AI Agent2** → Captions JSON
- Input: the final script
- Output: structured JSON `{ captions: [{ id, caption, tag }] }` with each caption ~10–15 words
- Validated by **Structured Output Parser**

### 4) **Split Out**
- Splits `output.captions` into individual items.

### 5) **AI Agent3** → Image Prompt JSON
- Input: each `caption`
- Output: JSON `{ "image Prompt": "…", "id": "…" }` with a clean **9:16** background brief (no text/logos/UI, SFW, single subject, depth, lighting, copy‑space)
- Validated by **Structured Output Parser1**

### 6) **Google Gemini: Generate an image**
- Model: `models/gemini-2.5-flash-image-preview`
- Prompt: `{{ $json.output['image Prompt'] }}`
- Produces binary image; adds `data_id: {{id}}` into JSON for downstream naming.

### 7) **Code (JavaScript)**
- Normalizes the first binary property to `binary.data`
- Builds a filename: `image_<id>.<ext>`

### 8) **Google Drive: Upload file**
- Uploads the generated image to Drive root.
- Returns metadata (e.g., `webViewLink`, `webContentLink`). You may need to **set file to public** to allow HeyGen to fetch it.

### 9) **HTTP Request** → **HeyGen v2** `POST /video/generate`
- Sends JSON body defining:
  - `title`
  - `dimension`: `{ width:1080, height:1920 }` (**vertical 9:16**)
  - `video_inputs[0]` with `character` (avatar), `voice`, `background` (image URL), and `text` overlay
- **Important**
  - Add the **HeyGen auth header** (e.g., API key) via Header Auth.
  - The provided body contains a **static** background URL; consider swapping to the **Drive link from previous node**.

### 10) **HTTP Request1** → **HeyGen v1** `GET /video_status.get`
- Query param `video_id` from `{{$json.data.video_id}}` (returned by generate call)
- Polls status until `data.status == completed`
- The node currently also sets a `video_id` header; this is typically **not required**—focus on proper authentication.

### 11) **If1 + Wait**
- If `completed` → end
- Else **Wait 60s** → re‑check status (loop)

---

## Setup Instructions

1. **Import the workflow** and confirm all nodes are present.
2. **Credentials**
   - **OpenAI**: attach to `Write Script` and Agents
   - **Gemini**: attach to `Generate an image`
   - **Google Drive OAuth2**: attach to `Upload file`
   - **HeyGen Header Auth**: attach to both HTTP Request nodes; include your required auth header (see HeyGen docs)
3. **HeyGen POST Body**
   - Replace placeholders (`avatar_id`, `voice_id`) with valid IDs from your HeyGen account.
   - Replace `background.url` with the **uploaded image URL** from Drive, e.g. `={{ $json.webContentLink }}` or a direct file URL. Ensure the file is **publicly accessible** or signed.
4. **Fix obvious typos**
   - The sample background URL ends with `.../view?usp=sharingr` → change to `.../view?usp=sharing`.
5. **Make Drive file public** (if required by HeyGen)
   - Add a Drive **permission** step if your org enforces private uploads.
6. **Run** via the manual trigger and watch **Executions**.

---

## Customization

- **Hook strength & style**: Tweak the Agent prompt to adjust virality vs. balance.
- **Caption count & length**: Change AI Agent2 rules and/or Split Out behavior.
- **Image aesthetics**: Add negative cues (e.g., “no text, no watermark”)—already included—but you can extend with styles or camera parameters.
- **Model choices**: Swap `gpt-4.1`/`gemini-2.5-flash-image-preview` for alternatives.
- **Polling cadence**: Adjust Wait from 60s to suit HeyGen queue times; optionally add **max retries** & **timeout guards**.
- **Output handling**: After status==completed, add a final GET to retrieve the **video URL** and post it to Slack/Drive/Sheets.

---

## Important Implementation Notes

- **Auth headers**: Ensure the correct header name/value for HeyGen (API‑key or bearer token). The current workflow only sets `accept: application/json`; add your auth.
- **URL accessibility**: HeyGen must be able to **fetch** the background image. Google Drive `view` links may not be direct binaries; consider using `webContentLink` or a CDN URL.
- **Version mix**: This workflow uses **v2** for `video/generate` and **v1** for `video_status.get`. That’s OK if the API supports it; otherwise align versions.
- **Status field**: The IF node checks `{{$json.data.status}} == "completed"`. If your API returns different states (e.g., `queued`, `processing`, `failed`), add branches for **error** and **retry**.
- **Headers in status call**: Passing `video_id` as a header is unusual; relying on the **query parameter** is typically sufficient.

---

## Troubleshooting

**Script is empty or badly formatted**
- Loosen punctuation constraints in the prompt; ensure the AI output is not truncated.

**No captions produced**
- Verify the **Structured Output Parser** schema matches AI Agent2’s JSON, and that `Split Out` targets `output.captions`.

**Gemini image is missing**
- Ensure model credentials are attached and quota is available; check binary output on the item.

**Drive upload succeeded but HeyGen fails to fetch background**
- Make the file public and use a direct link; test the URL in an incognito browser.

**Status never becomes `completed`**
- Add a **max loop count** and a **failure path** that logs the last status and `video_id`.

**HTTP 401/403 on HeyGen**
- Check the header auth is attached to **both** HeyGen HTTP nodes and that the key has privileges.

---

## Security & Cost Considerations

- News and comments are fetched from public HN; avoid processing private URLs.
- LLM + image gen + render can be **costly**; consider rate‑limiting and caching.
- Generated media may include **brand imagery**; review usage rights for distribution.

---

## Quick Start (TL;DR)
1) Attach OpenAI, Gemini, Drive, and HeyGen credentials
2) Replace `avatar_id`, `voice_id`, and **use a valid background URL** (preferably from the uploaded Drive file)
3) Fix the Drive URL typo and make the file accessible
4) Run once from Manual Trigger and monitor the status polling
5) (Optional) Add a final step to fetch the rendered video URL

---

**Happy building viral AI explainer clips!**

