# OrbitKit Deploy

A GitHub Action that deploys your [OrbitKit](https://orbitkit.io) app site — privacy policies, support pages, data deletion pages, app site association files, and more — straight from your CI/CD pipeline.

## Usage

```yaml
- uses: OrbitKit-io/OrbitKit-Deploy@v1
  with:
    api-key: ${{ secrets.ORBITKIT_API_KEY }}
    app-id: ${{ vars.ORBITKIT_APP_ID }}
```

## Setup

1. **Get your API key** from the [OrbitKit Dashboard](https://orbitkit.io/dashboard) under Settings → API Keys
2. **Add it as a GitHub secret** in your repo: Settings → Secrets → Actions → `ORBITKIT_API_KEY`
3. **Add your app ID** as a variable: Settings → Variables → Actions → `ORBITKIT_APP_ID`

## Inputs

| Input | Required | Description |
|-------|----------|-------------|
| `api-key` | Yes | OrbitKit API key (`ok_xxxx`). Store as a [GitHub secret](https://docs.github.com/en/actions/security-guides/using-secrets-in-github-actions). |
| `app-id` | Yes | Your OrbitKit app ID |
| `api-endpoint` | No | API base URL (default: `https://api.orbitkit.io`) |

## Outputs

| Output | Description |
|--------|-------------|
| `site-url` | The deployed site URL |
| `deployed-at` | Deploy timestamp |

## Examples

### Deploy on push to main

```yaml
name: Deploy Privacy Policy
on:
  push:
    branches: [main]

jobs:
  deploy:
    runs-on: ubuntu-latest
    steps:
      - uses: OrbitKit-io/OrbitKit-Deploy@v1
        with:
          api-key: ${{ secrets.ORBITKIT_API_KEY }}
          app-id: ${{ vars.ORBITKIT_APP_ID }}
```

### Deploy when policy files change

```yaml
name: Deploy on Policy Update
on:
  push:
    branches: [main]
    paths:
      - 'privacy-policy.json'
      - 'support-page.json'

jobs:
  deploy:
    runs-on: ubuntu-latest
    steps:
      - uses: OrbitKit-io/OrbitKit-Deploy@v1
        id: deploy
        with:
          api-key: ${{ secrets.ORBITKIT_API_KEY }}
          app-id: ${{ vars.ORBITKIT_APP_ID }}

      - name: Print site URL
        run: echo "Deployed to ${{ steps.deploy.outputs.site-url }}"
```

### Deploy with the OrbitKit CLI

If you need to update policy data before deploying, use the [OrbitKit CLI](https://github.com/OrbitKit-io/OrbitKit-CLI) in an earlier step:

```yaml
jobs:
  deploy:
    runs-on: ubuntu-latest
    env:
      ORBITKIT_API_KEY: ${{ secrets.ORBITKIT_API_KEY }}
      ORBITKIT_APP_ID: ${{ vars.ORBITKIT_APP_ID }}
    steps:
      - uses: actions/checkout@v4

      - uses: actions/setup-node@v4
        with:
          node-version: 22

      - name: Update privacy policy
        run: npx orbitkit policy set privacy-policy.json

      - name: Deploy
        uses: OrbitKit-io/OrbitKit-Deploy@v1
        with:
          api-key: ${{ secrets.ORBITKIT_API_KEY }}
          app-id: ${{ vars.ORBITKIT_APP_ID }}
```

## Security

- **API key as secret**: Always store your API key as a [GitHub secret](https://docs.github.com/en/actions/security-guides/using-secrets-in-github-actions), never hardcode it in workflows
- **Secret scanning**: OrbitKit API keys use the `ok_` prefix, which is [auto-detected by GitHub secret scanning](https://docs.github.com/en/code-security/secret-scanning) if accidentally committed
- **No script injection**: All inputs are passed as environment variables, not interpolated in shell commands

## License

[Apache License 2.0](LICENSE) © OrbitKit, Inc.
