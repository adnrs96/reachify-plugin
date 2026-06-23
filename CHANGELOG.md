# Changelog

All notable changes to **reachify-plugin** are documented here.

The format is based on [Keep a Changelog](https://keepachangelog.com/en/1.1.0/),
and this project adheres to [Semantic Versioning](https://semver.org/spec/v2.0.0.html).

> Releasing: rename `[Unreleased]` to the new version with today's date and bump
> `version` in `.claude-plugin/plugin.json` to match, then `claude plugin tag --push`.

## [Unreleased]

### Added
- `userConfig` prompts for `api_key` and `api_token` at enable time; both are
  `sensitive` (stored in the OS keychain, never `settings.json`) and `required`.
- `SessionStart` hook (`hooks/hooks.json`) that runs `reachify login` automatically
  each session, using the configured key as the id and token as the token.
- `allowed-tools` on the worker skill so it runs the bundled `reachify` CLI
  (`Bash(reachify *)`) and reads and writes the per-job working dir
  (`Read(/tmp/.reachify/**)`, `Write(/tmp/.reachify/**)`) without per-call
  permission prompts.
- `description` field on the marketplace manifest.

### Changed
- **Breaking:** renamed the plugin from `reachify` to `reachify-plugin`. The install
  id is now `reachify-plugin@reachify` and the skill is invoked as
  `/reachify-plugin:reachify`.
- Skill troubleshooting now notes that login runs automatically via the
  `SessionStart` hook.

## [0.0.2]

### Added
- Reachify judgement-job worker plugin: the `reachify` worker-loop skill, the
  bundled `reachify` CLI under `bin/` (with a platform-dispatch wrapper), and the
  plugin + marketplace manifests.
