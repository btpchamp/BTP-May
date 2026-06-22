# Day 26: Git & GitHub for Team Collaboration

---

## Day Schedule (8 Hours)

| Time | Session | Duration |
|------|---------|----------|
| 09:00 - 09:15 | Week 6 Kickoff & Motivation | 15 min |
| 09:15 - 10:30 | Session 1: Version Control & Core Git Concepts | 75 min |
| 10:30 - 10:45 | Break | 15 min |
| 10:45 - 12:00 | Session 2: Branching, Merging & Git Workflow | 75 min |
| 12:00 - 13:00 | Session 3: Hands-on — Initialize, Commit & Branch | 60 min |
| 13:00 - 13:45 | Lunch Break | 45 min |
| 13:45 - 14:45 | Session 4: Remote Repositories, Push, Pull & GitHub | 60 min |
| 14:45 - 15:00 | Break | 15 min |
| 15:00 - 16:00 | Session 5: Pull Requests, Code Review & Conflict Resolution | 60 min |
| 16:00 - 16:45 | Session 6: Hands-on — Push to GitHub & Resolve Conflicts | 45 min |
| 16:45 - 17:00 | Assessment & Assignment | 15 min |

---

## What You'll Learn Today

By the end of this session, you will be able to:
- Explain why version control is essential for any software project
- Use core Git commands: init, add, commit, status, log, diff
- Create and switch between branches for feature development
- Push code to GitHub and pull changes from teammates
- Configure `.gitignore` for CAP projects properly
- Create Pull Requests and understand code review workflow
- Resolve merge conflicts when two people edit the same file

---

## Week 6 Kickoff & Motivation (09:00 - 09:15)

### Where We Are in the Journey

```
Week 1-2: Foundations      → HTML, JS, Node.js, Express
Week 3:   Database Layer   → CAP, CDS, entities, views
Week 4:   Service Layer    → OData, CRUD, handlers, actions
Week 5:   UI Layer         → Fiori Elements, annotations, draft
Week 6:   DevOps & Deploy  → 👈 YOU ARE HERE!
```

### Why This Week Matters

You've built a complete application (DB + Service + UI). But in the real world:
- Multiple developers work on the SAME project
- Code needs to be backed up and versioned
- Changes need to be reviewed before going live
- The app must be deployed to the cloud

**This week: Git (collaboration) → Security → Deployment → HANA**

---

## Session 1: Version Control & Core Git Concepts (09:15 - 10:30)

### Why Do We Need Version Control?

**Without version control:**

```
project/
├── schema.cds
├── schema_v2.cds
├── schema_v2_final.cds
├── schema_v2_final_REAL.cds
├── schema_v2_final_REAL_fixed.cds        ← 😱 Which one is correct?!
├── schema_backup_june1.cds
└── schema_DO_NOT_DELETE.cds
```

**With Git:**

```
project/
├── schema.cds          ← Always the latest version

Git log shows:
  commit 5: "Add purchase order views"         (June 4)
  commit 4: "Fix supplier association"          (June 3)
  commit 3: "Add order items composition"       (June 2)
  commit 2: "Add basic product entity"          (June 1)
  commit 1: "Initial project setup"             (May 31)

  → You can go back to ANY version at any time!
```

---

### What is Git?

**Git** is a version control system that tracks changes to your files over time. Think of it as an "unlimited undo" system for your entire project.

```
┌─────────────────────────────────────────────────────────────┐
│                    WHAT GIT GIVES YOU                         │
├─────────────────────────────────────────────────────────────┤
│                                                             │
│  📜 HISTORY      — See every change ever made              │
│  ⏪ TIME TRAVEL  — Go back to any previous version         │
│  👥 COLLABORATION — Multiple people work on same project   │
│  🌿 BRANCHING    — Try new ideas without breaking main     │
│  🔒 BACKUP      — Code lives on a remote server (GitHub)   │
│  📋 BLAME       — See who changed what and when            │
│                                                             │
└─────────────────────────────────────────────────────────────┘
```

---

### Git vs GitHub — They're Different!

| Git | GitHub |
|-----|--------|
| The **tool** (runs on your computer) | The **website** (hosts your code online) |
| Version control software | Cloud platform for Git repositories |
| Works offline (local) | Requires internet (remote) |
| Command-line tool | Web interface + collaboration features |
| Free & open source | Free for public repos, paid for enterprise |

