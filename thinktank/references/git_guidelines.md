# Git & Version Control Guidelines

This document outlines standard, high-leverage version control and git practices when working with Antigravity CLI and collaborative codebases.

---

## 1. Commit Message Standards (Conventional Commits)

Always write descriptive, structured commit messages that indicate the nature of the change. Use the following format:
`<type>(<scope>): <short description>`

### Common Types:
*   `feat`: A new user-facing feature.
*   `fix`: A bug fix.
*   `refactor`: Code changes that neither fix a bug nor add a feature (e.g., performance tuning, modularization).
*   `style`: Formatting, missing semi-colons, etc.; no production code change.
*   `docs`: Documentation only changes.
*   `test`: Adding missing tests or correcting existing tests.
*   `chore`: Updating build tasks, package manager configs, dependency versions, etc.

### Examples:
*   `feat(ui): add persistent bottom sheet for dashboard items`
*   `fix(auth): resolve token refresh loop on session expiry`
*   `chore(deps): update supabase-js to v2.87.1`

---

## 2. Commit Strategy: Atomic Commits

*   **One Change per Commit**: Commit small, logical, self-contained units of work. Avoid grouping unrelated fixes or features into a single large commit.
*   **Traceability**: Small commits make it trivial to revert bad changes, trace back bugs via `git bisect`, and run readable code reviews.
*   **Staging Selective Lines**: Use `git add -p` to stage specific hunks or lines if you have made multiple changes across files.

---

## 3. Branching Best Practices

*   **Prefix Branches**: Use descriptive prefixes for branch names:
    *   `feature/feature-name` — for new features.
    *   `bugfix/issue-description` — for standard bug fixes.
    *   `hotfix/urgent-issue` — for critical production patches.
    *   `refactor/cleanup-area` — for technical debt cleanup.
*   **Keep Branches Short-Lived**: Merge frequently back to the main branch to avoid large, painful merge conflicts.

---

## 4. Conflict Resolution & Safety

*   **Rebase vs. Merge**: Rebasing (`git pull --rebase origin main`) is preferred on local feature branches to keep a clean, linear git history before merging into the main branch.
*   **Conflict Markers**: When resolving merge conflicts, ensure you inspect the files carefully, remove conflict markers (`<<<<<<<`, `=======`, `>>>>>>>`), test the code, and run build checks before finalizing the merge.
*   **Stashing**: Use `git stash` to save dirty local changes before pulling updates or switching branches, and retrieve them with `git stash pop`.

---

## 5. Security & Sensitive Info Prevention (CRITICAL)

*   **Gitignore**: Always ensure there is a robust `.gitignore` in the repository root. Never track:
    *   Secrets, private keys, API tokens, or `.env` files.
    *   Large binary files (e.g., custom database dumps, unused assets).
    *   Local IDE files (e.g., `.vscode/`, `.idea/`).
    *   OS-specific system files (e.g., `.DS_Store`, `Thumbs.db`).
*   **Pre-Commit Verification**: If you accidentally commit a secret, use `git reset --soft HEAD~1` to un-commit it before pushing. If it is already pushed, immediately rotate the credentials.

---

## 6. Git Diagnostics & Debugging (Git Bisect)

When a regression bug is introduced and the cause is not obvious, use `git bisect` to locate the exact breaking commit:
1.  Start bisecting: `git bisect start`
2.  Mark the current commit as bad: `git bisect bad`
3.  Mark a known working past commit: `git bisect good <commit-hash>`
4.  Test the commits checked out by Git, marking them `git bisect good` or `git bisect bad` until the culprit commit is pinpointed.
5.  Reset bisect state when done: `git bisect reset`
