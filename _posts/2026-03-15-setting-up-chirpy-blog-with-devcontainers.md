---
title: "Setting Up a Jekyll Chirpy on Windows with Dev Containers"
date: 2026-03-15 10:00:00 +0530
categories: [StaticWebsite, JekyllCChirpy]
tags: [jekyll, chirpy, devcontainers, docker, wsl, github-pages, windows]
image:
  path: assets/posts/jekyllChirpy/blog1.png
  alt: Jekyll Chirpy Setup
---
> 🔗 [Official Chirpy Repository](https://github.com/cotes2020/jekyll-theme-chirpy)

So I wanted a clean blog where I could focus entirely on writing — no server management, no complicated hosting bills, just write and publish. After some research I landed on **Jekyll Chirpy** hosted on **GitHub Pages**. Free hosting, automatic deployments, and a beautiful theme out of the box.

This post is a complete guide to how I set it up on Windows using WSL, Docker, and VS Code Dev Containers. If you follow this exactly, you'll have a live blog by the end.

---

## What We Are Building

- A static blog powered by [Jekyll Chirpy](https://github.com/cotes2020/chirpy-starter)
- Hosted for free on GitHub Pages at `yourusername.github.io`
- A Dev Container environment so all dependencies are isolated inside Docker — no Ruby or Jekyll installation on your actual machine
- A workflow where you write a post, push to GitHub, and it deploys automatically in ~2 minutes

---

## Prerequisites

Before starting, make sure you have the following installed on Windows:

- **WSL 2** with Ubuntu distro
- **Docker Desktop** (with WSL 2 integration enabled)
- **VS Code** with the Dev Containers and WSL extensions

---

## Step 1 — Create the GitHub Repository

Go to the [Chirpy Starter](https://github.com/cotes2020/chirpy-starter) and click **"Use this template"** → **"Create a new repository"**.

Name it exactly:

```
yourusername.github.io
```

Set visibility to **Public** and leave **"Include all branches"** off. Click **Create repository**.

---

## Step 2 — Set Up WSL and Ubuntu

Open **PowerShell as Administrator** and run:

```powershell
wsl --install -d Ubuntu
```

After installation it will ask for a username and password. Set those up, then restart your PC.

Once restarted, open **Docker Desktop** → Settings → Resources → **WSL Integration** and toggle on **Ubuntu**. Click **Apply & Restart**.

---

## Step 3 — Set Up Git and SSH

Open the **Ubuntu terminal** from the Start menu and install Git:

```bash
sudo apt update && sudo apt install git -y
```

Configure your identity:

```bash
git config --global user.name "Your Name"
git config --global user.email "you@example.com"
```

Generate an SSH key so you can push to GitHub without passwords:

```bash
ssh-keygen -t ed25519 -C "you@example.com"
```

Press Enter three times to accept the defaults. Then copy your public key:

```bash
cat ~/.ssh/id_ed25519.pub
```

Go to **GitHub** → Profile → **Settings** → **SSH and GPG keys** → **New SSH key**, paste it in, and save.

Test it works:

```bash
ssh -T git@github.com
```

You should see:

```
Hi yourusername! You've successfully authenticated, but GitHub does not provide shell access.
```

The "no shell access" part is completely normal — that's the success message.

---

## Step 4 — Clone the Repository

Create a projects folder and clone your repo:

```bash
mkdir ~/Projects && cd ~/Projects
git clone git@github.com:yourusername/yourusername.github.io.git
cd yourusername.github.io
```

---

## Step 5 — Open in VS Code

First, add VS Code to your PATH so you can launch it from the terminal:

```bash
echo 'export PATH="$PATH:/mnt/c/Users/YOUR_WINDOWS_USERNAME/AppData/Local/Programs/Microsoft VS Code/bin"' >> ~/.bashrc
source ~/.bashrc
```

Replace `YOUR_WINDOWS_USERNAME` with your actual Windows username (check it by running `cmd.exe /c "echo %USERNAME%"` in Ubuntu).

Then open the project:

```bash
code .
```

VS Code will launch connected to Ubuntu, showing **WSL: Ubuntu** in the bottom-left corner.

---

## Step 6 — Open in Dev Container

Make sure the **Dev Containers** extension is installed in VS Code. Then press `Ctrl+Shift+P` and select:

```
Dev Containers: Reopen in Container
```

The first time this runs it will download Ruby, Jekyll, and all Chirpy dependencies inside Docker. This takes about 5–10 minutes. Once done, the bottom-left corner will show:

```
Dev Container: Jekyll Chirpy
```

If you see a permissions error about `Gemfile.lock`, run this in the VS Code terminal:

```bash
sudo chown -R $(whoami) /workspaces/yourusername.github.io
bundle install
```

---

## Step 7 — Configure Your Blog

Open `_config.yml` and update these fields:

```yaml
url: "https://yourusername.github.io"
title: "Your Blog Name"
tagline: "A short description"
timezone: "Asia/Kolkata"
lang: "en"
```

Save with `Ctrl+S`.

---

## Step 8 — Preview Locally

In the VS Code terminal run:

```bash
bundle exec jekyll serve
```

Open your browser and go to `http://127.0.0.1:4000`. Your blog is running locally.

**Pro tip:** Open a split view in VS Code with `Ctrl+Shift+P` → **Simple Browser: Show** → enter `http://127.0.0.1:4000` to see a live preview right inside VS Code alongside your editor.

---

## Step 9 — Deploy to GitHub Pages

Run this once to add Linux platform support to the lockfile:

```bash
bundle lock --add-platform x86_64-linux
```

Since the Dev Container has its own Git environment, configure your identity here too:

```bash
git config --global user.name "Your Name"
git config --global user.email "you@example.com"
```

Generate a new SSH key inside the container (the container can't access your WSL SSH key):

```bash
ssh-keygen -t ed25519 -C "you@example.com"
cat ~/.ssh/id_ed25519.pub
```

Add this new key to GitHub the same way as before — give it a title like **Dev Container**.

Then push:

```bash
git add .
git commit -m "initial setup"
git push
```

On GitHub, go to your repo → **Settings** → **Pages** → **Source** → select **GitHub Actions** → Save.

Go to the **Actions** tab and watch the build. Once you see three green checkmarks — build, report-build-status, and deploy — your site is live at:

```
https://yourusername.github.io
```

---

## Writing a New Post

Every post is a Markdown file in the `_posts/` folder. The filename must follow this exact format:

```
YYYY-MM-DD-your-post-title.md
```

Every post starts with front matter:

```markdown
---
title: "Your Post Title"
date: 2026-03-15 10:00:00 +0530
categories: [Category]
tags: [tag1, tag2]
---

Your content starts here...
```

After writing, push to deploy:

```bash
git add .
git commit -m "new post: your title"
git push
```

GitHub Actions handles the rest automatically.

---

## Daily Workflow

Once everything is set up, this is all you need to do every day:

```
1. Open Ubuntu terminal
2. Run: blog   (alias that opens VS Code in your project)
3. Ctrl+Shift+P → Dev Containers: Reopen in Container
4. Run: bundle exec jekyll serve
5. Write your post in _posts/
6. Preview at http://127.0.0.1:4000
7. git add . → git commit → git push
8. Done ✅
```

---

That is the entire setup. The goal was to spend time writing, not configuring — and this setup delivers exactly that. Every push to the main branch triggers an automatic build and deployment. No servers to manage, no costs, no complexity.
