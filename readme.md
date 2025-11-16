# jellyfin redirect
the world's shittiest way to run a server without sacrificing your soul to the whois gods <br>
want an explination? here take this vibecoded chatgpt summary because it did all the work for me (kill me, i know nothing about bash scripts and can't be bothered to learn)
# Jellyfin Cloudflare Tunnel Auto-Redirect System

This project provides a fully automated, self-updating Cloudflare Tunnel for Jellyfin, designed to run on any Linux server. It creates a TryCloudflare tunnel, extracts the tunnel URL, updates your redirect repository on GitHub, and publishes a redirecting index.html so you always have a stable entrypoint to your Jellyfin instance without needing a domain name.

Everything is handled by:

A robust Bash launcher script

A hardened systemd service

Git-based publishing of the current tunnel URL

# 🚀 Features
✔ Automatic TryCloudflare tunnel

Cloudflared creates a temporary URL like:

https://color-malpractice-naval-operated.trycloudflare.com


The script automatically detects the generated URL from Cloudflared’s log.

# ✔ Auto-publishes redirect page

Whenever the URL changes, the script updates your GitHub repository:

index.html → <meta http-equiv="refresh" content="0; url=<new tunnel>">


This gives you a permanent public entrypoint:

https://<your-github-redirect>.github.io/

# ✔ Duplicate cloudflared detection

The script force-kills any existing cloudflared processes to prevent:

Multiple tunnels from starting

Cloudflare 1033 errors

Stale orphaned processes breaking systemd restarts

# ✔ Built for systemd

The unit uses:

KillMode=process

Restart logic

Correct environment for non-root users

Clean cgroup control

# ✔ Crash-safe

If cloudflared crashes, the entire service restarts and generates a new usable tunnel.

# 📂 File Overview
/home/jellyfin/jellyfin-tunnel.sh

The main script. Responsible for:

Killing stray cloudflared processes

Launching a new tunnel

Extracting the generated URL

Generating index.html.new

Publishing changes to GitHub

Logging all actions

Keeping cloudflared alive with wait

/etc/systemd/system/jellyfin-tunnel.service

The systemd unit that manages the script:

Runs as the jellyfin user

Automatically restarts on failure

Ensures cloudflared is killed properly

Provides clean logs in journalctl

# 🔄 How the Tunnel Starts (Flow Diagram)
systemd → jellyfin-tunnel.sh → pkill cloudflared
                           ↓
                     start cloudflared (background)
                           ↓
                 cloudflared writes logs → /tmp/cloudflared-output.log
                           ↓
               script scans log → finds https://xxxx.trycloudflare.com
                           ↓
               script compares URL with last known URL
                           ↓
              changed? → update GitHub → new redirect URL
               same?   → skip update
                           ↓
                   script waits on cloudflared PID

# 🛠 Requirements

You need:

Jellyfin running on localhost (default: http://localhost:8096)

Cloudflared installed (cloudflared tunnel …)

A GitHub repo (jellyfin-redirect/) where index.html lives

SSH or PAT authentication for Git pushes

Systemd-based Linux environment

# 📦 Installation

You used a single setup command that:

Installed a bulletproof jellyfin-tunnel.sh

Installed the corrected systemd unit

Reloaded systemd

Restarted the service cleanly

Ensured no duplicate cloudflared processes exist

Your system now starts the tunnel automatically on boot.

# 🧪 Testing the Tunnel

Check the tunnel process:

ps aux | grep cloudflared


→ Should show exactly 1 cloudflared process.

Check logs:

cat /tmp/cloudflared-output.log
journalctl -u jellyfin-tunnel.service -f


Check redirect page updated:

curl -I https://<your-github-username>.github.io/<repo>/

# 🩹 Troubleshooting
Cloudflare Error 1033

Cause: Multiple cloudflared processes.
Fix: Script automatically prevents this, but manually run:

pkill -9 cloudflared
systemctl restart jellyfin-tunnel.service

Redirect not updating

Run:

cd /home/jellyfin/jellyfin-redirect
git pull
git status

Tunnel not generating URL

Check:

Jellyfin is listening on localhost:8096

/tmp/cloudflared-output.log exists

Cloudflared version is recent

# 💡 Why TryCloudflare?

TryCloudflare gives:

Free HTTPS access

No domain name required

No DNS setup

No Cloudflare account needed

For a more stable, permanent setup, you can upgrade to:

cloudflared tunnel create <NAME>
cloudflared tunnel route dns <NAME> jellyfin.<your-domain>


But TryCloudflare is perfect for quick, temporary, or hobby servers.

# 🎉 Summary

Your system now:

Generates a Cloudflare tunnel on boot

Extracts the live tunnel URL automatically

Updates your GitHub redirect page

Avoids duplicate processes

Recovers automatically from crashes

Gives you a stable external Jellyfin entrypoint

Completely automated.
Hands-free.
Reliable.
