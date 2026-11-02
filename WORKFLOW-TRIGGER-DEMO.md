# Pull request workflow trigger demo

## Project

This repository contains a small Node.js calculator with two tests. The workflow runs those tests, so the example demonstrates a real project check rather than a trigger-only workflow. Run the same check locally with `npm test` (Node.js 20 or later; no dependencies to install).

## Reproduce the workflow issue

The active [workflow.yml](.github/workflows/workflow.yml) has this trigger:

```yaml
on:
  pull_request:
    branches:
      - main
    types:
      - opened
```

The workflow runs when a pull request targeting `main` is first opened. It does **not** run again when new commits are pushed to that open pull request, because GitHub reports that activity as `synchronize`. As a result, the check shown on the pull request can be stale and does not validate the latest proposed changes.

## Correct the trigger

Include the pull-request activity types that should start checks:

```yaml
on:
  pull_request:
    branches:
      - main
    types:
      - opened
      - synchronize
      - reopened
```

To apply the fix, add `synchronize` and `reopened` under `types` in `.github/workflows/workflow.yml`. The `branches` filter selects the pull request's **target/base** branch; change `main` if the repository uses a different default branch. The rest of the workflow can stay the same.

## Validate in GitHub

The workflow definition needs to be on the pull request's base branch for this demonstration. Start with the intentionally broken `opened`-only version on `main`; after observing the missed run, put the corrected version on `main` before testing another push to the open pull request.

### Publish the demo safely

Use the empty demo repository you created. Configure a demo author locally before making commits; do not use a personal email:

```powershell
git init -b main
git config user.name "Workflow Demo"
git config user.email "YOUR_GITHUB_NOREPLY_ADDRESS"
git add .
git commit -m "Add pull request trigger demo"
git remote add origin https://github.com/OWNER/REPOSITORY.git
git push -u origin main
```

Replace the email with the exact address ending in `users.noreply.github.com` from the demo account's GitHub **Settings → Emails** page. Replace the remote URL with the HTTPS URL for the empty repository. Check the commit author before pushing with `git log -1 --format="%an <%ae>"`. If GitHub asks to authenticate, use the normal Git Credential Manager/browser flow; never put a password or token in the remote URL or in this guide.

### Reproduce the missed run

1. Confirm the `opened`-only workflow is present on `main` and the initial push succeeded.
2. Create a feature branch and make a harmless initial change so it differs from `main` and GitHub will allow a pull request:

   ```powershell
   git switch -c demo/stale-check
   Set-Content -Path demo-pr-note.txt -Value "Initial change for the pull request trigger demo."
   git add demo-pr-note.txt
   git commit -m "Add initial demo change"
   git push -u origin demo/stale-check
   ```

3. Open a pull request from `demo/stale-check` into `main`. The `opened` event should start `npm test`; confirm it passes in the Actions tab.
4. On the feature branch, temporarily change `subtract` in `src/calculator.js` to return `left + right` instead of `left - right`. Commit and push that change:

   ```powershell
   git add src/calculator.js
   git commit -m "Demonstrate stale pull request check"
   git push
   ```

5. Revisit the open pull request. With the broken trigger, no new run starts, even though the latest commit makes a test fail. The earlier passing check is stale.

### Apply and verify the fix

1. Put the corrected trigger on `main`, either with a direct demo-repository commit or by opening and merging a small workflow-fix pull request. Under `pull_request.types`, include `opened`, `synchronize`, and `reopened`.
2. Restore `subtract` to `return left - right` on the feature branch and remove the temporary marker file, then commit and push:

   ```powershell
   git add src/calculator.js
   git rm demo-pr-note.txt
   git commit -m "Restore calculator behavior"
   git push
   ```

3. Confirm that this new push starts another `npm test` run and that it passes.
4. Optionally close and reopen the pull request to confirm `reopened` starts a run. Editing its title or labels should not start a run, and a pull request targeting a branch other than `main` should not match this workflow.

If `main` is protected, use a workflow-fix pull request and merge it before step 2. Once finished, delete the demo branches or repository if you no longer need them.

| Test action | Before fix (`opened` only) | After fix |
|---|---|---|
| Open a pull request targeting `main` | Run starts | Run starts |
| Push a commit to the open pull request | No new run | New run starts |
| Reopen the pull request | No new run | New run starts |
| Edit the title or add a label | No run | No run |
| Open a pull request targeting another branch | No run | No run |

For a live demonstration, first verify the current `opened`-only behavior, then add the two missing event types and push the workflow change. Push another commit to the pull request and verify a new test run starts against its latest revision. Reopening the pull request can be demonstrated as an additional check. Use a demo repository or branch, not a production workflow.

## Suggested 10–15 minute presentation

1. **Context (2 min):** Introduce the calculator project and explain why checks should validate the latest pull-request changes.
2. **Reproduce the bug (3–4 min):** Open a pull request targeting `main`, show the passing test run, then push another commit and show that no new run starts.
3. **Explain the cause (2 min):** Explain that pull-request activity types are explicit when `types` is specified; a new commit is `synchronize`, not `opened`.
4. **Apply the fix (2 min):** Add `synchronize` and `reopened`; explain that the branch filter concerns the target branch.
5. **Validate (3–4 min):** Push another commit and verify the new run; optionally reopen the PR and confirm the title/label and non-`main` cases do not trigger.
6. **Wrap up (1 min):** Summarize how choosing the right event types keeps required checks current without running on every PR activity.
