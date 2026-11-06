# Changelog

All notable changes to this plugin are documented here. The format follows
[Keep a Changelog](https://keepachangelog.com/en/1.1.0/), and the plugin uses
[Semantic Versioning](https://semver.org/spec/v2.0.0.html). It is pre-1.0, so a
breaking change may land in a minor release; any that does is marked here.

This file starts at 0.6.6. Earlier releases are recorded in the git history.

## [Unreleased]

## [0.6.7] - 2026-11-06

### Fixed
- `/tdd`: the Pest browser plugin (v5.2.0) records only `console.log` and uncaught
  errors, so `console.error` and `console.warn` pass `assertNoConsoleLogs()` and
  `assertNoJavaScriptErrors()`. The skill now says so and shows the test-only
  forwarder that catches them. Checked against the plugin's source; reported by a
  consumer, PS Mapper.

## [0.6.6] - 2026-11-05

### Added
- `tdd` skill (`/tdd`): test-driven development for an app built on Fancy UI.
  Covers the test-first loop (including a bug fix that starts with a test that
  fails on the old code), Pest 5, vitest + jsdom and the Pest browser plugin,
  testing Fancy components by their stable handles rather than CSS paths or
  visible text, checks that report green without checking, and a lite GitHub
  Actions workflow for a Laravel + Inertia + React app.

### Changed
- The Codex manifest (`plugins/fancy-ui/.codex-plugin/plugin.json`) moves from
  0.6.4 to 0.6.6, back in step with the Claude Code manifest, which 0.6.5 bumped
  alone. Nothing to do.