**Analogy:**
- Git = Microsoft Word (the tool you write documents with)
- GitHub = Google Drive (where you store and share documents)

---

### The Three Areas of Git

Git organizes your work into 3 areas:

```
┌──────────────────────────────────────────────────────────────┐
│                                                              │
│  ① WORKING DIRECTORY        ② STAGING AREA       ③ REPOSITORY│
│  (your actual files)         (ready to commit)    (committed) │
│                                                              │
│  ┌────────────────┐         ┌──────────────┐    ┌──────────┐│
│  │ schema.cds     │──add──►│ schema.cds   │─commit─►│ v1..v5 ││
│  │ service.cds    │         │              │    │ (history) ││
│  │ handler.js     │         └──────────────┘    └──────────┘│
│  └────────────────┘                                          │
│                                                              │
│  You EDIT here     You STAGE here      You SAVE here         │
│  (make changes)    (choose what to     (permanent record)    │
│                     save)                                     │
└──────────────────────────────────────────────────────────────┘
```

**Analogy — Shopping:**
1. **Working Directory** = Walking through the store (picking items off shelves)
2. **Staging Area** = Your shopping cart (what you've chosen but haven't paid for)
3. **Repository** = After checkout (confirmed purchase — it's official!)

---

### Core Git Commands — The Big 6

| Command | What It Does | Analogy |
|---------|-------------|---------|
| `git init` | Start tracking a folder | "Open a new bank account" |
| `git status` | See what's changed | "Check your bank balance" |
| `git add` | Stage files for commit | "Put items in cart" |
| `git commit` | Save a snapshot permanently | "Checkout / pay" |
| `git log` | View history of commits | "View transaction history" |
| `git diff` | See exact changes made | "Compare before/after" |

---

### `git init` — Start Version Control

```bash
# Navigate to your CAP project
cd my-cap-project

# Initialize git (only done ONCE per project!)
git init
```

Output:
```
Initialized empty Git repository in /path/my-cap-project/.git/
```

This creates a hidden `.git` folder that stores all version history. Never delete it!

---

### `git status` — What's Changed?

```bash
git status
```

Output:
```
On branch main

Untracked files:
  (use "git add <file>..." to include in what will be committed)
        db/
        srv/
        app/
        package.json

nothing added to commit but untracked files present
```

**Status tells you:**
- Which branch you're on
- What files are new (untracked)
- What files are modified
- What files are staged (ready to commit)

---

### `git add` — Stage Your Changes

```bash
# Add a specific file
git add db/schema.cds

# Add multiple files
git add db/schema.cds srv/catalog-service.cds

# Add all files in a folder
git add db/

# Add everything that's changed (use carefully!)
git add .
```

**After `git add`, files move from Working Directory → Staging Area.**

---

### `git commit` — Save a Permanent Snapshot

```bash
# Commit with a message describing WHAT you did
git commit -m "Add initial product and supplier entities"
```

Output:
```
[main abc1234] Add initial product and supplier entities
 3 files changed, 45 insertions(+)
 create mode 100644 db/schema.cds
 create mode 100644 srv/catalog-service.cds
 create mode 100644 package.json
```

**A commit is like taking a photograph — it captures EXACTLY what your code looks like at this moment.**

---

### Writing Good Commit Messages

```bash
# ❌ BAD commit messages:
git commit -m "update"
git commit -m "fix"
git commit -m "done"
git commit -m "asdfgh"
git commit -m "changes"

# ✅ GOOD commit messages:
git commit -m "Add product entity with price and stock fields"
git commit -m "Fix supplier association missing foreign key"
git commit -m "Add validation for negative stock values"
git commit -m "Implement PO approval action handler"
git commit -m "Add Fiori annotations for Product List Report"
```

**Rules for good commit messages:**
1. Start with a verb: Add, Fix, Update, Remove, Implement, Refactor
2. Describe WHAT changed (not how)
3. Keep it under 72 characters
4. Be specific (not "updated files")

---

### `git log` — View History

```bash
# Show commit history
git log

# Show compact one-line history
git log --oneline

# Show last 5 commits
git log --oneline -5
```

Output of `git log --oneline`:
```
abc1234 Add Fiori annotations for Product List Report
def5678 Implement PO approval action handler
ghi9012 Add validation for negative stock values
jkl3456 Add basic service and handler skeleton
mno7890 Initial project setup with cds init
```

Each commit has a **unique hash** (like `abc1234`) — you can use this to go back to any point!

---

### `git diff` — See Exact Changes

```bash
# See what's changed but not yet staged
git diff

# See what's staged (ready to commit)
git diff --staged

# Compare two commits
git diff abc1234 def5678
```

Output:
```diff
--- a/db/schema.cds
+++ b/db/schema.cds
@@ -5,6 +5,7 @@ entity Products : cuid, managed {
   productName : String(100);
   price       : Decimal(10,2);
   stock       : Integer default 0;
+  rating      : Decimal(2,1) default 0.0;    ← Added this line!
 }
```

Green `+` lines = added. Red `-` lines = removed.

---

### The Basic Git Workflow (Daily Use)

```
1. Make changes to your files (write code)
2. git status              → see what changed
3. git add <files>         → stage the changes
4. git commit -m "..."     → save permanently
5. Repeat!
```

```bash
# Example daily workflow:
vim db/schema.cds          # Edit a file
git status                 # "modified: db/schema.cds"
git add db/schema.cds      # Stage it
git commit -m "Add rating field to Products entity"  # Commit!
```

---

## Session 2: Branching, Merging & Git Workflow (10:45 - 12:00)

### What is a Branch?

A **branch** is a parallel copy of your code where you can make changes without affecting the main version.

**Analogy — Writing a Book:**
- `main` branch = the published book (stable, complete)
- `feature/add-chapter-5` branch = your draft of chapter 5 (work in progress)
- When chapter 5 is done → merge it into the main book!

```
                    main branch (stable)
─────●────●────●────●────●────●────●──────────────────
                    │              ▲
                    │              │ merge
                    ▼              │
                    ●────●────●───┘
                    feature branch (your work)
```

---

### Why Use Branches?

| Without Branches | With Branches |
|-----------------|---------------|
| Everyone edits the same code | Each person works in isolation |
| Your half-finished code breaks the app for everyone | Main branch always works |
| Hard to undo a bad feature | Just delete the branch |
| No code review before merging | Pull Requests enforce review |
| "Who broke it?" drama | Clear ownership per branch |

---

### Branch Commands

```bash
# See all branches (* marks current branch)
git branch
# Output: * main

# Create a new branch
git branch feature/add-po-validation

# Switch to the new branch
git checkout feature/add-po-validation
# OR (newer syntax):
git switch feature/add-po-validation

# Create AND switch in one command
git checkout -b feature/add-po-validation
# OR:
git switch -c feature/add-po-validation

# Switch back to main
git checkout main

# Delete a branch (after merge)
git branch -d feature/add-po-validation
```

---

### Branch Naming Conventions

```bash
# ✅ GOOD branch names:
feature/add-po-approval      # New feature
feature/product-annotations  # New feature
fix/stock-negative-bug       # Bug fix
fix/login-redirect-issue     # Bug fix
refactor/handler-cleanup     # Code improvement
docs/update-readme           # Documentation

# ❌ BAD branch names:
my-branch
test
stuff
fix
john-working
```

**Pattern:** `type/short-description`
- `feature/` — new functionality
- `fix/` — bug fix
- `refactor/` — code improvement (no new features)
- `docs/` — documentation only

---

### Merging — Bringing Branches Together

When your feature is done, merge it back into `main`:

```bash
# Step 1: Switch to main
git checkout main

# Step 2: Merge your feature branch INTO main
git merge feature/add-po-validation

# Step 3: Delete the feature branch (it's been merged)
git branch -d feature/add-po-validation
```

**Visual:**
```
BEFORE MERGE:
main:    ──●────●────●
                      │
feature:              └──●────●────● (your changes)

AFTER MERGE:
main:    ──●────●────●────────────●  (includes feature changes!)
                                   ↑
                                merge commit
```

---

### The Team Git Workflow (Feature Branch Strategy)

```
┌─────────────────────────────────────────────────────────────┐
│              TEAM GIT WORKFLOW                                │
├─────────────────────────────────────────────────────────────┤
│                                                             │
│  1. Pull latest main:    git pull origin main               │
│  2. Create branch:       git checkout -b feature/my-task    │
│  3. Make changes:        (write code, test)                 │
│  4. Commit:              git add . && git commit -m "..."   │
│  5. Push branch:         git push origin feature/my-task    │
│  6. Create Pull Request: (on GitHub website)                │
│  7. Code Review:         (teammate reviews your code)       │
│  8. Merge PR:            (merge into main on GitHub)        │
│  9. Pull updated main:   git pull origin main               │
│ 10. Delete branch:       git branch -d feature/my-task      │
│                                                             │
│  Repeat for next task!                                      │
│                                                             │
└─────────────────────────────────────────────────────────────┘
```

---

## Session 3: Hands-on — Initialize, Commit & Branch (12:00 - 13:00)

### Exercise 1: Initialize and First Commit (15 minutes)

```bash
# 1. Navigate to your CAP project
cd ~/projects/po-management

# 2. Initialize git (if not already done)
git init

# 3. Check status — see all untracked files
git status

# 4. Create .gitignore FIRST (before adding files)
# (see .gitignore content below)

# 5. Add all files
git add .

# 6. Make your first commit
git commit -m "Initial project setup: PO Management CAP application"

# 7. Verify
git log --oneline
```

---

### The `.gitignore` File for CAP Projects

**Create this file at the project root: `.gitignore`**

```gitignore
# Node.js
node_modules/
package-lock.json

# CAP generated files
gen/
app/*/dist/

# Database files (local SQLite)
db/*.db
db/*.sqlite
*.db

# IDE files
.vscode/
.idea/
*.swp
*.swo
*~

# OS files
.DS_Store
Thumbs.db

# Environment files (NEVER commit secrets!)
.env
default-env.json
*.pem
*.key

# Build artifacts
dist/
mta_archives/
*.mtar

# Logs
*.log
npm-debug.log*
```

**Why `.gitignore` matters:**
- `node_modules/` = 30,000+ files! Never commit this (use `npm install` to recreate)
- `*.db` = SQLite databases are local dev data (regenerated by `cds deploy`)
- `.env` = Contains secrets (API keys, passwords) — NEVER in Git!
- `gen/` = Generated files that can be recreated

---

### Exercise 2: Make Changes and Commit (15 minutes)

```bash
# 1. Create a new file
echo "// Product validation handler" > srv/product-validation.js

# 2. Check what changed
git status
# Output: new file: srv/product-validation.js

# 3. Stage the file
git add srv/product-validation.js

# 4. Commit with a message
git commit -m "Add product validation handler skeleton"

# 5. Make another change (edit schema.cds — add a field)
# ... edit the file ...

# 6. See the diff
git diff db/schema.cds

# 7. Stage and commit
git add db/schema.cds
git commit -m "Add rating field to Products entity"

# 8. View history
git log --oneline
```

---

### Exercise 3: Create and Work on a Branch (30 minutes)

```bash
# 1. Create a feature branch
git checkout -b feature/add-supplier-validation

# 2. Verify you're on the new branch
git branch
# Output:
#   main
# * feature/add-supplier-validation

# 3. Make changes (add validation logic in handler)
# ... edit srv/purchasing-service.js ...

# 4. Commit on the feature branch
git add srv/purchasing-service.js
git commit -m "Add supplier existence check in PO validation"

# 5. Make another change and commit
# ... edit srv/annotations.cds ...
git add srv/annotations.cds
git commit -m "Add supplier value help annotation"

# 6. View log (shows feature branch commits)
git log --oneline
# Output:
# aaa1111 Add supplier value help annotation
# bbb2222 Add supplier existence check in PO validation
# ccc3333 Add rating field to Products entity      ← from main
# ddd4444 Initial project setup

# 7. Switch back to main (your feature changes disappear from files!)
git checkout main

# 8. Look at your files — feature changes are gone! (They're on the branch)
git log --oneline
# Output: only original commits

# 9. Merge the feature branch into main
git merge feature/add-supplier-validation
# Output: Fast-forward merge. Now main has the feature changes!

# 10. Delete the merged branch
git branch -d feature/add-supplier-validation

# 11. Verify
git log --oneline
# All commits now visible on main!
```

---

## Session 4: Remote Repositories, Push, Pull & GitHub (13:45 - 14:45)

### Local vs Remote

```
┌──────────────────┐              ┌──────────────────┐
│  YOUR COMPUTER   │              │    GITHUB.COM    │
│  (local repo)    │              │  (remote repo)   │
│                  │    push →    │                  │
│  git commit      │ ──────────► │  Stored safely   │
│  git branch      │              │  Team can see    │
│  git merge       │ ◄────────── │  Team pushes too │
│                  │    ← pull    │                  │
└──────────────────┘              └──────────────────┘
```

---

### Setting Up a Remote Repository

**Step 1: Create a repository on GitHub**
1. Go to https://github.com
2. Click "+" → "New repository"
3. Name it: `po-management` (or your project name)
4. Choose: Public or Private
5. DON'T initialize with README (you already have code!)
6. Click "Create repository"

**Step 2: Connect your local repo to GitHub**

```bash
# Add the remote URL (only done ONCE per project)
git remote add origin https://github.com/YOUR-USERNAME/po-management.git

# Verify the remote is added
git remote -v
# Output:
# origin  https://github.com/YOUR-USERNAME/po-management.git (fetch)
# origin  https://github.com/YOUR-USERNAME/po-management.git (push)
```

---

### Push — Send Your Code to GitHub

```bash
# Push main branch to GitHub (first time: -u sets upstream tracking)
git push -u origin main

# After the first time, just:
git push
```

**What happens:** Your commits are uploaded to GitHub. Your teammates can now see and pull your code!

---

### Pull — Get Teammates' Changes

```bash
# Get and apply latest changes from GitHub
git pull origin main

# Shorter (if tracking is set up):
git pull
```

**When to pull:** ALWAYS before starting new work! Get the latest changes first.

---

### Clone — Download an Existing Project

When you join a team that already has a repo:

```bash
# Download the entire project (with full history!)
git clone https://github.com/team/po-management.git

# This creates a folder 'po-management' with all code
cd po-management
npm install    # Install dependencies
cds watch      # Start the project!
```

---

### The Push/Pull Dance (Daily Workflow)

```bash
# MORNING: Start your day
git pull origin main                    # Get latest from team

# DURING: Work on your feature
git checkout -b feature/my-task         # Create branch
# ... write code ...
git add .
git commit -m "Implement feature X"

# AFTERNOON: Share your work
git push origin feature/my-task         # Push YOUR branch

# ON GITHUB: Create Pull Request (PR)
# Teammate reviews → approves → merges

# NEXT MORNING:
git checkout main                       # Switch to main
git pull origin main                    # Get the merged changes
git branch -d feature/my-task          # Clean up old branch
```

---

### Common Remote Commands

| Command | What It Does |
|---------|-------------|
| `git remote add origin <url>` | Connect local repo to GitHub |
| `git push origin main` | Upload commits to GitHub |
| `git push origin feature/x` | Upload a feature branch |
| `git pull origin main` | Download and apply latest changes |
| `git clone <url>` | Download an entire repository |
| `git fetch origin` | Download changes without applying (just check) |
| `git remote -v` | Show connected remotes |

---

### What to NEVER Push

```
❌ NEVER push these to GitHub:
  • node_modules/     (too large! use .gitignore)
  • .env files        (contains secrets!)
  • *.db files        (local database)
  • API keys          (security risk!)
  • passwords         (security risk!)
  • default-env.json  (BTP credentials)
  
✅ ALWAYS push these:
  • db/schema.cds     (your data model)
  • srv/*.cds         (service definitions)
  • srv/*.js          (handlers)
  • app/              (Fiori app files)
  • package.json      (dependencies list)
  • .gitignore        (what to exclude)
  • db/data/*.csv     (sample data)
```

---

## Session 5: Pull Requests, Code Review & Conflict Resolution (15:00 - 16:00)

### What is a Pull Request (PR)?

A **Pull Request** is a way to say: "Hey team, I finished my feature. Can someone review it before we merge it into main?"

```
┌─────────────────────────────────────────────────────────────┐
│                    PULL REQUEST FLOW                          │
├─────────────────────────────────────────────────────────────┤
│                                                             │
│  1. You push your branch to GitHub                          │
│  2. You create a Pull Request (on GitHub website)           │
│  3. Teammate reviews your code (comments, suggestions)      │
│  4. You fix any issues they found                          │
│  5. Teammate approves ✅                                    │
│  6. PR is merged into main                                 │
│  7. Your feature is now live! 🎉                           │
│                                                             │
└─────────────────────────────────────────────────────────────┘
```

---

### Creating a Pull Request on GitHub

1. Push your branch: `git push origin feature/add-po-validation`
2. Go to GitHub → your repository
3. You'll see a banner: "feature/add-po-validation had recent pushes — Compare & pull request"
4. Click **"Compare & pull request"**
5. Fill in:
   - **Title:** "Add supplier validation to PO creation"
   - **Description:** What you changed and why
6. Assign a reviewer
7. Click **"Create pull request"**

---

### Code Review — What Reviewers Check

| Check | Example |
|-------|---------|
| Correctness | Does the code actually work? Are there bugs? |
| Style | Does it follow team conventions? (naming, formatting) |
| Security | Any hardcoded secrets? SQL injection risks? |
| Performance | Any N+1 queries? Unnecessary loops? |
| Test coverage | Are validations tested? Edge cases handled? |
| Documentation | Are complex parts explained? Good commit messages? |

---

### Merge Conflicts — When Two People Edit the Same Line

**When does a conflict happen?**

```
Person A (on branch-a):            Person B (on branch-b):
─────────────────────              ─────────────────────
Edits line 5 of schema.cds:       Also edits line 5 of schema.cds:
  price : Decimal(12,2);            price : Decimal(10,2);

Both push → merge → CONFLICT! Git doesn't know which is correct.
```

---

### What a Conflict Looks Like

```
// db/schema.cds (after failed merge)
entity Products : cuid {
  productName : String(100);
<<<<<<< HEAD
  price       : Decimal(12,2);    // YOUR change
=======
  price       : Decimal(10,2);    // THEIR change
>>>>>>> feature/other-branch
  stock       : Integer;
}
```

**The markers:**
- `<<<<<<< HEAD` → Start of YOUR version
- `=======` → Divider
- `>>>>>>> branch-name` → End of THEIR version

---

### Resolving a Merge Conflict

**Step 1:** Open the conflicted file

**Step 2:** Choose which version to keep (or combine both):

```cds
// RESOLVED (pick one or combine):
entity Products : cuid {
  productName : String(100);
  price       : Decimal(12,2);    // We decided 12,2 is correct
  stock       : Integer;
}
```

Remove ALL conflict markers (`<<<<<<<`, `=======`, `>>>>>>>`).

**Step 3:** Stage and commit the resolution:

```bash
git add db/schema.cds
git commit -m "Resolve merge conflict: use Decimal(12,2) for price"
```

---

### Avoiding Conflicts (Prevention > Cure)

| Tip | How |
|-----|-----|
| Pull frequently | `git pull` before starting work each day |
| Small, focused branches | One feature per branch (less overlap) |
| Communicate with team | "I'm working on schema.cds today" |
| Don't edit the same files | Split work by file/feature area |
| Merge main into your branch often | Keep up to date |

---

### When Push Gets Rejected

If `git push` fails with "rejected — fetch first":

```bash
# Fix: pull first, then push
git pull --rebase origin main
git push
```

This means someone else pushed to main while you were working. `--rebase` puts your changes on top of theirs cleanly.

---

## Session 6: Hands-on — Push to GitHub & Resolve Conflicts (16:00 - 16:45)

### Exercise 1: Push Your Project to GitHub (20 minutes)

```bash
# 1. Make sure .gitignore is set up
cat .gitignore    # Verify it has node_modules, *.db, .env

# 2. Stage and commit everything
git add .
git commit -m "Complete PO Management CAP application"

# 3. Create repo on GitHub (if not done already)
# Go to github.com → New repository → "po-management"

# 4. Add remote
git remote add origin https://github.com/YOUR-USERNAME/po-management.git

# 5. Push!
git push -u origin main

# 6. Open GitHub in browser → verify your code is there!
```

---

### Exercise 2: Feature Branch Workflow (15 minutes)

```bash
# 1. Create a feature branch
git checkout -b feature/improve-annotations

# 2. Make a change (add a label to an annotation)
# ... edit srv/annotations.cds ...

# 3. Commit
git add srv/annotations.cds
git commit -m "Improve product annotations with better labels"

# 4. Push the BRANCH (not main!)
git push origin feature/improve-annotations

# 5. Go to GitHub → Create a Pull Request
# 6. Review it yourself → Merge it
# 7. Come back to terminal:
git checkout main
git pull origin main
git branch -d feature/improve-annotations
```

---

### Exercise 3: Simulate and Resolve a Merge Conflict (10 minutes)

```bash
# 1. Create two branches from main
git checkout -b branch-a
# Edit line 10 of db/schema.cds (change stock default to 5)
git add . && git commit -m "Set default stock to 5"

git checkout main
git checkout -b branch-b
# Edit the SAME line 10 of db/schema.cds (change stock default to 100)
git add . && git commit -m "Set default stock to 100"

# 2. Merge branch-a into main
git checkout main
git merge branch-a    # Works fine!

# 3. Try to merge branch-b (CONFLICT!)
git merge branch-b
# Output: CONFLICT in db/schema.cds

# 4. Open the file → you'll see conflict markers
# 5. Choose the correct value and remove markers
# 6. Resolve:
git add db/schema.cds
git commit -m "Resolve conflict: keep stock default as 100"

# 7. Clean up
git branch -d branch-a
git branch -d branch-b
```

---

## Assessment: MCQ — 15 Questions on Git & GitHub

**Q1.** What does `git init` do?

- A) Installs Git on your computer
- B) Creates a new GitHub repository
- C) Initializes a new Git repository in the current folder
- D) Clones a remote repository

