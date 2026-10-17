
Private SillyTavern via Cloudflare Tunnel
Run your own fully private, unfiltered AI companion on your own PC or home server. No monthly hosting fees. Your data never leaves your machine. No port forwarding. No exposed home IP.

This repository contains a complete Docker Compose stack that runs SillyTavern securely and connects it to the outside world using Cloudflare Tunnel. You can access your companion from your phone, laptop, or tablet from anywhere in the world, protected by Cloudflare's enterprise-grade security.

Table of Contents
How It Works (Architecture)

Prerequisites

Step 1: Prepare the Files

Step 2: Cloudflare Tunnel Setup

Step 3: Launch the Stack

Step 4: Lock Down Your Companion

Troubleshooting

Managed Hosting (24/7 Uptime)

How It Works
Instead of opening a port on your router (which is a massive security risk), this stack uses Cloudflare Tunnels.

The cloudflared container reaches out to Cloudflare and creates a secure, encrypted tunnel.

Cloudflare assigns your domain name (e.g., home.yourdomain.com) to this tunnel.

When you visit the URL, Cloudflare sends the traffic down the tunnel directly to your SillyTavern container.

Result: Your home IP is hidden, your traffic is encrypted with SSL, and you do not need a static IP.

Prerequisites
Before you begin, ensure you have:

Docker Desktop installed and running on your PC (Windows/Mac) or Docker Engine (Linux).

A Domain Name (e.g., yourdomain.com). You can buy a cheap one (like a .xyz or .cfd domain) for roughly $1 to $2 a year.

A free Cloudflare Account.

Step 1: Prepare the Files
Clone or download this repository to your computer.

Open a terminal (Command Prompt, PowerShell, or Terminal) inside the folder.

Copy the example environment file:

bash
cp .env.example .env
Open the .env file in a text editor. You will fill it out in the next step.

Step 2: Cloudflare Tunnel Setup
Log into your Cloudflare Dashboard.

Add your domain to Cloudflare and ensure the nameservers are updated. This can take up to 24 hours, but usually takes about 10 minutes.

On the left sidebar, go to Zero Trust > Networks > Tunnels.

Click Create a tunnel, choose Cloudflared, and name it (e.g., sillytavern-tunnel).

Copy the Token (the long string starting with eyJ...).

Paste the token into your .env file:

env
CLOUDFLARE_TUNNEL_TOKEN=eyJhIjoi...
Click Next, then Add a public hostname:

Subdomain: home (or whatever you want)

Domain: Select your domain.

Service Type: HTTP

URL: host.docker.internal:8000
(Important: Use host.docker.internal instead of localhost or the container name. This allows Docker Desktop on Windows/Mac to route the traffic correctly).

Click Save hostname.

Step 3: Launch the Stack
Make sure Docker Desktop is running in the background.

In your terminal, start the containers:

bash
docker compose up -d
Wait about 30 seconds for the images to pull and the containers to start. Open your browser and go to https://home.yourdomain.com.

Note: You might see a "Forbidden" error on the first load. We fix that in the Troubleshooting section below.

Step 4: Lock Down Your Companion
Right now, anyone with the link can access your AI. Let's use Cloudflare Access to lock it behind your Gmail account.

Go to Zero Trust > Access > Applications.

Click Add an application > Self-hosted.

Name it "SillyTavern".

Set the domain to the one you created in Step 2 (e.g., home.yourdomain.com).

Create an Allow policy.

Action: Allow

Rule: Emails -> Enter your personal email address.

Save.

Now, when you visit the site, Cloudflare will ask for your email. It will send you a 6-digit PIN (One-Time Password). Once you enter it, you get in. No one else can.

Troubleshooting
Error: "Forbidden - add your IP address to the whitelist"
This is the most common issue users face. Because SillyTavern is running inside Docker and sitting behind Cloudflare, it sees the traffic coming from an internal Docker IP address, not the public internet. By default, SillyTavern blocks this.

The Fix:

Open the config/config.yaml file in this folder.

Find the whitelist: section.

Update it to look exactly like this:

yaml
whitelist:
  - ::1
  - 127.0.0.1
  - 172.16.0.0/12
  - 0.0.0.0/0
Save the file and restart the container:

bash
docker compose restart sillytavern
Error: Cloudflare shows "502 Bad Gateway"
This means Cloudflare is connected, but it cannot reach SillyTavern.

Check the URL: Make sure the Cloudflare dashboard is set to http://host.docker.internal:8000.

Check Docker: Run docker ps to make sure the sillytavern container is actually running and has not crashed.

Nothing loads at all / Cloudflare shows "1033"
This means the cloudflared container is not running or your token is wrong.

Check the logs: docker compose logs cloudflared. Look for "Registered tunnel connection".

Important Limitations
Your PC must be turned on. If your PC sleeps, shuts down, or loses internet, your companion goes offline. This is the trade-off of self-hosting.

Server Plugins are disabled. Do not enable enableServerPlugins in the config unless you wrote the plugin yourself. They have full access to your PC.

Managed Hosting (24/7 Uptime)
If you want your companion to always be online, even when your PC is off, I offer a Managed Hosting Service. I run this exact secure setup on an isolated, private Docker container on a high-performance VPS just for you.

No hardware required.

True isolation (your own container).

24/7 Uptime.

Your own custom domain.