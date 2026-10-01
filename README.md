# 🤖 LinkedIn Auto-Post Agent

A simple **n8n automation that uses AI to generate and publish LinkedIn posts automatically.**

The workflow can run on a schedule or manually for testing.

## 🔄 Work-flow

```
Schedule / Manual Trigger
          ↓
Get previous post details (Google sheets)
          ↓
Topic agent (Generates topic)
          ↓     
Duplicate check (To avoid repeated posts)
          ↓
Post writer agent (Writes post)
          ↓
Image prompt generation 
          ↓
Image generation (Cloudflare)
          ↓
Mail sent to owner
          ↓
Waiting for approval from mail
          ↓
LinkedIn Post
          ↓
Google sheet updated
          ↓
 Repeat
```

The AI generates a short professional LinkedIn post with relevant hashtags, and the workflow publishes it directly to LinkedIn.


## 🛠️ Built With

* **n8n** - Workflow automation
* **OpenAI** - AI-powered content generation
* **LinkedIn API** - Publishing posts
* **Cron Schedule** - Automated scheduling


## ✨ Main Features

* 🤖 Automatically generates LinkedIn content
* 📅 Scheduled posting
* ▶️ Manual trigger for testing
* #️⃣ Generates relevant hashtags
* 🔗 Publishes directly to LinkedIn
* 🔄 Automatic retry on temporary failures

## 📌 Project Purpose

This project was created to explore how **AI, workflow automation, and social media APIs** can be combined to simplify content creation and publishing.

The workflow is intentionally simple so it can be easily understood, customized, and extended.

## 📚 Documentation

For complete installation, configuration, LinkedIn Developer setup, API credentials, and usage instructions:

👉 **[Visit the Project Documentation](YOUR-WEBSITE-LINK)**

## 👨‍💻 Author

**Abhiram Dashewar**

[LinkedIn](https://www.linkedin.com/in/abhiramdashewar)


---

