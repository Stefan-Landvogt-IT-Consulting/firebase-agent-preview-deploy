# Firebase Agent Preview Deploy

Reusable GitHub Actions workflow for automatically deploying Firebase Hosting previews when AI agents (Claude, Gemini, etc.) push to specific branch patterns.

## Features

- 🤖 **AI Agent Integration** - Triggers on agent branch patterns
- 🏗️ **Auto-Detection** - Detects monorepo vs single-app structure
- 🔧 **Simple Configuration** - Just a few inputs required
- 📝 **Issue Integration** - Posts preview URLs to linked GitHub issues
- 🚀 **Firebase Hosting** - Deploys to preview channels
- 🔐 **Auth Domain Management** - Automatically adds preview domains to Firebase Auth
- ♻️ **Reusable** - Use across multiple repositories

## Quick Setup

1. **Create a workflow** in your repository at `.github/workflows/agent-preview.yml`:

```yaml
name: AI Agent Preview Deploy

on:
  push:
    branches:
      - "claude/**"
      - "gemini/**"
      - "copilot/**"
      - "jules/**"

jobs:
  deploy-preview:
    permissions:
      contents: read
      pull-requests: write
      issues: write
      id-token: write
    uses: Stefan-Landvogt-IT-Consulting/firebase-agent-preview-deploy/.github/workflows/agent-preview-deploy.yml@main
    with:
      firebase_project: "your-firebase-project-id"
      # For single-app repos (uncomment if needed):
      # default_app: 'web'
      # default_target: 'web'
      # For multi-app repos with different Firebase sites (optional):
      # site_mappings: '{"web-app":"my-project-web","admin-app":"my-project-admin"}'
      # Auto-add preview domains to Firebase Auth (requires additional permissions):
      add_auth_domain: true
    secrets:
      firebase_service_account: ${{ secrets.FIREBASE_SERVICE_ACCOUNT }}
```

2. **Add your Firebase service account** as a repository secret:

   - Go to your repo settings > Secrets and variables > Actions
   - Add `FIREBASE_SERVICE_ACCOUNT` with your service account JSON

3. **Done!** Push to an agent branch like `claude/issue-123-fix` and watch it deploy.

> **💡 Tip:** Use `@main` for the latest features. Tagged releases coming soon!

## Configuration

All configuration is done via **workflow inputs** when calling the reusable workflow. See the inputs section below for available options.

### Available Inputs

| Input              | Description                                     | Required        | Default |
| ------------------ | ----------------------------------------------- | --------------- | ------- |
| `firebase_project` | Firebase project ID                             | Yes             | -       |
| `default_app`      | App name (for single-app repos)                 | Single-app only | -       |
| `default_target`   | Firebase hosting target (for single-app repos)  | Single-app only | -       |
| `site_mappings`    | JSON mapping NX app names to Firebase site IDs  | No              | -       |
| `build_command`    | Custom build command (overrides auto-detection) | No              | -       |
| `node_version`     | Node.js version                                 | No              | `22`    |
| `base_branch`      | Base branch for change detection                | No              | `main`  |
| `add_auth_domain`  | Add preview URL domain to Firebase Auth         | No              | `false` |
| `npm_registry_scope` | NPM scope for private GitHub Packages (e.g. `@my-org`) | No    | -       |

### Secrets

| Secret                    | Description                                      | Required |
| ------------------------- | ------------------------------------------------ | -------- |
| `firebase_service_account` | Firebase service account JSON key               | Yes      |
| `github_npm_token`        | GitHub PAT with `read:packages` for private npm  | No       |


### Multi-App Site Mapping

For monorepos with apps deployed to different Firebase sites:

```yaml
site_mappings: '{"web-app":"my-project-web","admin-app":"my-project-admin","mobile-app":"my-project-mobile"}'
```

### Custom Build Commands

Override auto-detection:

```yaml
build_command: "npm run build:preview"
```

The build command can use placeholders:

- `$APP` - replaced with the detected app name
- `$TARGET` - replaced with the target name

### Automatic Auth Domain Management

When using Firebase Authentication, preview channel URLs must be added to the authorized domains list. Enable automatic domain management:

```yaml
add_auth_domain: true
```

**Requirements:**

1. **Enable Identity Toolkit API** in Google Cloud Console
2. **Service account permissions** - Add `roles/firebaseauth.admin` role to your service account
3. **Domain limit** - Firebase has a maximum of ~36 authorized domains

The workflow will automatically:

- Extract the domain from the deployed preview URL
- Check if the domain is already authorized
- Add it to the authorized domains list if not present

**Example usage:**

