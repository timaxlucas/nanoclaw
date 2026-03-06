---
name: yt-dlp
description: Download videos from YouTube, Instagram, TikTok, Twitter/X, and 1000+ other sites and send them as MP4 files. Use whenever a user sends a video link and wants it downloaded or converted.
allowed-tools: Bash(yt-dlp *), Write
---

# Video Download with yt-dlp

Download a video from any supported site and send it back as an MP4 via WhatsApp.

## Full workflow

```bash
# 1. Download as mp4 to the group workspace
yt-dlp -f "bestvideo[ext=mp4]+bestaudio[ext=m4a]/best[ext=mp4]/best" \
  --merge-output-format mp4 \
  -o "/workspace/group/%(title)s.%(ext)s" \
  "<URL>"

# 2. Find the downloaded file
ls /workspace/group/*.mp4

# 3. Send it via IPC
```

Then write an IPC media message to send the file:

```json
// Write to: /workspace/ipc/messages/<timestamp>.json
{
  "type": "media_message",
  "chatJid": "<CHAT_JID>",
  "filePath": "video_title.mp4",
  "caption": "Here's your video!",
  "mimeType": "video/mp4"
}
```

The `filePath` is relative to the group folder — just the filename, no `/workspace/group/` prefix.

## IPC write example

```bash
cat > /workspace/ipc/messages/$(date +%s%N).json << 'EOF'
{
  "type": "media_message",
  "chatJid": "CHAT_JID_HERE",
  "filePath": "video.mp4",
  "caption": "Here's your video!",
  "mimeType": "video/mp4"
}
EOF
```

## Tips

- Always use `-f "bestvideo[ext=mp4]+bestaudio[ext=m4a]/best[ext=mp4]/best"` to get a proper mp4
- Use `--no-playlist` if the URL might be a playlist but the user only wants one video
- Use `--max-filesize 50m` to avoid very large files that WhatsApp rejects
- Clean up the file after sending: `rm /workspace/group/video.mp4`
- If the title has special characters, use `-o "/workspace/group/video.%(ext)s"` for a safe name

## Common flags

```bash
yt-dlp --no-playlist          # Single video only
yt-dlp --max-filesize 50m     # Skip files over 50MB
yt-dlp --restrict-filenames   # Safe filenames (no spaces/special chars)
yt-dlp -x --audio-format mp3  # Audio only (mp3)
yt-dlp --list-formats <URL>   # See all available formats
```
