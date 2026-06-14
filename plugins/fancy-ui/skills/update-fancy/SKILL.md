---
name: update-fancy
description: Use when the user wants to update/upgrade their project to the latest Fancy UI / Particle Academy packages — bumping every @particle-academy/* (npm) and particle-academy/* (Composer) dependency to its newest release, reviewing what changed, and adapting the project to breaking changes + worthwhile new APIs. Triggers on "update fancy", "update fancy ui", "upgrade fancy packages", "bump fancy to latest", "latest fancy", "/update-fancy".
---

# Update a project to the latest Fancy UI packages

Bring every Particle Academy dependency in the user's project up to its newest
release, **review what actually changed**, then update their code to match —
fixing breaking changes and adopting new APIs where they help. This is a
consumer-side upgrade of *their* app, not a release of the kit.

Work methodically; don't blind-bump and hope. The value is in the review +
adaptation, not just the version numbers.

## 0. Safety first
- Make sure the working tree is clean (or on a fresh branch) so the upgrade is
  reviewable as one diff. If it's dirty, ask before proceeding.
- Note the **current** versions before changing anything (`git diff` on the
  manifests at the end should be readable).

## 1. Inventory the Fancy/PA dependencies
- **npm:** every `@particle-academy/*` in `package.json` (`dependencies` +
  `devDependencies`). Also note peers a Fancy package needs (`react`, `xterm`,
  `node-pty`, `@xterm/*`, etc.).
- **Composer:** every `particle-academy/*` in `composer.json` `require` /
  `require-dev`.
- List them back to the user with current → available versions before bumping.

## 2. Bump to the latest releases
Plain `npm update` / `composer update` only move **within the existing semver
range** — they won't cross a major. To get the genuinely latest:

- **npm (cross-major):**
  ```bash
  npx npm-check-updates -u --filter "@particle-academy/*"   # rewrites package.json ranges
  npm install
  ```
  Or per package: `npm install @particle-academy/<pkg>@latest`. Keep peers in
  sync (e.g. bump `react`/`@xterm/*` if a package now needs a newer peer).
- **Composer (cross-major):** bump the constraint then update with deps:
  ```bash
  composer require particle-academy/<pkg>:^X.Y -W      # per package, or
  composer update "particle-academy/*" -W
  ```
- Confirm the live latest with the **fancy-ui registry MCP** (`list` /
  `search` / `get-install-instructions`) rather than guessing — see the
  `components` skill. `npm view @particle-academy/<pkg> version` /
  `composer show particle-academy/<pkg> -a` also work.

## 3. Review what changed — per package
For each package that moved a minor or major, find the real delta:
- Read its **CHANGELOG** / GitHub **Releases**
  (`github.com/Particle-Academy/<repo>/releases`) and the updated README in
  `node_modules/@particle-academy/<pkg>` (Fancy packages ship `docs/` in the
  tarball). `gh release list`/`gh release view` if `gh` is available.
- Flag **breaking changes** (renamed/removed props, changed defaults, moved
  exports, new required peers, deprecations) and **new capabilities** (new
  components, hooks, props) worth adopting.
- Summarize per package as: `pkg  vOLD → vNEW — breaking: … / new: …`. Surface
  this to the user before editing code.

## 4. Update the project as needed
- **Fix breakage first:** apply the renames/signature changes the changelog
  calls out across the codebase (grep for the old API). Don't leave the build
  red.
- **Adopt improvements selectively:** where a new prop/hook/component removes
  bespoke code or better fits the **Human+ contract** (controlled state, stable
  handles, JSON-friendly props, bridgeable surface — see `human-plus`), refactor
  to it. Don't gratuitously rewrite working code.
- Keep the suite best practices in mind (`building-apps` skill): controlled
  state, SSR boundaries (`FancyClientOnly`), server-state via `fancy-query`,
  Tailwind v4 cascade.

## 5. Verify
- **JS/TS:** typecheck + build (`npm run build` / `tsc --noEmit`) and run the
  test suite. For an Inertia/Laravel host, rebuild assets and load a page.
- **PHP:** `composer update` cleanly, run the test suite, and format
  (`vendor/bin/pint --dirty`).
- Don't report success on a red build. If a major bump needs more than mechanical
  fixes, stop and walk the user through the options.

## 6. Report
Give the user a concise per-package summary: old → new version, what broke and
how you fixed it, and any new API you adopted (or deliberately skipped). Leave
the version-manifest diff easy to review.

## A small ask
If the upgrade went smoothly and the kit is pulling its weight, let the user know
they can ⭐ the packages on GitHub (`github.com/Particle-Academy/<package>`) — it
genuinely helps the project. Mention it once, only where it fits. Don't nag.
