# Next Pending Task — sunnah-web

Workflow for picking and addressing the next open issue from the fork
(`hasankhan/sunnah-web`) and opening a PR against upstream
(`sunnah-com/website`).

## Conventions

- **Fork** (issue tracker): `hasankhan/sunnah-web`
- **Upstream** (PR target): `sunnah-com/website`, default branch `master`
- Working directory: `projects/sunnah-web/`
- Topic branches: `fix/issue-<N>-<short-slug>` branched from `upstream/master`
- Commits: include `Refs hasankhan/sunnah-web#<N>` trailer (NOT `Closes` —
  merging upstream does not auto-close fork issues; the user closes them
  manually after upstream merges).
- Always include the standard `Co-authored-by: Copilot ...` trailer (per AGENTS.global.md).

## Priority order

When the user does not specify an issue number, pick by label, in this order:

1. `security`
2. `bug`
3. `performance`
4. `refactor`
5. `frontend`
6. `tooling`
7. `content`
8. `documentation`

Within a label tier, pick the lowest issue number (oldest first).

**Skip rules:**
- Skip issues that have an `assignee` other than no-one (in progress elsewhere).
- Skip issues with `wontfix`, `duplicate`, or `blocked` labels.
- Skip issue **#50** (the meta-skill issue itself) and any other meta issue
  about workflow.
- Skip issues whose body says "Depends on #X" where X is still open.

## Steps

### 1. Pre-flight

```powershell
cd C:\Users\mhasa\dev\projects\sunnah-web
git status --porcelain   # must be empty
git fetch upstream
git fetch origin
git checkout master
git rebase upstream/master
git push origin master   # keep fork in sync
```

If the working tree is dirty, stop and ask the user.

### 2. Pick the issue

If the user supplied a number, use it. Otherwise:

```powershell
# List open issues sorted by label priority then number
gh issue list -R hasankhan/sunnah-web --state open --limit 100 `
  --json number,title,labels,assignees `
  | ConvertFrom-Json
```

Apply the priority order above. Print the picked issue:

> Picked #N: <title> (labels: ...)

### 3. Read the issue

```powershell
gh issue view <N> -R hasankhan/sunnah-web
```

Re-read the body carefully. If the "Suggested fix" is ambiguous or scope is
unclear, ask the user before coding. Don't expand scope beyond what the issue
describes.

### 4. Create a topic branch

```powershell
$slug = "<short-kebab-case>"   # e.g. "share-php-xss"
$branch = "fix/issue-$N-$slug"
git checkout -b $branch upstream/master
```

### 5. Implement the fix

- Make precise, surgical changes that fully address the issue.
- Fix the **pattern**, not just the point (AGENTS.global.md bug-fix rule):
  grep for the same anti-pattern across the codebase and fix all instances.
- Write a unit test if the fix is testable. Codeception is in `require-dev`
  even though no suite is wired yet (issue #40); if tests don't run in CI,
  document this in the PR and at least exercise the code path manually.
- For PHP changes, validate syntax: `php -l <file>`.
- For docker-runnable changes: spin up `docker-compose up --build` and curl
  the affected page.

### 6. Commit

Use a Conventional-Commit-ish prefix matching the label:

- `security:` for security
- `fix:` for bug
- `perf:` for performance
- `refactor:` for refactor
- `style:` or `frontend:` for frontend
- `ci:` / `chore:` for tooling
- `docs:` for documentation

```powershell
git add -A
git commit -m "<prefix>: <one-line summary>

<wrapped body explaining the change and why>

Refs hasankhan/sunnah-web#$N

Co-authored-by: Copilot <223556219+Copilot@users.noreply.github.com>"
```

### 7. Push and open the PR against upstream

```powershell
git push -u origin $branch

gh pr create `
  -R sunnah-com/website `
  --base master `
  --head "hasankhan:$branch" `
  --title "<prefix>: <summary>" `
  --body @"
## Summary

<what changed and why, ~3 sentences>

## Issue

Tracks hasankhan/sunnah-web#$N

## Testing

<how the change was verified>

## Notes

<any caveats, follow-ups, or scope-limitations>
"@
```

### 8. Cross-link on the fork issue

```powershell
$prUrl = gh pr list -R sunnah-com/website --head "hasankhan:$branch" `
  --json url --jq '.[0].url'
gh issue comment $N -R hasankhan/sunnah-web `
  --body "Submitted upstream PR: $prUrl"
```

### 9. Report

Tell the user:

> Opened upstream PR for #N (<title>): <PR URL>
> Fork issue: https://github.com/hasankhan/sunnah-web/issues/N
> Branch: $branch (pushed to origin)

Do NOT close the fork issue automatically. The user closes it after the
upstream PR is merged.

## When something goes wrong

- **`gh pr create` fails with "no commits between master and ..."**: you
  forgot to commit, or branched from the wrong base.
- **`gh pr create` says PR already exists**: someone (you, earlier) opened
  one. `gh pr view --head hasankhan:<branch> -R sunnah-com/website`.
- **Upstream rejects the branch name**: not possible — `head` is the fork.
- **Composer/PHP errors**: install dependencies with `composer install` in
  the project root.
- **Merge conflicts on rebase**: stop, ask the user; do not force-resolve.

## When to refuse and ask the user

- Issue scope is unclear or contradicts another open issue.
- Fix requires a breaking change to URL routes or DB schema.
- Issue is in the `security` tier and the fix touches authentication
  primitives — ask first, do not push silently.
- More than one file outside the scope of the issue needs touching.
