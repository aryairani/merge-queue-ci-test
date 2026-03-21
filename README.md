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

You can apply the required rules either through the GitHub UI or by importing the pre-built ruleset files in [`rulesets/`](rulesets/).

### Option A — Import rulesets (fastest)

1. Go to **Settings → Rules → Rulesets**.
2. Click **New ruleset → Import a ruleset**.
3. Import [`rulesets/require-check-a.json`](rulesets/require-check-a.json).
   - This creates a branch ruleset on `main` that requires **Check A** to pass and enables the merge queue.
4. Import [`rulesets/require-check-b.json`](rulesets/require-check-b.json).
   - This creates a merge-queue ruleset on `main` that requires **Check B** before merging.
5. Verify both rulesets are **Active** in the Rulesets list.

> **Note:** After import, open each ruleset and confirm the branch targeting pattern (`main`) and enforcement status look correct before saving.

---

### Option B — Configure manually through the UI

#### Step 1 — Require Check A before entering the merge queue

1. Go to **Settings → Rules → Rulesets → New ruleset → New branch ruleset**.
2. Set **Ruleset name** to `Require Check A`.
3. Set **Enforcement status** to **Active**.
4. Under **Target branches**, click **Add target → Include by pattern** and enter `main`.
5. Scroll to **Rules** and enable **Require status checks to pass**.
   - Click **Add checks**, search for `Check A`, and select it.
   - Leave *Require branches to be up to date* unchecked (the `push` trigger already tests the exact HEAD).
6. Still under **Rules**, enable **Require merge queue**.
   - Leave merge-method and grouping settings at their defaults unless you have a preference.
7. Click **Create**.

#### Step 2 — Require Check B before merging out of the queue

1. Go to **Settings → Rules → Rulesets → New ruleset → New branch ruleset**.
2. Set **Ruleset name** to `Require Check B (merge queue)`.
3. Set **Enforcement status** to **Active**.
4. Under **Target branches**, add `main` (same as above).
5. Under **Rules**, enable **Require status checks to pass**.
   - Click **Add checks**, search for `Check B`, and select it.
6. Click **Create**.

> **Tip:** Status check names are registered the first time a workflow run completes.
> If `Check A` or `Check B` don't appear in the search box yet, push a commit (to register Check A) and add a PR to the merge queue (to register Check B), then come back and add them.

---

### How the two-gate flow works

| Event | Workflow | Gate |
|---|---|---|
| Push to PR branch | `check-a.yml` (trigger: `push`) | PR must pass before it can enter the merge queue |
| PR enters merge queue | `check-b.yml` (trigger: `merge_group`) | Queue entry must pass before it merges into `main` |