**Answer: C** — `git init` creates the `.git` folder that tracks changes. Run once per project.

---

**Q2.** The three areas in Git are:

- A) Local, Remote, Cloud
- B) Working Directory, Staging Area, Repository
- C) Branch, Merge, Tag
- D) Add, Commit, Push

**Answer: B** — Working Directory (your files) → Staging (git add) → Repository (git commit).

---

**Q3.** `git add .` does what?

- A) Commits all files
- B) Stages ALL changed and new files for the next commit
- C) Pushes to GitHub
- D) Creates a new branch

**Answer: B** — `git add .` stages everything. Use `git add <file>` for specific files.

---

**Q4.** What makes a good commit message?

- A) `git commit -m "update"`
- B) `git commit -m "Add PO validation for negative amounts"`
- C) `git commit -m "fixed stuff"`
- D) `git commit -m "wip"`

**Answer: B** — Starts with a verb, describes WHAT changed, is specific.

---

**Q5.** A branch in Git is:

- A) A copy of the entire repository
- B) A parallel line of development that doesn't affect other branches
- C) A tag marking a release
- D) A folder on your computer

**Answer: B** — Branches are independent lines of work. Changes on one branch don't affect others until merged.

---

**Q6.** `git checkout -b feature/new-api` does what?

- A) Deletes the feature branch
- B) Creates a new branch AND switches to it in one command
- C) Merges the feature branch
- D) Pushes the branch to remote

