# 🤖 LinkedIn Auto-Post Agent

An **n8n workflow that uses Gemini AI agents to research a topic, write a human-sounding LinkedIn post, generate a matching image, and publish it, but only after you approve it by email.**

It runs on a schedule, or manually for testing, and keeps a log of every post in Google Sheets so the same topic is never posted twice.

![n8n](https://img.shields.io/badge/n8n-self--hosted-EA4B71)
![Gemini](https://img.shields.io/badge/AI-Google%20Gemini-4285F4)
![Cloudflare](https://img.shields.io/badge/Images-Cloudflare%20Workers%20AI-F38020)
![LinkedIn](https://img.shields.io/badge/Publish-LinkedIn%20API-0A66C2)

---

## 🔄 Workflow

```
Schedule Trigger / Manual Trigger
            ↓
Read previous posts (Google Sheets)
            ↓
Topic Agent (Gemini) → picks topic + target audience
            ↓
Duplicate check → regenerate if the topic was already posted
            ↓
Post Writer Agent (Gemini) → short, natural, human-toned post + hashtags
            ↓
Image Prompt Agent (Gemini) → prompt that matches the post content
            ↓
Image Generation (Cloudflare Workers AI – FLUX)
            ↓
Approval email sent to owner (post + image preview)
            ↓
Wait for approval ──── Rejected ──→ Stop / regenerate
            ↓ Approved
Publish to LinkedIn (personal profile)
            ↓
Log the post in Google Sheets
            ↓
Wait for next scheduled run
```

---

## 🧠 The Three Agents

| Agent | Model | Job |
|-------|-------|-----|
| **Topic Agent** | Gemini | Chooses a fresh topic and target audience, using the owner's goals, interests and past posts so nothing repeats |
| **Post Writer Agent** | Gemini | Writes a short LinkedIn post in a friendly, conversational tone, with relevant hashtags and a personal point of view |
| **Image Agent** | Gemini → Cloudflare FLUX | Turns the post into an image prompt, then generates a realistic image that fits the post instead of a generic AI look |

All agents share the owner's personal context (background, goals, opinions), so the posts sound like a real person and not a template.

---

## 🛠️ Built With

* **n8n** (self-hosted with Docker) – workflow automation
* **Google Gemini** – topic, writing, and image-prompt agents
* **Cloudflare Workers AI (FLUX)** – free image generation
* **Google Sheets** – post history and duplicate prevention
* **Gmail / SMTP** – approval email
* **LinkedIn API** – publishing

---

## ✨ Main Features

* 🤖 Fully AI-generated posts: topic, text, hashtags and image
* 🧬 Writes in your voice, using your own context and point of view
* 🚫 No repeated posts, thanks to the Google Sheets history check
* ✅ **Human-in-the-loop**: nothing goes live without your email approval
* 🖼️ Image prompts that match the post content
* 📅 Scheduled posting plus a manual trigger for testing
* 📝 Every post is logged automatically
* 🔄 Retry on temporary failures

---

## 📋 Prerequisites

* A running **n8n** instance (Docker or VPS)
* A **Google Gemini API key** ([Google AI Studio](https://aistudio.google.com/))
* A **Cloudflare account** with a *Workers AI* API token
* A **LinkedIn Developer app** with the *Share on LinkedIn* / *Sign In with LinkedIn* products enabled
* A **Google account** for Sheets and email (OAuth credentials in n8n)

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

### 7. Test, then schedule
Run the **Manual Trigger** first. Once the email approval and the LinkedIn post work end to end, activate the workflow with the **Schedule Trigger**.

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

## 🗺️ Roadmap

* [ ] Support multiple post formats (story, tip, carousel idea)
* [ ] Analytics: log likes and comments back to the sheet
* [ ] Telegram / WhatsApp approval as an alternative to email
* [ ] Topic queue driven by a content calendar

---

## 📌 Project Purpose

This project explores how **AI agents, workflow automation, and social media APIs** can be combined to simplify content creation while keeping a human in control of what gets published.

---

## 📚 Documentation

For full installation, LinkedIn Developer setup, API credentials, and usage instructions:

👉 **[Visit the Project Documentation](Documentation.pdf)**

---

## 🤝 Contributing

Suggestions and improvements are welcome. Open an issue or submit a pull request.

## 📄 License

This project is released under the [MIT License](LICENSE).

---

## 👨‍💻 Author

**Abhiram Dashewar**

[LinkedIn](https://www.linkedin.com/in/abhiramdashewar)
