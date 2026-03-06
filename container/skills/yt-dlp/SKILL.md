---
name: yt-dlp
description: Download videos from YouTube, Instagram, TikTok, Twitter/X, and 1000+ other sites and send them as MP4 files. Use whenever a user sends a video link and wants it downloaded or converted.
allowed-tools: Bash(yt-dlp *), Write
---

# Video Download with yt-dlp

Download a video from any supported site and send it back as an MP4.

## Full workflow

```bash
# 1. Download to /workspace/group/ (NOT /tmp/ — the host cannot access /tmp/)
yt-dlp -f "bestvideo[ext=mp4]+bestaudio[ext=m4a]/best[ext=mp4]/best" \
  --merge-output-format mp4 \
  --no-playlist \
  --restrict-filenames \
  -o "/workspace/group/video.%(ext)s" \
  "<URL>"

# 2. Send via IPC — use Python to write valid JSON (avoids shell escaping issues)
python3 -c "
import json, os, time, random, string
data = {
    'type': 'media_message',
    'chatJid': os.environ['NANOCLAW_CHAT_JID'],
    'filePath': 'video.mp4',
    'caption': 'Here is your video!',
    'mimeType': 'video/mp4',
}
fname = f'/workspace/ipc/messages/{time.time_ns()}.json'
with open(fname, 'w') as f:
    json.dump(data, f)
print('IPC written:', fname)
"

# 3. Clean up after sending
rm -f /workspace/group/video.mp4
```

**Key rules:**
- Always save to `/workspace/group/`, never `/tmp/` — the host can't reach `/tmp/`
- Always use Python to write the IPC JSON — shell heredocs break with special characters
- `filePath` is just the filename (e.g. `video.mp4`), not the full path

## Common flags

```bash
yt-dlp --no-playlist          # Single video only
yt-dlp --max-filesize 50m     # Skip files over 50MB
yt-dlp --restrict-filenames   # Safe filenames (no spaces/special chars)
yt-dlp -x --audio-format mp3  # Audio only (mp3)
yt-dlp --list-formats <URL>   # See all available formats
```
