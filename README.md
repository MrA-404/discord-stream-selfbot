# StreamBot: Discord Video Streaming Selfbot
**For Any Help Join My Server:** [https://discord.gg/BqNPP76Hvm](https://discord.gg/BqNPP76Hvm)

[![License: MIT](https://img.shields.io/badge/License-MIT-yellow.svg)](https://opensource.org/licenses/MIT)
[![Bun](https://img.shields.io/badge/Runtime-Bun-%23000000)](https://bun.sh)
[![FFmpeg](https://img.shields.io/badge/FFmpeg-Required-green)](https://ffmpeg.org)

StreamBot is a Discord selfbot built with Node.js and Bun runtime that allows you to stream videos, YouTube links, Twitch streams, and more directly into Discord voice channels. It uses tools like `yt-dlp` for downloading videos, `ffmpeg` for processing and streaming, and supports custom configurations for resolution, bitrate, etc. The bot can loop videos, handle live streams, and even run a simple web server for managing video files.

**Important Warning:**  
Using selfbots (bots that control your own Discord account) is against Discord's Terms of Service. This can result in your account being banned. Use at your own risk for educational or personal purposes only. Do not use this on public servers or for malicious activities.

## Features
- Stream local video files from a directory.
- Stream YouTube videos/livestreams (auto-downloads or streams directly if live).
- Stream Twitch VODs or live streams.
- Search YouTube videos and play them.
- Loop playback of videos.
- Custom stream settings (resolution, FPS, bitrate, hardware acceleration).
- Automatic screenshot previews for videos using FFmpeg.
- Optional web server for uploading/downloading/managing videos via a browser (with authentication).
- Logging with Winston for debugging.
- Auto-download and update `yt-dlp` binary.
- Supports muting audio during streams (e.g., for silent playback).

## Requirements
- **Runtime:** Bun (a fast JavaScript runtime). Install from [bun.sh](https://bun.sh).
- **Node.js Modules:** Installed via `bun install` (see package.json).
- **External Tools:**
  - FFmpeg: Must be installed on your system and available in PATH (for video processing and screenshots).
  - yt-dlp: Automatically downloaded by the script.
- **Discord Account:** A valid Discord token from your own account (not a bot token).
- **Operating System:** Tested on Linux, macOS, Windows. Some features (like hardware acceleration) may vary by OS.

## Installation
1. **Clone the Repository:**
   ```
   git clone https://github.com/yourusername/streambot.git
   cd streambot
   ```

2. **Install Dependencies:**
   Use Bun to install packages:
   ```
   bun install
   ```

3. **Install FFmpeg:**
   - **Ubuntu/Debian:** `sudo apt install ffmpeg`
   - **macOS:** `brew install ffmpeg`
   - **Windows:** Download from [ffmpeg.org](https://ffmpeg.org/download.html) and add to PATH.
   - **Docker:** If using Docker Compose (see below), FFmpeg is included.

4. **Set Up Environment Variables:**
   Create a `.env` file in the root directory (based on `config.ts`):
   ```
   TOKEN=your_discord_token_here
   PREFIX=!  # Command prefix, e.g., !play
   GUILD_ID=your_guild_id
   COMMAND_CHANNEL_ID=your_command_channel_id
   VIDEO_CHANNEL_ID=your_voice_channel_id

   # Stream Settings (optional, defaults provided)
   STREAM_RESPECT_VIDEO_PARAMS=false  # If true, auto-detect video params like resolution/FPS
   STREAM_WIDTH=1280
   STREAM_HEIGHT=720
   STREAM_FPS=30
   STREAM_BITRATE_KBPS=1000
   STREAM_MAX_BITRATE_KBPS=2500
   STREAM_HARDWARE_ACCELERATION=false  # Enable if your hardware supports it (e.g., GPU decoding)
   STREAM_H26X_PRESET=ultrafast  # Options: ultrafast, superfast, veryfast, etc.
   STREAM_VIDEO_CODEC=H264  # Options: VP8, H264, H265, VP9, AV1

   # Video Directories (optional)
   VIDEOS_DIR=./videos
   PREVIEW_CACHE_DIR=./tmp/preview-cache

   # Web Server (optional, enable if you want a UI for managing videos)
   SERVER_ENABLED=false
   SERVER_USERNAME=admin
   SERVER_PASSWORD=admin  # Change this! It will be hashed automatically.
   SERVER_PORT=8080
   ```
   - **How to Get Your Discord Token:**
     - Log in to Discord in a browser.
     - Open Developer Tools (F12) > Network tab.
     - Refresh the page and look for a request to `/api/v9/users/@me`.
     - In the request headers, find `Authorization` – that's your token.
     - **Never share your token!** It's like your password.
   - **Guild/Channel IDs:**
     - Right-click on the server (guild) or channel in Discord and select "Copy ID" (enable Developer Mode in Settings > Advanced).

5. **Prepare Video Directory:**
   - Place your video files (e.g., MP4) in the `VIDEOS_DIR` (default: `./videos`).
   - The bot will automatically list them on startup.

## Configuration Details
- **config.ts:** Loads settings from `.env`. Edit defaults here if needed.
- **Auto FFmpeg Usage:**
  - FFmpeg is used automatically for:
    - Generating video previews (screenshots at 10%, 30%, 50%, 70%, 90% timestamps).
    - Probing video metadata (resolution, bitrate, FPS) if `STREAM_RESPECT_VIDEO_PARAMS=true`.
    - Streaming videos to Discord (prepares the stream with custom options like resolution, codec).
  - No manual FFmpeg commands needed – the bot handles it via `fluent-ffmpeg` library.
  - If a video is live (e.g., YouTube live), it streams directly without downloading. For non-live, it downloads to a temp file using yt-dlp and streams it.
- **yt-dlp:** Auto-downloads the binary on first run and checks for updates. Used for YouTube/Twitch downloads.

## Running the Bot
1. **Start the Bot:**
   ```
   bun run index.ts
   ```
   - The bot will log in with your token and show available videos.
   - It joins the specified voice channel when commanded.

2. **Using Docker (Optional):**
   - For easy setup with FFmpeg included.
   - Edit `docker-compose.yml` or `docker-compose-warp.yml` with your env vars.
   - Run: `docker-compose up -d`
   - Or with Warp: `docker-compose -f docker-compose-warp.yml up -d`

3. **Commands (Send in the Command Channel):**
   - Prefix: Set in `.env` (default: `!`).
   - `!play <video_name> [mute]`: Streams a local video file. Add `mute` to stream without audio.
     - Example: `!play myvideo.mp4 mute` (loops 5 times by default).
     - Uses your own Discord ID to stream (selfbot streams as you).
   - `!playlink <url>`: Streams from a URL (YouTube, Twitch, direct link).
     - Example: `!playlink https://youtube.com/watch?v=video_id`
     - For live streams, it fetches the m3u8 URL automatically.
   - `!ytplay <title or number>`: Searches YouTube and plays.
     - First search: `!ytplay funny cat video` (shows top 5 results).
     - Then play: `!ytplay 1` (plays the first result).
   - `!ytlive <title>`: Searches and streams a YouTube live video.
   - `!stop`: Stops the stream and leaves the voice channel.
   - `!list`: Lists available local videos.
   - `!refresh`: Refreshes the video list.
   - `!info`: Shows current stream info.
   - `!preview <video_name>`: Generates and shows video previews (screenshots via FFmpeg).

4. **Streaming from Your Own ID:**
   - Since it's a selfbot, it uses your token, so streams appear as if **you** are sharing your screen/video.
   - Join the voice channel manually first (or the bot auto-joins on command).
   - No bot account needed – it's your own profile streaming.

5. **Web Server (If Enabled):**
   - Set `SERVER_ENABLED=true` in `.env`.
   - Access at `http://localhost:8080` (or your IP:port).
   - Login with username/password from `.env`.
   - Features: Upload videos, download, delete, preview (with FFmpeg-generated screenshots), dark mode toggle.
   - Useful for managing videos remotely.

## How It Works Internally
- **Streaming Process:**
  1. Command received → Joins voice channel.
  2. If local video: Uses FFmpeg to prepare stream with custom opts.
  3. If URL: Uses yt-dlp to download (non-live) or get stream URL (live), then FFmpeg processes it.
  4. Streams to Discord using `@dank074/discord-video-stream`.
  5. Auto-cleans up temp files after streaming.
- **Auto FFmpeg:**
  - In `ffmpeg.ts`: Handles screenshots and video params probing.
  - In `index.ts`: Prepares stream with `prepareStream` and plays with `playStream`.
  - No user intervention – triggered on commands like `!play` or previews.
- **Logging:** All actions logged to console (via `logger.ts`).

## Troubleshooting
- **Errors with yt-dlp/FFmpeg:** Ensure they're in PATH. Check logs for details.
- **Stream Not Starting:** Verify guild/channel IDs, token validity. Ensure you're in the voice channel.
- **Rate Limits:** Selfbots can hit Discord limits – use sparingly.
- **Dependencies Issues:** Run `bun install` again.
- **Windows-Specific:** Some paths/architectures handled in `yt-dlp.ts`.

## Contributing
- Fork the repo, make changes, submit a PR.
- Report issues on GitHub.


## Credits
- Built with: discord.js-selfbot-v13, fluent-ffmpeg, yt-dlp, play-dl, etc.
- Inspired by Discord streaming tools.

If you have questions, open an issue! 🚀