**Answer: B** — `-b` creates the branch, `checkout` switches to it. Combined: create + switch.

---

**Q7.** To get the latest changes from GitHub:

- A) `git push origin main`
- B) `git pull origin main`
- C) `git add origin main`
- D) `git merge origin main`

**Answer: B** — `git pull` downloads AND applies remote changes to your local branch.

---

**Q8.** `.gitignore` is used to:

- A) Ignore Git commands
- B) Specify files that Git should NOT track (like node_modules, .env)
- C) Hide commits from the log
- D) Delete files from the repository

**Answer: B** — Files listed in `.gitignore` are excluded from tracking. They won't be committed or pushed.

---

**Q9.** Which should NEVER be committed to Git?

- A) `package.json`
- B) `node_modules/` and `.env` files
- C) `db/schema.cds`
- D) `srv/handlers.js`

**Answer: B** — `node_modules` is too large (recreated by `npm install`). `.env` contains secrets.

---

**Q10.** A Pull Request (PR) is:

- A) A command to download code
- B) A request to review and merge your branch into main
- C) A way to delete branches
- D) A Git conflict

**Answer: B** — PRs are for code review. Your team reviews your changes before they become part of main.

---

**Q11.** A merge conflict occurs when:

- A) You forget to commit
- B) Two people edit the same line in the same file on different branches
- C) You push without pulling
- D) Your internet disconnects

