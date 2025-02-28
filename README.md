# About

Github action to rollout on Bytebase.

- Tutorial:
  [Database Release CI/CD with GitHub Actions](http://bytebase.com/docs/tutorials/github-release-cicd-workflow/)
- Sample repo: https://github.com/bytebase/release-cicd-workflows-example

## Inputs

| Input Name     | Description                                                                                                                                                                                                                                                                                    | Required | Default |
| -------------- | ---------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- | -------- | ------- |
| `url`          | The bytebase URL.                                                                                                                                                                                                                                                                              | Yes      |         |
| `token`        | The Bytebase access token.                                                                                                                                                                                                                                                                     | Yes      |         |
| `plan`         | The plan to create the rollout from. Format: `projects/{project}/plans/{plan}`                                                                                                                                                                                                                 | Yes      |         |
| `target-stage` | Bytebase rollout pipeline can contain multiple stages. This action will exit after complete deploying the `target-stage` stage. `target-stage` is the stage title in the deployment config in the project setting. If not provided, this action will rollout all stages. Example: `Test Stage` | No       |         |

## Example

```yaml
on:
  push:
    branches:
      - main

jobs:
  bytebase-cicd:
    runs-on: ubuntu-latest
    env:
      BYTEBASE_URL: 'https://demo.bytebase.com'
      BYTEBASE_PROJECT: 'projects/example'
      BYTEBASE_SERVICE_ACCOUNT: 'demo@service.bytebase.com'
      BYTEBASE_TARGETS: 'instances/mysql1/databases/db1,instances/mysql1/databases/db2'
    name: Bytebase cicd
    steps:
      - name: Checkout
        uses: actions/checkout@v4
      - name: Login to Bytebase
        id: login
        uses: bytebase/login-action@main
        with:
          url: ${{ env.BYTEBASE_URL }}
          service-account: ${{ env.BYTEBASE_SERVICE_ACCOUNT }}
          service-account-key: ${{ secrets.BYTEBASE_PASSWORD }}
      - name: Create release
        id: create_release
        uses: bytebase/create-release-action@main
        with:
          url: ${{ env.BYTEBASE_URL }}
          token: ${{ steps.login.outputs.token }}
          project: ${{ env.BYTEBASE_PROJECT }}
          file-pattern: 'migrations/*.sql'
      - name: Create plan
        id: create_plan
        uses: bytebase/create-plan-from-release-action@main
        with:
          url: ${{ env.BYTEBASE_URL }}
          token: ${{ steps.login.outputs.token }}
          project: ${{ env.BYTEBASE_PROJECT }}
          release: ${{ steps.create_release.outputs.release }}
          targets: ${{ env.BYTEBASE_TARGETS }}
      - name: rollout
        id: rollout
        uses: bytebase/rollout-action@main
        with:
          url: ${{ env.BYTEBASE_URL }}
          token: ${{ steps.login.outputs.token }}
          plan: ${{ steps.create_plan.outputs.plan }}
```
