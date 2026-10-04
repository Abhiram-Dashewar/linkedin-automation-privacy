# 🤖 LinkedIn Auto-Post Agent

An n8n workflow that turns a short **"What I learned today?"** note into a ready-to-publish LinkedIn post with an AI-generated image. You fill in a form, the workflow writes the post and image, emails you a preview, and publishes to LinkedIn only after you approve. **One important remainder that, we are using google gemini's free API. Sometimes this automation sends too many requests from our side. It is better to wait until each node executes perfectly.**

**Workflow name:** `MyFinalAutomation`
**Trigger:** n8n Form (manual, whenever you want to post)
**Human in the loop:** Yes, nothing is published without your approval by email

![n8n](https://img.shields.io/badge/n8n-self--hosted-EA4B71)
![Gemini](https://img.shields.io/badge/AI-Google%20Gemini-4285F4)
![Cloudflare](https://img.shields.io/badge/Images-Cloudflare%20Workers%20AI-F38020)
![LinkedIn](https://img.shields.io/badge/Publish-LinkedIn%20API-0A66C2)


## What it does ?

1. You submit a form with your topic and what you learned.
2. Gemini writes a LinkedIn post in your own voice, using only the facts you gave it.
3. A second Gemini agent turns the post into a photo-style image prompt.
4. Cloudflare Workers AI (FLUX.1 schnell) generates the image.
5. You get two emails: an image preview, then an approval email with **Post it** / **Skip** buttons.
6. If you approve, the post and image go to LinkedIn and the details are logged in Google Sheets.


## Workflow diagram

```
Form Trigger
    |
Config  (your profile, goals, image style, IDs)
    |
Duplicate Check  (validates + cleans form input)
    |
Wait-1 (30)
    |
Post Writer Agent  <--- Gemini - Writer
    |
Wait-2 (40)
    |
Image Prompt Agent <--- Gemini - Image Prompt
    |
Wait-3 (50)
    |
Cloudflare Image (FLUX.1 schnell)
    |
To Binary (base64 -> image file)
    |
Email Preview (image attached)
    |
Email Approval (Post it / Skip)
    |
 Approved?  --- no ---> end (nothing posted)
    |
   yes
    |
Restore Image (re-attach image data)
    |
Post to LinkedIn
    |
Save to Google Sheet
```

## Workflow structure

![Alt Text](/Workflow.png)

## Node-by-node

| # | Node | Type | Purpose |
|---|------|------|---------|
| 1 | **Form Trigger1** | Form Trigger | Form titled "What did you learn today?" with 3 fields: **Topic** (required), **What I learned** (required, 3-6 lines), **Who is this for?** (dropdown: Students and beginners / Aspiring cybersecurity learners / Recruiters and professionals). |
| 2 | **Config** | Set | Central place for your settings: `sheetId`, `niche`, `myProfile`, `myGoals`, `imageStyle`, `cloudflareAccountId`. |
| 3 | **Duplicate Check** | Code | Reads the form, trims the text, throws an error if Topic or What I learned is empty, and outputs `topic`, `audience`, `angle`. (Despite the name, it currently validates input only; it does not compare against past posts in the sheet.) |
| 4 | **Wait-1 / Wait-2 / Wait-3** | Wait | Short pauses (30, 40 and 50) between AI calls to stay under API rate limits. |
| 5 | **Post Writer Agent** + **Gemini - Writer** | AI Agent + Google Gemini (temp 0.7) | Writes the LinkedIn post in first person. See the writing rules below. |
| 6 | **Image Prompt Agent** + **Gemini - Image Prompt** | AI Agent + Google Gemini (temp 0.9) | Builds a 45-70 word photo prompt for FLUX, with a random camera framing each run (top-down flat lay, close-up at desk level, over-the-shoulder, or wide corner shot). |
| 7 | **Cloudflare Image1** | HTTP Request | POSTs the prompt to Cloudflare Workers AI `@cf/black-forest-labs/flux-1-schnell` (6 steps, 120 s timeout, header-auth credential). |
| 8 | **To Binary** | Code | Converts the base64 image in Cloudflare's response into a real `post-image.jpg` binary file. |
| 9 | **Email Preview1** | Gmail | Sends you the image as an attachment, plus the post text. |
| 10 | **Email Approval1** | Gmail (Send and Wait) | Sends the post text with **Post it** and **Skip** buttons and pauses the workflow until you reply (with a limited wait time of 12). |
| 11 | **Approved?1** | IF | Continues only if `approved === true`. The Skip path ends the workflow. |
| 12 | **Restore Image1** | Code | Re-attaches the image binary from `To Binary` (it gets lost after the approval step). |
| 13 | **Post to LinkedIn** | LinkedIn | Publishes the post text with the image (`shareMediaCategory: IMAGE`). |
| 14 | **Save to Google Sheet** | Google Sheets | Appends a row to the `Posts` sheet: Date, Topic, Audience, Post, ImagePrompt, LinkedInPostId. |

---
This is the clear description about how each node works. 


## How the post is written

The Post Writer Agent is told to:

- Write in first person, like texting a smart friend, with simple words, short sentences and contractions.
- Use **only** facts from your profile and your notes. It must not invent stories, results, numbers or news.
- Explain at beginner level and present it as something you *learned*, not expert advice.
- Avoid AI-sounding phrases ("delve", "unlock", "game-changer", "let's dive in", etc.), em dashes and hype.
- Write 90-160 words, end with one genuine question, and add exactly 3 hashtags.
- Output plain text only (no markdown, no bold) and never reveal private details like health, family, finances, exact location or contact info.

## How the image is made

The Image Prompt Agent describes a realistic student-workspace photo (laptop terminal, breadboard, router, tea mug, notebook, etc.) with these rules: no readable text or logos, no faces, no cliché hacker or padlock or matrix imagery, and a style taken from `imageStyle` in the Config node.

---

## 🚀 Setup

[⬇️ Download n8n Automation. ](workflow.json)

### 1. Import the workflow
In n8n: **Workflows → Import from File** and select `workflow.json`. 

### 2. Create the Google Sheet
Create a sheet with these columns in row 1:

| Date | Topic | Audience | Post | ImagePrompt | LinkedInPostId |
|------|-------|----------|------|-------------|----------------|

### 3. Add credentials in n8n

| Credential | Used by |
|------------|---------|
| Google Gemini (PaLM) API | Topic, Writer and Image Prompt agents |
| Google Sheets OAuth2 | Reading and logging posts |
| Cloudflare Workers AI (HTTP header auth) | Image generation |
| Gmail OAuth2 / SMTP | Approval email |
| LinkedIn OAuth2 | Publishing |

### 4. Set the Cloudflare image request
* **Method:** `POST`
* **URL:** `https://api.cloudflare.com/client/v4/accounts/<ACCOUNT_ID>/ai/run/@cf/black-forest-labs/flux-1-schnell`
* **Header:** `Authorization: Bearer <WORKERS_AI_TOKEN>`
* **Body:** `{ "prompt": "<image prompt>" }`

### 5. Configure your context
Paste your background, goals and tone preferences into the agents' system prompts so the posts reflect you.

### 6. Set the approval email
Update the email node with the address where approval requests should be sent.

### 7. Test
Manually fill the form, remaining nodes will be executed. **This is a semi-automated project**.


## Typical run

1. Submit the form (Topic: *What did you learned today?*, plus 3-6 lines of notes).
2. Wait about 2-3 minutes while the post and image are generated.
3. Check email 1 (**Image preview**) and email 2 (**Approve today's LinkedIn post**).
4. Tap **Post it** to publish or **Skip** to cancel.
5. On approval, the post goes live and a row is added to your sheet.
6. **Note**: Sometimes our workflow sends too many requests. We are using free account, so it is better to change model and re-execute the node.


---

## Notes and limitations

- **No duplicate detection yet.** The `Duplicate Check` node only validates the form. To really prevent repeats, add a Google Sheets "Get rows" step that checks the Topic column before writing.
- **Skip does nothing.** If you press Skip (or the approval times out), the workflow ends and nothing is posted or logged.
- **Cost and limits.** Gemini and Cloudflare free tiers have rate limits, which is why the Wait nodes are there.
- **Image quality.** FLUX schnell is fast but can produce odd details; always review the preview image before approving.
- **Privacy.** Your profile text is sent to Gemini on every run. Keep sensitive personal details out of `myProfile`.

---

## 📋 Prerequisites

* A running **n8n** instance (Docker or VPS)
* A **Google Gemini API key** ([Google AI Studio](https://aistudio.google.com/))
* A **Cloudflare account** with a *Workers AI* API token
* A **LinkedIn Developer app** with the *Share on LinkedIn* / *Sign In with LinkedIn* products enabled
* A **Google account** for Sheets and email (OAuth credentials in n8n)


---

## 🧯 Troubleshooting

| Problem | Likely cause | Fix |
|---------|--------------|-----|
| `401` from Cloudflare | Wrong token type (e.g. an R2 token) | Create a token with **Workers AI** permission |
| `429` / quota `limit: 0` from Gemini image API | Free tier has no image generation quota | Use Cloudflare FLUX for images (as in this workflow) |
| `402 Payment Required` from an image API | The service moved behind a paywall | Switch to a free provider such as Cloudflare Workers AI |
| Same topic posted again | Sheet not read or columns misnamed | Check the Google Sheets node and the column names |
| Post sounds robotic | Writer prompt is too generic | Add tone rules and your own context to the prompt |
| LinkedIn token expired | OAuth tokens expire | Re-authenticate the LinkedIn credential in n8n |

---

## 🛠️ Tech Stack

* **n8n** (self-hosted with Docker) – workflow automation
* **Google Gemini** – topic, writing, and image-prompt agents
* **Cloudflare Workers AI (FLUX)** – free image generation
* **Google Sheets** – post history and duplicate prevention
* **Gmail / SMTP** – approval email
* **LinkedIn API** – publishing


---

## 📌 Project Purpose

This project explores how **AI agents, workflow automation, and social media APIs** can be combined to simplify content creation while keeping a human in control of what gets published.

---

## 📚 Documentation

For full installation, LinkedIn Developer setup, API credentials, and usage instructions:

👉 **[Visit the Project Documentation]()**

---

## 🤝 Contributing

Suggestions and improvements are welcome. I know that this is the basic semi-automated automation. Assume that, it's the first version. For changes or any queries, you always invited. Open an issue or submit a pull request. For direct query mail - **abhidashewar@gmail.com**

## 📄 License

This project is released under the [MIT License](LICENSE).

---