**Answer: B** — Conflicts happen when Git can't automatically decide which change to keep for the same line.

---

**Q12.** To resolve a merge conflict, you must:

- A) Delete the file and recreate it
- B) Edit the file to remove conflict markers, choose the correct code, then add and commit
- C) Run `git conflict-resolve`
- D) Create a new branch

**Answer: B** — Manually edit the file, remove `<<<<<<<` `=======` `>>>>>>>` markers, keep correct code, then `git add` + `git commit`.

---

**Q13.** `git push` was rejected because "remote contains work you don't have." Fix:

- A) `git push --force` (dangerous!)
- B) `git pull --rebase origin main` then `git push`
- C) Delete the remote repository
- D) `git reset --hard`

**Answer: B** — Pull first (with rebase to keep clean history), then push. Never force-push to shared branches.

---

**Q14.** `git clone <url>` does what?

- A) Creates an empty repository
- B) Downloads an entire repository with all history to your computer
- C) Pushes your code to the URL
- D) Creates a branch

**Answer: B** — Clone downloads everything: code, branches, history. You're ready to work immediately.

---

**Q15.** The recommended branching strategy for teams is:

- A) Everyone commits directly to main
- B) Feature branches: create a branch per task, merge via PR after review
- C) One branch per person forever
- D) Never use branches

