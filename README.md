# fly

`dentarg/fly` is an [composite run steps action] that deploys your app to [Fly](https://fly.io).

The Fly Personal Access Token (`FLY_API_TOKEN`) needs to be saved as a [secret in GitHub Actions]. Use [`fly auth token`] to get a token or generate one from [the Fly dashboard].

If you pass `github-token`, the action will create [deployments] in your repository. When you use this, the workflow needs not to be triggered by the [`push` event] to work. See the example below.

The example workflow below deploys the app when the `CI` workflow ran succesfully against the default branch.

```yaml
name: Deploy

on:
  workflow_run:
    workflows: [CI]
    types: [completed]

permissions:
  contents: read
  deployments: write

jobs:
  deploy:
    if: |
      github.event.workflow_run.conclusion == 'success' &&
      github.event.workflow_run.head_branch == github.event.repository.default_branch
    concurrency: deploy
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      - run: |
          echo "RUBY_VERSION=$(cat .ruby-version)" >> $GITHUB_ENV
      - uses: dentarg/fly@v1
        with:
          build-args: "RUBY_VERSION=${{ env.RUBY_VERSION }}"
          fly-token: ${{ secrets.FLY_API_TOKEN }}
          github-token: ${{ secrets.GITHUB_TOKEN }}
```

[composite run steps action]: https://docs.github.com/en/free-pro-team@latest/actions/creating-actions/creating-a-composite-run-steps-action
[secret in GitHub Actions]: https://docs.github.com/en/actions/security-guides/encrypted-secrets
[`fly auth token`]: https://fly.io/docs/flyctl/auth-token/
[the Fly dashboard]: https://fly.io/user/personal_access_tokens
[deployments]: https://docs.github.com/en/rest/deployments/deployments
[`push` event]: https://docs.github.com/en/actions/using-workflows/events-that-trigger-workflows#push
