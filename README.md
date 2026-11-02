# Pull request trigger demo

A tiny Node.js calculator project used to demonstrate a GitHub Actions pull-request trigger bug.

## Run locally

Requires Node.js 20 or later. No third-party packages are needed.

```sh
npm test
```

## Demonstrate the workflow issue

The workflow in [`.github/workflows/workflow.yml`](.github/workflows/workflow.yml) runs tests when a pull request targeting `main` is opened. Its trigger intentionally omits `synchronize`, so pushing another commit to an already-open pull request does not start a fresh test run.

See [WORKFLOW-TRIGGER-DEMO.md](WORKFLOW-TRIGGER-DEMO.md) for the corrected trigger, validation steps, and a 10–15 minute presentation outline. After the demonstration, add `synchronize` and `reopened` to the workflow's `types` list.