**Answer: B** — Feature branch workflow: short-lived branches per task, merged via reviewed Pull Requests.

---

## Assignment: Push PO Management Project to GitHub

### Task

Push your complete PO Management CAP project to GitHub using proper branching strategy.

### Steps

1. **Create a GitHub repository** named `cap-po-management`

2. **Initialize and push main branch:**
   ```bash
   git init
   # Create .gitignore for CAP
   git add .
   git commit -m "Initial setup: PO Management CAP application"
   git remote add origin https://github.com/YOUR-NAME/cap-po-management.git
   git push -u origin main
   ```

3. **Create 3 feature branches** (one at a time) and push each:

   | Branch | Changes to Make | Commit Message |
   |--------|----------------|----------------|
   | `feature/db-model` | Add/update `db/schema.cds` with all entities | "Add PO data model with entities and associations" |
   | `feature/services` | Add `srv/` service definitions and handlers | "Implement PO service with CRUD and actions" |
   | `feature/fiori-ui` | Add `app/` annotations and Fiori config | "Add Fiori Elements annotations for PO management" |

4. **For each branch:**
   ```bash
   git checkout -b feature/db-model
   # Make changes...
   git add .
   git commit -m "Add PO data model with entities and associations"
   git push origin feature/db-model
   # Create PR on GitHub → Merge into main
   git checkout main
   git pull origin main
   ```

