# 🎬 YouTube Automation with n8n

An end-to-end **n8n workflow** that automatically generates faceless/avatar YouTube videos — from AI script writing to voiceover, image/video generation, audio-video merging, and final upload to YouTube.

Built with **n8n**, **OpenAI**, **ElevenLabs**, **Kling AI**, and **fal.run**.

---

## ⚙️ How It Works

The workflow is split into two parts:

### Part 1 — Content & Media Generation (`YT_Part_01.json`)
1. **Ideator (OpenAI)** – Generates a video concept (title, description, script) based on a user prompt.
2. **Script Splitter** – Breaks the script into ~6-second chunks for scene-by-scene generation.
3. **Image/Video Prompt Generator (OpenAI)** – Creates visual prompts for each chunk.
4. **Image Generation** – Generates avatar/face images per scene.
5. **Audio Voice Generator (ElevenLabs)** – Converts the script into natural voiceover audio.
6. **Google Drive Upload** – Stores generated images/audio and shares the folder for downstream use.

### Part 2 — Assembly & Upload (`YT_Part_02.json`)
1. **Webhook Trigger** – Kicks off Part 2 once Part 1's assets are ready.
2. **Kling AI (image2video)** – Converts generated images into short video clips.
3. **Status Polling (IF node)** – Waits until video generation status is `succeed`.
4. **Google Drive Download** – Fetches the generated voiceover audio.
5. **fal.run (ffmpeg-api)** – Merges the video and audio into one final file.
6. **YouTube Upload** – Publishes the final merged video directly to a YouTube channel.

---

## 🧩 Tech Stack

| Purpose | Service |
|---|---|
| Script/Prompt generation | OpenAI (GPT-4o / o3-mini) |
| Text-to-Speech | ElevenLabs |
| Image → Video | Kling AI |
| Audio/Video Merge | fal.run (ffmpeg-api) |
| Storage | Google Drive |
| Publishing | YouTube Data API |
| Orchestration | [n8n](https://n8n.io) |

---

## 🚀 Setup

1. Import both `YT_Part_01.json` and `YT_Part_02.json` into your n8n instance.
2. Add credentials in n8n for:
   - OpenAI API
   - Google Drive (OAuth2)
   - YouTube (OAuth2)
3. Replace the placeholder API keys in the HTTP Request nodes with your own:
   - `YOUR_KLING_API_KEY` → your Kling AI key
   - `YOUR_FAL_API_KEY` → your fal.run key
   - `YOUR_ELEVENLABS_API_KEY` → your ElevenLabs key

   > 🔒 **Tip:** Instead of pasting keys directly into nodes, use n8n's built-in **Credentials** manager or environment variables so secrets never end up in exported JSON.

4. Update the Google Drive file/folder IDs and the audio file link to point to your own Drive.
5. Activate the Webhook (Part 2) and connect it to run after Part 1 completes.

---

## ⚠️ Notes

- This is a **personal/experimental automation project** — review generated content before publishing, especially titles, descriptions, and thumbnails.
- API usage from OpenAI, ElevenLabs, Kling AI, and fal.run may incur costs — check each provider's pricing before running at scale.
- Never commit real API keys to version control. This repo's JSON exports have all secrets replaced with placeholders.

---

## ❓ FAQ

**Q: Do I need to know how to code to use this?**
A: No. This is a no-code n8n workflow — you just import the JSON files and configure credentials/API keys through the n8n UI.

**Q: Which API keys do I need?**
A: OpenAI, ElevenLabs, Kling AI, fal.run, plus Google Drive and YouTube OAuth2 credentials in n8n.

**Q: Why are there two separate workflow files instead of one?**
A: Part 1 (generation) and Part 2 (assembly + upload) are split so Part 2 can be triggered independently via webhook — useful if you want to queue multiple videos or run the merge/upload step on a schedule.

**Q: The Kling AI / fal.run URLs have hardcoded task or request IDs — do I need to change those?**
A: Yes. URLs like the `image2video/<id>` endpoint or `response_url` are specific to a single generation job. In production you'd wire these dynamically from the previous node's output rather than hardcoding them — check and update before running your own jobs.

**Q: Is this free to run?**
A: No. OpenAI, ElevenLabs, Kling AI, and fal.run all charge per use. Check each provider's pricing page before generating videos at scale.

**Q: Can I use a different TTS/image/video provider?**
A: Yes — swap out the relevant HTTP Request node for another provider's API. The workflow structure (script → chunks → media → merge → upload) stays the same.

**Q: I got an error / the workflow doesn't trigger — what should I check first?**
A: Most common issues are (1) missing or expired credentials, (2) leftover placeholder API keys not replaced with your real ones, or (3) Google Drive file/folder IDs still pointing to the original owner's Drive instead of yours.

**Q: Is my API key safe if I fork this repo?**
A: Only if you don't commit it. Use n8n's Credentials manager or environment variables — never hardcode keys directly into node parameters before exporting/pushing your workflow.

---

## 📄 License

MIT — feel free to fork and adapt for your own content pipeline.
