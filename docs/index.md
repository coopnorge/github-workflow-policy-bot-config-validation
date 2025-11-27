# github-workflow-policy-bot-config-validation

Validate policy-bot config.

!!! note "Mage workflows"

    Repositories using mage workflows don't need this as policy-bot config
    validation is already a part of mage as of v16.

## Goals

* Make sure that policy-bot config file is valid.

## Usage

### Inputs

```yaml
inputs:
  policy-bot-config-path:
    type: string
    default: .policy.yml
    required: false
    description: |
      Relative path of policy-bot config file. Defaults to `.policy.yml`.
secrets:
  policy-bot-server-url:
    required: true
    description: |
      Base url where policy bot is available. This is defined as a secret
      because GitHub requires secrets to be passed as secrets and does
      not allow passing it as a regular input.
```

This job can be added to the CI/CD workflow as follows:

```yaml
jobs:
  setup:
    name: Setup
    runs-on: ubuntu-latest
    permissions:
      contents: read
      pull-requests: read
    outputs:
      # ...
      # ...
      validate-policy-bot-config: ${{ steps.changes.outputs.policy-bot == 'true' }}
    steps:
      - uses: actions/checkout@v4
      - uses: dorny/paths-filter@de90cc6fb38fc0963ad72b210f1f284cd68cea36 # v3
        id: changes
        with:
          list-files: json
          filters: |
            # ...
            # ...
            policy-bot:
              - '.policy.yml'

  # <some other jobs>
  validate-policy-bot-config:
    name: Validate policy-bot Configuration
    needs: setup
    if: needs.setup.outputs.validate-policy-bot-config == 'true'
    uses: coopnorge/github-workflow-policy-bot-config-validation/.github/workflows/policy-bot-config-validation.yaml@v0
    permissions:
      contents: read
    secrets:
      policy-bot-server-url: ${{ secrets.POLICY_BOT_BASE_URL }}
  # <some other jobs>
```

It can be added as a required check as follows:

```yaml
  validate-ci-results:
    needs:
      - ...
      - ...
      - validate-policy-bot-config
    permissions: {}
    if: always()
    runs-on: ubuntu-24.04
    steps:
      - run: exit 1
        name: "Catch errors"
        if: ${{ contains(join(needs.*.result, ','), 'failure') || contains(join(needs.*.result, ','), 'cancelled') }}
```