5. **Verify on GitHub:**
   - All code visible on GitHub
   - 3 merged Pull Requests in the history
   - Clean commit history with meaningful messages
   - `.gitignore` properly excludes node_modules, *.db, .env

---

## End of Day Summary

### What We Learned

| Topic | Key Takeaway |
|-------|-------------|
| Git | Version control system — tracks all changes to your code |
| Working Directory → Staging → Repository | Three stages of saving changes |
| `git add` + `git commit` | Stage changes then save permanently |
| Branches | Parallel lines of work — feature branches keep main stable |
| GitHub | Cloud hosting for Git repos — collaboration platform |
| `git push` / `git pull` | Upload/download changes to/from GitHub |
| `.gitignore` | Excludes node_modules, .env, *.db from tracking |
| Pull Requests | Code review before merging — quality gate |
| Merge Conflicts | Two people edit same line — resolve manually |
| Feature Branch Workflow | Branch per task → PR → Review → Merge |

### The One-Liner

> **Git = unlimited undo for your code + safe collaboration with your team. Branch, commit, push, review, merge — repeat.**

---

### Looking Ahead: Day 27

Tomorrow: **Authentication & Authorization in CAP**
- Securing your services
- Adding user authentication
- Role-based access control
- `@requires` and `@restrict` annotations
- Testing with mock users

---

*End of Day 26 — You can now collaborate with any development team using Git & GitHub!*
