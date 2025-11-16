# Changelog

All notable changes to this project will be documented in this file.

The format is based on [Keep a Changelog](https://keepachangelog.com/en/1.0.0/),
and this project adheres to [Semantic Versioning](https://semver.org/spec/v2.0.0.html).

## [Unreleased]

### Added

- **Automatic Firebase Auth Domain Management** - New `add_auth_domain` input parameter that automatically adds preview channel domains to Firebase Authentication authorized domains
  - Uses Identity Toolkit API to manage authorized domains
  - Checks for existing domains to avoid duplicates
  - Includes error handling and helpful error messages
  - Requires service account with `roles/firebaseauth.admin` permission
  - Requires Identity Toolkit API to be enabled in Google Cloud Console
- **NPM Registry Scope Support** - New `npm_registry_scope` input for private package access via GitHub Packages (65a12d0, 2025-10-21)

### Fixed

- Fixed typo in workflow (9fe061f, 2025-10-21)

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
- Private NPM package support via GitHub Packages

### Configuration Options

- `firebase_project` - Firebase project ID (required)
- `default_app` - App name for single-app repos
- `default_target` - Firebase hosting target for single-app repos
- `site_mappings` - JSON mapping for multi-app deployments
- `build_command` - Custom build command override
- `node_version` - Node.js version (default: 20)
- `base_branch` - Base branch for change detection (default: main)
- `comment_on_issue` - Enable/disable issue comments (default: true)
- `npm_registry_scope` - NPM registry scope for private packages
