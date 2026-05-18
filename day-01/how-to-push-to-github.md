# How to Push Work to GitHub
## Step-by-step guide — macOS — no prior experience needed

---

## What is GitHub
GitHub stores all your QA work publicly. Employers can see your portfolio, your daily commits, and every test case and bug report you write. This is your proof of experience.

---

## One-time setup

### 1. Create GitHub account
Go to https://github.com → Sign up → verify email → Free plan

### 2. Create repository
Go to https://github.com/new
- Name: qa-journey
- Public ✅
- Add README ✅
- Click Create repository

### 3. Check Git is installed
Open Terminal (Cmd + Space → Terminal → Enter)
```bash
git --version
```
If you see a version number — Git is installed.
If a popup appears — click Install and wait.

### 4. Configure Git
```bash
git config --global user.name "Your Name"
git config --global user.email "youremail@gmail.com"
```

### 5. Get your GitHub token
Go to https://github.com/settings/tokens
→ Generate new token (classic)
→ Note: qa-journey-token
→ Expiry: 90 days
→ Tick: repo ✅
→ Generate token
→ Copy the ghp_... token — save it in Notes app immediately

---

## Push your files — first time only

```bash
cd ~/Desktop/qa-journey
git init
git remote add origin https://github.com/YOUR-USERNAME/qa-journey.git
git add .
git commit -m "Day 01 — QA Foundations: test cases, bug reports, notes"
git branch -M main
git push -u origin main
```

When asked for password — paste your ghp_... token.

---

## Every day after that — just 4 commands

```bash
cd ~/Desktop/qa-journey
git add .
git commit -m "Day 02 — Test Design Techniques"
git push
```

---

## Commit message format to use every day

```
Day XX — [Topic]: [what you did]
```

Examples:
```
Day 01 — QA Foundations: 20 test cases, 5 bug reports for OrangeHRM login
Day 02 — Test Design: BVA and EP techniques, 15 test cases for leave module
Day 13 — Network Analysis: Charles Proxy setup, 20 API calls documented
```

---

## Common errors

**remote origin already exists**
```bash
git remote set-url origin https://github.com/YOUR-USERNAME/qa-journey.git
```

**Authentication failed**
Generate a new token at https://github.com/settings/tokens and use it as password.

**src refspec main does not match**
You forgot to commit. Run git add . then git commit -m "message" first.

