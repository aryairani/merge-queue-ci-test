# merge-queue-ci-test

A sample repository demonstrating GitHub Actions with merge queues.

## Workflows

### Check A (`check-a.yml`)
- **Trigger:** `push`
- Runs on every push, testing the exact pushed commit SHA (not a synthetic merge commit).
- Must pass before a PR can be added to the merge queue.
- Configure this as a required status check in the branch protection rule for `main`.

### Check B (`check-b.yml`)
- **Trigger:** `merge_group`
- Runs when a PR enters the merge queue.
- Must pass before the PR is merged into the main branch.
- Configure this as a required status check in the merge queue rule for `main`.

## Configuring the Repository

### Prerequisites — plan and visibility requirements

The **Require merge queue** option is only visible in the GitHub UI when both conditions are met:

| Repository visibility | Plan required |
|---|---|
| **Public** | Any plan (Free, Pro, Team, Enterprise) |
| **Private** | GitHub **Team** or **Enterprise** |

If you don't see **Require merge queue** in the UI, your repository is likely private and on the free plan. Either make the repository public, upgrade the plan, or use the [GitHub CLI approach](#option-c--github-cli--rest-api) below.

> **Tip:** Status check names (`Check A`, `Check B`) are registered the first time a workflow run completes. If they don't appear in the search box yet, push a commit (to register Check A) and open a PR and add it to the queue (to register Check B), then come back and add the checks.

---

### Option A — Classic branch protection rules (most widely available)

This path works for **public repositories on any plan** and also for private repos on Team/Enterprise.

1. Go to **Settings → Branches → Branch protection rules → Add rule** (or **Add classic branch protection rule**).
2. Set **Branch name pattern** to `main`.
3. Check **Require status checks to pass before merging**.
   - Search for `Check A` and add it as a required check.
4. Check **Require merge queue**.
   - This reveals merge queue options; leave defaults unless you have a preference.
5. In the merge queue's **Required checks for the merge queue** field, add `Check B`.
6. Click **Create** (or **Save changes**).

---

### Option B — Rulesets UI (GitHub Team / Enterprise, or public repos)

You can also apply the rules by importing the pre-built ruleset files in [`rulesets/`](rulesets/).

1. Go to **Settings → Rules → Rulesets**.
2. Click **New ruleset → Import a ruleset**.
3. Import [`rulesets/require-check-a.json`](rulesets/require-check-a.json).
   - This creates a branch ruleset on `main` that requires **Check A** to pass and enables the merge queue.
4. Import [`rulesets/require-check-b.json`](rulesets/require-check-b.json).
   - This requires **Check B** inside the merge queue before merging.
5. Verify both rulesets are **Active** in the list.

To configure manually instead of importing:

**Ruleset 1 — Require Check A + enable merge queue**
1. **Settings → Rules → Rulesets → New ruleset → New branch ruleset**.
2. Name: `Require Check A`, Enforcement: **Active**, Target branch: `main`.
3. Enable **Require status checks to pass** → add `Check A`.
4. Enable **Require merge queue** (defaults are fine).
5. Click **Create**.

**Ruleset 2 — Require Check B in merge queue**
1. Same path, name it `Require Check B (merge queue)`, target `main`.
2. Enable **Require status checks to pass** → add `Check B`.
3. Click **Create**.

---

### Option C — GitHub CLI / REST API

If you prefer the command line or need to script the setup, you can configure branch protection (including merge queue) using the GitHub REST API via `gh api`. The full set of parameters for the branch protection endpoint is documented at [GitHub Docs — Update branch protection](https://docs.github.com/en/rest/branches/branch-protection?apiVersion=2022-11-28#update-branch-protection). Merge queue settings live under the `required_merge_queue` field.

```bash
# Minimal example — requires a token with 'repo' scope
REPO="aryairani/merge-queue-ci-test"

gh api --method PUT "repos/$REPO/branches/main/protection" \
  --input - <<'EOF'
{
  "required_status_checks": {
    "strict": false,
    "contexts": ["Check A"]
  },
  "enforce_admins": false,
  "required_pull_request_reviews": null,
  "restrictions": null,
  "required_merge_queue": true
}
EOF
```

After enabling the merge queue this way, go to **Settings → Branches → Branch protection rules → main** and add `Check B` under **Required checks for the merge queue**.

> **Note:** The merge queue REST API also respects the same plan/visibility requirements. For private repos on the free plan, upgrading to Team is the only path.

---

### How the two-gate flow works

| Event | Workflow | Gate |
|---|---|---|
| Push to PR branch | `check-a.yml` (trigger: `push`) | PR must pass before it can enter the merge queue |
| PR enters merge queue | `check-b.yml` (trigger: `merge_group`) | Queue entry must pass before it merges into `main` |
