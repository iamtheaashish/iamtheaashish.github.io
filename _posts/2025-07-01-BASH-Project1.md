---
title: "Auto Uploading Files to Google Drive with Bash Script"
date: 2025-07-01 00:00:00 +0530
tags: [bash, linux, systemd, rclone, personal-project]
---

For a long time, I’ve been saving helpful prompts from ChatGPT in a text file called `Prompt.txt` on my desktop. I wanted an automated way to back it up to Google Drive whenever it’s updated.

So I challenged myself to **build a solution using only Bash scripting** and available CLI tools.

---

## 🔧 The Problem

I wanted `Prompt.txt` to upload to Google Drive **automatically** whenever I edited it — and **without needing to remember to do it manually**.

---

## 💡 The Solution

I built a Bash script that:
- Watches `Prompt.txt` for changes using `inotifywait`
- Uploads it to Google Drive using `rclone`
- Runs in the background on boot using a systemd user service

---

## 📁 Tools Used

- `rclone` (not installed by default — needed setup)
- `inotify-tools` for real-time file monitoring
- systemd user service (created my own unit)

---

## ⚙️ How I Did It

I wrote this script:  
```bash
# ~/.local/bin/auto_upload.sh
#!/bin/bash

PROMPT_FILE="$HOME/Desktop/Prompt.txt"
GDRIVE_FOLDER="aashish_drive:BashUploads"
TEMP_DIR="$HOME/.local/tmp_uploaded_files"

mkdir -p "$TEMP_DIR"

# Wait until file exists
while [ ! -f "$PROMPT_FILE" ]; do
    echo "Waiting for $PROMPT_FILE to exist..."
    sleep 5
done

inotifywait -m -e close_write --format '%w%f' "$PROMPT_FILE" | while read file; do
    TIMESTAMP=$(date +"%H-%M_%Y-%m-%d")
    BASENAME=$(basename "$file" .txt)
    NEWFILE="${BASENAME}_${TIMESTAMP}.txt"
    TEMP_PATH="$TEMP_DIR/$NEWFILE"

    cp "$file" "$TEMP_PATH"
    rclone copy "$TEMP_PATH" "$GDRIVE_FOLDER"
    echo "[$(date +"%H:%M:%S")] Uploaded: $NEWFILE"
done

...

Then created a systemd unit at:
```bash
# ~/.config/systemd/user/auto_upload.service
[Unit]
Description=Auto upload Prompts.txt to Google Drive
After=network.target

[Service]
Type=simple
ExecStart=/home/aashish/.local/bin/auto_upload.sh
Restart=on-failure

[Install]
WantedBy=default.target

...