```yaml
uses: Stefan-Landvogt-IT-Consulting/firebase-agent-preview-deploy/.github/workflows/agent-preview-deploy.yml@main
with:
  firebase_project: "your-project"
  add_auth_domain: true # Enable automatic auth domain management
secrets:
  firebase_service_account: ${{ secrets.FIREBASE_SERVICE_ACCOUNT }}
```

## How It Works

1. **Branch Detection** - Extracts agent name from branch (e.g., `claude/issue-123-fix`)
2. **App Detection** - Determines which app changed:
   - Monorepo: Analyzes changes in `apps/` subdirectories
   - Single-app: Uses `DEFAULT_APP`
3. **Build & Deploy** - Builds the app and deploys to Firebase preview channel
4. **Issue Comment** - Posts preview URL to linked GitHub issue

## Branch Naming Convention

Expected pattern: `{agent}/issue-{number}-{description}`

Examples:

- `claude/issue-123-fix-header`
- `gemini/issue-456-new-feature`

## Supported Project Structures

### Monorepo

```
apps/
  web/
  admin/
  mobile/
```

### Single App

```
src/
public/
package.json
```

### Build System Support

Auto-detects:

- **NX** - `nx.json` present
- **Angular CLI** - `angular.json` present
- **npm scripts** - `build` script in `package.json`

## Firebase Setup

1. Create a service account with Firebase Hosting permissions
2. Download the JSON key file
3. Add as `FIREBASE_SERVICE_ACCOUNT` secret
4. Ensure your `firebase.json` is configured for hosting
5. **Add preview domains to Firebase Auth** (if using Firebase Authentication):
   - **Option A: Automatic** - Set `add_auth_domain: true` in your workflow (requires additional permissions, see [Automatic Auth Domain Management](#automatic-auth-domain-management))
   - **Option B: Manual** - Go to Firebase Console > Authentication > Settings > Authorized Domains
     - Add your preview channel domains (format: `{project-id}--{channel-id}-{site-id}.web.app`)
     - Example: `my-project--claude-web-preview-my-site.web.app`

## Troubleshooting

### "No app detected" error

- For single-app repos: Set `DEFAULT_APP` and `DEFAULT_TARGET` variables
- For monorepos: Ensure changes are in `apps/` subdirectories

### Build failures

- Check your build command with `BUILD_COMMAND` variable
- Verify dependencies install correctly

### Firebase deployment issues

- Verify service account has proper permissions
- Check `firebase.json` configuration
- Ensure hosting targets match your configuration

### Authentication issues on preview URLs

- Add preview channel domains to Firebase Console > Authentication > Authorized Domains
- Preview domains follow the pattern: `{project-id}--{channel-id}-{site-id}.web.app`
- You may need to add wildcard patterns like `{project-id}--*-{site-id}.web.app`
- Or enable automatic domain management with `add_auth_domain: true`

### Automatic auth domain management failures

- **"Identity Toolkit API not enabled"** - Enable the API in Google Cloud Console:
  1. Go to [Google Cloud Console](https://console.cloud.google.com/)
  2. Select your project
  3. Navigate to APIs & Services > Library
  4. Search for "Identity Toolkit API" and enable it
- **"Permission denied"** - Add `roles/firebaseauth.admin` role to your service account:
  1. Go to IAM & Admin > IAM
  2. Find your service account
  3. Add the "Firebase Authentication Admin" role
- **"Too many domains"** - Firebase has a limit of ~36 authorized domains. Remove unused domains from Firebase Console > Authentication > Authorized Domains

## Recent Changes

See [CHANGELOG.md](CHANGELOG.md) for full history.

### Latest Updates (v1.1.0 - 2026-02-25)

- **Bug Fix** - Firebase CLI install no longer fails with 403 on runners with project `.npmrc` auth
- **Bug Fix** - Credential check now handles multi-line JSON service account secrets correctly
- **Bug Fix** - Issue comments skipped when preview URL is unavailable
- **Bug Fix** - `.npmrc` is merged instead of overwritten, preserving project settings
- **Changed** - Node.js default version updated from `20` to `22`

## Contributing

Contributions are welcome! Please feel free to submit a Pull Request.

## Support

If you encounter issues or have questions:

1. Check the [troubleshooting section](#troubleshooting)
2. Search [existing issues](https://github.com/Stefan-Landvogt-IT-Consulting/firebase-agent-preview-deploy/issues)
3. Open a new issue with details about your setup

## License

MIT License - see [LICENSE](LICENSE) file for details.

## Author

Created by [Stefan Landvogt IT Consulting](https://github.com/Stefan-Landvogt-IT-Consulting)
