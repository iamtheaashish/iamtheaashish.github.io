---
title: "Auto Uploading Files to Google Drive with Bash Script"
date: 2025-07-01 00:00:00 +0530
tags: [bash, linux, systemd, rclone, personal-project]
---

Sometimes, the best way to learn something is to follow a random idea and see where it leads.

I’ve been learning Bash scripting lately, and while practicing, I had this thought:  
> *“Wouldn't it be cool if a text file on my desktop could automatically upload to Google Drive every time I update it?”*

It wasn't a "big" idea. I wasn't even sure if it would be useful for others. But I had a file named `Prompt.txt` that I use to store helpful prompts from ChatGPT, and backing it up automatically felt like a good personal challenge — and a great excuse to dig deeper into the Linux ecosystem.

---

## 🔧 What I Wanted to Build

- A script that watches `Prompt.txt` for changes  
- Whenever it's modified, it should upload the file to Google Drive  
- The process should run in the background and persist across reboots  
- The setup should be clean and not depend on a desktop GUI

---

## 💡 Tools I Discovered

This was a great opportunity to explore a few powerful command-line tools:

- **`rclone`** — to upload files to Google Drive  
- **`inotifywait`** (from `inotify-tools`) — to detect when the file changes  
- **`systemd`** (as a user-level service) — to ensure the script runs at startup  

I didn’t know about these tools beforehand. While the idea and the script were mine, I used ChatGPT to help me discover and configure the tools. It was less about copying and more about understanding how they worked.

---

## 🖥️ My Environment

- **Host OS**: Debian 12 (not a VM — I wanted this to run on my actual system)  
- **File to monitor**: `~/Desktop/Prompt.txt`  
- **Script location**: `~/.local/bin/auto_upload.sh`  
- **Systemd unit file**: `~/.config/systemd/user/auto_upload.service`  

---

## 📜 The Systemd Service

To make the script run automatically, I created a user-level systemd service:

<pre lang="markdown"> ```[Unit]
Description=Auto upload Prompt.txt to Google Drive 
After=network.target 
[Service]
Type=simple 
ExecStart=/home/aashish/.local/bin/auto_upload.sh 
Restart=on-failure 
[Install]
WantedBy=default.target ``` </pre>

I enabled it with:
systemctl --user daemon-reexec
systemctl --user daemon-reload
systemctl --user enable --now auto_upload.service

## 🧪 Testing and Mistakes

Naturally, I ran into a few mistakes — at one point, I mistakenly typed the filename as `Prompts.txt` instead of `Prompt.txt`, and the script silently failed. Small hiccups like that helped me understand why precision matters in scripting.

Eventually, I edited the correct file, saved it, and — it worked! The updated file showed up in the specific Google Drive folder I had configured.

---

## 🧠 What I Learned

- How to monitor file changes with `inotifywait`  
- How to upload files to Google Drive using `rclone`  
- How to create and manage user-level `systemd` services  
- That curiosity-driven learning can lead to real-world usable scripts  

I didn’t build this to show off or solve a major problem — I built it because I was curious. It started with an itch to automate something personal, and in the process, I learned about useful Linux tools that are widely used in real sysadmin and DevOps workflows.
