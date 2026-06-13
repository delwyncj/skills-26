# Updating this repo from Anthropic's upstream

This is a **fork/mirror** of Anthropic's official skills repository
([`anthropics/skills`](https://github.com/anthropics/skills)). Over time the
upstream repo gets new skills and fixes, and your copy falls behind. This guide
walks you through pulling those changes in **by hand**, so you learn the moving
parts instead of letting an automation do it.

> Goal: get the latest commits from `anthropics/skills` into your fork
> `delwyncj/skills-26`.

---

## Mental model (read this once)

You are juggling **three** copies of the project:

| Name        | What it is                                   | Git "remote" name |
|-------------|----------------------------------------------|-------------------|
| Upstream    | Anthropic's original repo (you can only read it) | `upstream`     |
| Origin      | Your fork on GitHub (you can read + write)    | `origin`         |
| Local       | The clone on this machine / in this session   | *(no remote — it's local)* |

The flow is always: **upstream → local → origin**

```
anthropics/skills (upstream)
        │  git fetch upstream
        ▼
   your local clone  ──── git merge ───►  updated local
        │  git push origin
        ▼
delwyncj/skills-26 (origin / your fork on GitHub)
```

`fetch` downloads commits but changes nothing in your files. `merge` is the step
that actually updates your files. `push` publishes the result to GitHub.

---

## One-time setup: add the `upstream` remote

Right now your repo only knows about `origin` (your fork). Check with:

```bash
git remote -v
```

You'll see only `origin`. Add Anthropic's repo as a second remote called
`upstream` (you only ever do this **once** per clone):

```bash
git remote add upstream https://github.com/anthropics/skills.git
```

Verify it worked — you should now see both `origin` and `upstream`:

```bash
git remote -v
```

---

## Step 1 — Save your work first (safety)

Make sure you have nothing uncommitted that could get lost:

```bash
git status
```

If it says **"nothing to commit, working tree clean"**, you're good. If not,
either commit your changes or stash them (`git stash`) before continuing.

---

## Step 2 — Download upstream's latest commits

```bash
git fetch upstream
```

This downloads Anthropic's latest history into your local repo but **does not
touch your files yet**. Nothing visible changes — that's expected.

---

## Step 3 — Preview what's new (optional but recommended)

Before merging, look at what you're about to pull in. This compares your current
branch to upstream's `main`:

```bash
# One line per new commit
git log HEAD..upstream/main --oneline

# See which files changed and by how much
git diff HEAD..upstream/main --stat
```

If the first command prints nothing, you're already up to date — you can stop here.

---

## Step 4 — Decide which branch to update

You have two reasonable choices.

### Option A — Update your working branch directly (simplest)

You are on `claude/busy-tesla-mr0y5d`. Merge upstream straight into it:

```bash
git merge upstream/main
```

### Option B — Update `main` first, then branch off (cleaner long-term habit)

This keeps your fork's `main` as a faithful copy of upstream, which is the more
conventional workflow:

```bash
git checkout main
git merge upstream/main      # fast-forwards main to match upstream
git push origin main         # publish the updated main to your fork
```

Then bring those updates into your feature branch:

```bash
git checkout claude/busy-tesla-mr0y5d
git merge main
```

> Because this fork has no local edits diverging from upstream, the merge in
> either option should **fast-forward** cleanly with no conflicts.

---

## Step 5 — Handle merge conflicts (only if Git complains)

If Git reports a conflict (it won't, unless you've edited the same files
upstream changed):

1. Run `git status` to see the conflicted files.
2. Open each one and look for the `<<<<<<<`, `=======`, `>>>>>>>` markers.
3. Edit the file to the version you want and delete the markers.
4. Mark it resolved and finish the merge:

```bash
git add <the-file-you-fixed>
git commit          # completes the merge (your editor opens — just save & close)
```

To bail out and return to before the merge:

```bash
git merge --abort
```

---

## Step 6 — Push the result to your fork

Publish the updated branch back to GitHub:

```bash
git push -u origin claude/busy-tesla-mr0y5d
```

If you used Option B and also want `main` updated on GitHub, you already pushed
it in Step 4.

---

## Step 7 — Confirm you're up to date

```bash
git fetch upstream
git log HEAD..upstream/main --oneline
```

If this prints **nothing**, your branch now contains everything upstream has. 🎉

---

## Quick reference (once setup is done)

After the one-time `git remote add upstream ...`, the recurring update is just:

```bash
git fetch upstream
git merge upstream/main
git push origin claude/busy-tesla-mr0y5d
```

---

## Where "GitHub Actions" fits in (bonus)

The steps above are plain **Git** commands. "**GitHub Actions**" is a separate
thing: it's GitHub's automation system that runs workflows (defined in
`.github/workflows/*.yml`) on events like a push or a pull request — for example
to run tests or auto-sync a fork. You don't need Actions to do the manual update
above. Once you're comfortable with the manual flow, a natural next exercise is
to look at the "**Sync fork**" button on your fork's GitHub page, which does
Steps 2–6 for you in the browser.
