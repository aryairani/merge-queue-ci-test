# merge-queue-ci-test

A sample repository demonstrating GitHub Actions with merge queues.

## Workflows

### Check A (`check-a.yml`)
- **Trigger:** `pull_request`
- Runs on every push to a pull request.
- Must pass before a PR can be added to the merge queue.
- Configure this as a required status check in the branch protection rule for `main`.

### Check B (`check-b.yml`)
- **Trigger:** `merge_group`
- Runs when a PR enters the merge queue.
- Must pass before the PR is merged into the main branch.
- Configure this as a required status check in the merge queue rule for `main`.

## Branch Protection Setup

To enforce this flow, configure the following in **Settings → Branches → Branch protection rules** for `main`:

1. **Require status checks to pass before merging** — add `Check A` as a required check.
2. **Require merge queue** — enable the merge queue and add `Check B` as a required check for the queue.

With this setup:
- A PR cannot enter the merge queue until Check A passes.
- A PR cannot be merged from the queue into `main` until Check B passes.
