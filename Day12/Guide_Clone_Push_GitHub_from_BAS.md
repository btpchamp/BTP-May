# Guide: Clone & Push Code to GitHub from SAP Business Application Studio (BAS)

## Step-by-Step for Beginners

---

## Prerequisites

Before you start, make sure you have:
- [ ] SAP Business Application Studio (BAS) Dev Space running
- [ ] A GitHub account (https://github.com)
- [ ] A repository already created on GitHub (can be empty)
- [ ] GitHub Personal Access Token (PAT) — we'll create this below

---

## Part 1: Generate a GitHub Personal Access Token (PAT)

GitHub no longer accepts passwords for git operations. You need a token.

### Steps:

1. Go to **https://github.com/settings/tokens**
2. Click **"Generate new token (classic)"**
3. Fill in:
   - **Note:** `BAS Access` (any descriptive name)
   - **Expiration:** 90 days (or your preference)
   - **Scopes:** Check ✅ `repo` (full control of private repositories)
4. Click **"Generate token"**
5. **COPY THE TOKEN IMMEDIATELY** — you won't see it again!
   - It looks like: `ghp_xxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxx`
   - Save it somewhere safe (notepad, password manager)

```
⚠️ IMPORTANT: This token is like a password.
   - Never share it
   - Never commit it to code
   - Regenerate if exposed
```

---

## Part 2: Open Terminal in BAS

1. Open SAP Business Application Studio
2. Open your Dev Space (should be "RUNNING")
3. Open Terminal: **Menu → Terminal → New Terminal**
   - Or use keyboard shortcut: `` Ctrl + ` ``
4. You should see a terminal prompt like:
   ```
   user: ~/projects $
   ```

---

## Part 3: Configure Git Identity (One-Time Setup)

Set your name and email (appears on commits):

```bash
git config --global user.name "Your Name"
git config --global user.email "your-email@example.com"
```

Verify:
```bash
git config --global user.name
git config --global user.email
```

---

## Part 4: Clone a Repository from GitHub

### Option A: Clone an EXISTING Repository (has code on GitHub)

```bash
# Navigate to projects folder:
cd ~/projects

# Clone the repository:
git clone https://github.com/YOUR-USERNAME/YOUR-REPO-NAME.git

# Enter the cloned folder:
cd YOUR-REPO-NAME

# Verify:
ls
```

**If asked for credentials:**
- Username: Your GitHub username
- Password: Paste your **PAT token** (NOT your GitHub password!)

---

### Option B: Clone Using Token in URL (Avoids Password Prompts)

```bash
git clone https://YOUR-USERNAME:YOUR-PAT-TOKEN@github.com/YOUR-USERNAME/YOUR-REPO-NAME.git
```

**Example:**
```bash
git clone https://btpchamp:ghp_abc123def456xyz@github.com/btpchamp/BTP-May.git
```

Then:
```bash
cd BTP-May
ls
```

---

### Option C: Push an EXISTING BAS Project to a New GitHub Repo

If you already have a project in BAS that's NOT connected to GitHub:

```bash
# Go to your project folder:
cd ~/projects/my-cap-project

# Initialize git (if not already):
git init

# Add all files:
git add .

# Create first commit:
git commit -m "Initial commit"

# Connect to GitHub (use your token in the URL):
git remote add origin https://YOUR-USERNAME:YOUR-TOKEN@github.com/YOUR-USERNAME/YOUR-REPO.git

# Push to GitHub:
git push -u origin main
```

---

## Part 5: Daily Workflow — Make Changes & Push

Once your repo is cloned/connected, here's your daily workflow:

### Step 1: Pull Latest Changes (Start of Day)

```bash
git pull origin main
```

This downloads any changes others (or you from another machine) pushed.

---

### Step 2: Make Your Changes

Edit files, create new files, add CDS entities, etc. using BAS editor.

---

### Step 3: Check What Changed

```bash
git status
```

You'll see:
```
modified:   db/schema.cds
new file:   srv/new-service.cds
```

---

### Step 4: Stage Your Changes

```bash
# Stage specific files:
git add db/schema.cds
git add srv/new-service.cds

# OR stage everything at once:
git add .
```

---

### Step 5: Commit with a Message

```bash
git commit -m "Add new entities for purchase order management"
```

**Good commit messages:**
- ✅ `"Add Books and Authors entities"`
- ✅ `"Fix price data type from Double to Decimal"`
- ❌ `"update"` (too vague!)
- ❌ `"asdfgh"` (meaningless!)

---

### Step 6: Push to GitHub

```bash
git push origin main
```

**Done!** Your code is now on GitHub.

---

## Part 6: Handle Common Issues

### Issue 1: "Push Rejected — Remote Contains Work You Don't Have"

```
error: failed to push some refs
hint: Updates were rejected because the remote contains work...
```

**Fix:**
```bash
git pull origin main --rebase
git push origin main
```

---

### Issue 2: "Authentication Failed"

```
remote: Invalid username or token
fatal: Authentication failed
```

**Fix:** Your token may be expired or incorrect. Update the remote URL:
```bash
git remote set-url origin https://YOUR-USERNAME:NEW-TOKEN@github.com/YOUR-USERNAME/YOUR-REPO.git
```

---

### Issue 3: "Remote Origin Already Exists"

```
error: remote origin already exists
```

**Fix:** Update instead of add:
```bash
git remote set-url origin https://YOUR-USERNAME:TOKEN@github.com/YOUR-USERNAME/REPO.git
```

---

### Issue 4: "Merge Conflicts"

If you and someone else changed the same file:

```bash
# After git pull shows conflicts:
# 1. Open the conflicted file in BAS editor
# 2. Look for <<<<<<< markers
# 3. Choose which version to keep (or combine both)
# 4. Remove the conflict markers (<<<, ===, >>>)
# 5. Save the file
# 6. Then:
git add .
git commit -m "Resolve merge conflict"
git push origin main
```

---

### Issue 5: "Not a Git Repository"

```
fatal: not a git repository
```

**Fix:** You're not inside a git project folder. Navigate to it:
```bash
cd ~/projects/your-project-name
```

Or initialize git:
```bash
git init
```

---

## Part 7: Useful Git Commands Reference

| Command | What It Does |
|---------|-------------|
| `git status` | Show what's changed |
| `git add .` | Stage all changes |
| `git commit -m "msg"` | Save staged changes |
| `git push origin main` | Upload to GitHub |
| `git pull origin main` | Download from GitHub |
| `git log --oneline -5` | Show last 5 commits |
| `git diff` | Show unsaved changes |
| `git remote -v` | Show connected remote URL |
| `git branch` | Show current branch |
| `git clone <url>` | Download a repo |

---

## Part 8: Store Credentials (Avoid Typing Token Every Time)

### Option A: Credential Cache (Temporary — 15 minutes)

```bash
git config --global credential.helper cache
```

### Option B: Store in URL (Permanent for this repo)

```bash
git remote set-url origin https://username:token@github.com/username/repo.git
```

### Option C: Credential Store (Permanent — stores on disk)

```bash
git config --global credential.helper store
```

Then the next time you push/pull and enter credentials, they'll be saved in `~/.git-credentials`.

```
⚠️ Note: Option C stores the token in plain text on BAS disk.
   Acceptable for trial/training, not recommended for production.
```

---

## Part 9: Best Practices for BAS + GitHub

| Practice | Why |
|----------|-----|
| Always `git pull` before starting work | Avoid conflicts with latest changes |
| Commit small, commit often | Easier to track and revert changes |
| Write meaningful commit messages | Future you will thank present you |
| Never commit `node_modules/` | Add to `.gitignore` (too large!) |
| Never commit tokens/secrets | Add `.env` and `default-env.json` to `.gitignore` |
| Create `.gitignore` in every project | Prevents accidental commits of junk |

### Sample `.gitignore` for CAP Projects

```gitignore
node_modules/
*.sqlite
default-env.json
.env
mta_archives/
gen/
```

---

## Quick Reference Card

```
┌─────────────────────────────────────────────────────┐
│  DAILY GIT WORKFLOW IN BAS                          │
├─────────────────────────────────────────────────────┤
│                                                     │
│  START OF DAY:                                      │
│    git pull origin main                             │
│                                                     │
│  AFTER MAKING CHANGES:                              │
│    git status              ← see what changed       │
│    git add .               ← stage everything       │
│    git commit -m "msg"     ← save with message      │
│    git push origin main    ← upload to GitHub       │
│                                                     │
│  IF PUSH FAILS:                                     │
│    git pull origin main --rebase                    │
│    git push origin main                             │
│                                                     │
└─────────────────────────────────────────────────────┘
```

---

*Keep this guide handy — you'll use it every day!*
