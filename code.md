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
