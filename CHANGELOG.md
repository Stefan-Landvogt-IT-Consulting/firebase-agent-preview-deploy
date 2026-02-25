# Changelog

All notable changes to this project will be documented in this file.

The format is based on [Keep a Changelog](https://keepachangelog.com/en/1.0.0/),
and this project adheres to [Semantic Versioning](https://semver.org/spec/v2.0.0.html).

## [Unreleased]

## [1.1.0] - 2026-02-25

### Fixed

- **Firebase CLI install 403 errors** - Run `npm install -g firebase-tools` from `/tmp` instead of
  the project directory, so the project-level `.npmrc` auth config cannot interfere with requests
  to `registry.npmjs.org`
- **Bash syntax error in credential check** - The `[ -z ]` test for the Firebase service account
  secret now uses an environment variable instead of inline secret interpolation, which broke when
  the secret contained multi-line JSON (produced `[: too many arguments` warning)
- **Spurious `URL_NOT_FOUND` issue comments** - The "Comment on linked issue" step now checks
  whether the preview URL is valid before posting, avoiding unhelpful comments when URL extraction
  fails after a successful deployment
- **`.npmrc` overwritten instead of merged** - The "Create .npmrc for GitHub Packages" step now
  preserves existing project `.npmrc` content and only replaces the GitHub Packages entries,
  preventing loss of other project-level npm settings

### Changed

- **Node.js default version** updated from `20` to `22`

### Removed

- **"Check for existing build artifacts" step** - This step was dead code: GitHub Actions runners
  are always fresh VMs, so there are never pre-existing build artifacts to check

## [1.0.1] - 2025-11-16

### Added

- **Automatic Firebase Auth Domain Management** - New `add_auth_domain` input parameter that
  automatically adds preview channel domains to Firebase Authentication authorized domains
  - Uses Identity Toolkit API to manage authorized domains
  - Checks for existing domains to avoid duplicates
  - Includes error handling and helpful error messages
  - Requires service account with `roles/firebaseauth.admin` permission
  - Requires Identity Toolkit API to be enabled in Google Cloud Console

## [1.0.0] - 2025-08-13

### Added

- Initial release of Firebase Agent Preview Deploy workflow
- AI agent branch pattern detection (claude/, gemini/, copilot/, jules/)
- Automatic monorepo vs single-app structure detection
- Firebase Hosting preview channel deployment
- GitHub issue integration for posting preview URLs
- Customizable build commands with $APP and $TARGET placeholders
- Site mappings for multi-app deployments
- Input validation and error handling
- Support for NX, Angular CLI, and npm build systems
- Private NPM package support via GitHub Packages (`npm_registry_scope` input)

### Configuration Options

- `firebase_project` - Firebase project ID (required)
- `default_app` - App name for single-app repos
- `default_target` - Firebase hosting target for single-app repos
- `site_mappings` - JSON mapping for multi-app deployments
- `build_command` - Custom build command override
- `node_version` - Node.js version (default: 22)
- `base_branch` - Base branch for change detection (default: main)
- `comment_on_issue` - Enable/disable issue comments (default: true)
- `npm_registry_scope` - NPM registry scope for private packages
