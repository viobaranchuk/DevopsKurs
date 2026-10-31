# Branching strategy

## Summary

This repository uses **GitHub Flow**, a simple form of trunk-based development:

- `main` is the only long-lived branch. It must always be in a working state.
- Every change happens on a **short-lived branch** created from `main`.
- Every branch is merged back through a **pull request (PR)**, even though I am
  the only contributor.
- Finished milestones are marked with **tags** (`v0.1.0`, `v0.2.0`, ...).

## Why this strategy?

| Option | Verdict | Reason |
|---|---|---|
| Commit directly to `main` | No | Fast, but I would not practise PRs, reviews, or CI checks. |
| **GitHub Flow** | **Yes** | One main branch plus short feature branches. It is the most common workflow in modern DevOps teams and works well with CI/CD. |
| Git Flow (`develop`, `release/*`, `hotfix/*`) | No | Built for scheduled releases and large teams. Too much overhead for one person, and many teams no longer use it. |

The goal is to work **as if I were in a team**, so the habits carry over to a real job.

## Branches

### `main`

- Protected (see [Branch protection](#branch-protection)).
- No direct pushes. Changes arrive only through PRs.
- Every commit on `main` should be something I could "deploy".

### Short-lived branches

Format: `<type>/<short-description>`, lowercase, words separated by `-`.

| Type | Used for | Example |
|---|---|---|
| `feat/` | New content or functionality | `feat/docker-lesson` |
| `fix/` | Fixing a mistake | `fix/broken-link-in-readme` |
| `docs/` | Documentation only | `docs/branching-strategy` |
| `ci/` | GitHub Actions / pipelines | `ci/add-markdown-lint` |
| `chore/` | Maintenance (config, cleanup) | `chore/update-gitignore` |
| `exp/` | Experiments that may be thrown away | `exp/try-terraform` |

Rules:

- One branch = one topic. If it grows too big, split it.
- Keep branches short-lived (hours to a few days).
- Delete the branch after it is merged.

## Commit messages

Use [Conventional Commits](https://www.conventionalcommits.org/):

```
<type>: <what changed, in imperative form>
```

Examples:

```
docs: add branching strategy
feat: add lesson about Docker volumes
fix: correct typo in CI workflow name
ci: run markdown lint on pull requests
```

The types are the same as the branch prefixes above.

## Everyday workflow

```bash
# 1. Start from an up-to-date main
git switch main
git pull

# 2. Create a branch
git switch -c feat/docker-lesson

# 3. Work and commit in small steps
git add <files>
git commit -m "feat: add docker lesson outline"

# 4. Push the branch
git push -u origin feat/docker-lesson

# 5. Open a PR (fill in the PR template)
gh pr create --fill

# 6. Check the diff on GitHub, wait for CI to pass, then merge
gh pr merge --squash --delete-branch

# 7. Go back to main and clean up locally
git switch main
git pull
git branch -d feat/docker-lesson
```

### If `main` has moved while I worked on my branch

Prefer **rebase** to keep a straight history:

```bash
git fetch origin
git rebase origin/main
# fix any conflicts, then: git add <file> && git rebase --continue
git push --force-with-lease
```

`--force-with-lease` is the safe version of force-push: it refuses to overwrite
work on the remote that I have not seen. Never force-push to `main`.

## Pull requests

Even alone, PRs are worth it because they:

- give me a moment to re-read my own diff before it lands,
- are where CI checks (GitHub Actions) run,
- leave a history of *why* things changed (the PR template asks for this).

Merge method: **squash merge**. Each PR becomes one clean commit on `main`,
with the PR number in the message, e.g. `docs: add branching strategy (#1)`.

## Branch protection

Settings on GitHub → **Settings → Branches → Add branch ruleset** for `main`:

- [x] Require a pull request before merging
  - Required approvals: **0** (I cannot approve my own PR)
- [x] Require status checks to pass (once CI exists)
- [x] Block force pushes
- [x] Restrict deletions
- [x] Require linear history (fits with squash merge)

## Releases and tags

When a part of the course is finished, tag it using
[Semantic Versioning](https://semver.org/):

```bash
git switch main && git pull
git tag -a v0.1.0 -m "Module 1: Git basics"
git push origin v0.1.0
```

- `MAJOR` – big restructure of the course
- `MINOR` – new module or lesson
- `PATCH` – small fixes

Later, a GitHub Actions workflow can trigger on tags (`on: push: tags: ['v*']`)
to practise automatic releases.

## Hotfixes

There is no separate hotfix branch type. A hotfix is just a `fix/` branch from
`main` with a fast PR. This is one of the advantages of GitHub Flow.

## Possible next steps (to practise more DevOps)

1. Add a CI workflow in `.github/workflows/` that runs on every PR
   (for example markdown lint or a link checker).
2. Make that check *required* in branch protection.
3. Add a release workflow that runs on `v*` tags.
4. Try a deployment, e.g. publish `docs/` with GitHub Pages from `main`.
