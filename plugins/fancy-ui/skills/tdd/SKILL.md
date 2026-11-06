---
name: tdd
description: Use when writing tests or setting up testing and CI for a Fancy UI app — test-driven development (TDD), test first, writing a failing test, a regression test for a bug fix, Pest (PHP / Laravel), vitest (TypeScript / React), browser tests, accessibility tests, testing Fancy UI components by their stable handles, and a lite CI workflow on GitHub Actions. Triggers on "TDD", "write tests", "test first", "failing test", "regression test", "Pest", "vitest", "browser tests", "accessibility tests", "a11y test", "CI", "GitHub Actions", "set up testing", "/tdd".
---

# Test-driven development for a Fancy UI app

Write the test before the code, keep the suite honest, and give the project one
small CI workflow that runs it on every change. This is about *the user's app*;
follow the project's existing conventions wherever they already exist.

## The loop

1. **Write the failing test first.** Describe the behaviour you want, not the
   implementation you plan.
2. **Watch it fail, for the right reason.** A test that fails on a typo, a
   missing import or a bad selector has not tested anything yet. Read the
   failure message and confirm it is the missing behaviour.
3. **Make it pass** with the smallest change that does.
4. **Refactor** with the test green, then run it again.

- **A bug fix starts with a test that reproduces the bug** and fails on the old
  code. Check that it does: run it before the fix, see red, then fix. A
  regression test that passes either way proves nothing.
- **Run the focused test while working** (`php artisan test --filter=...`,
  `npx vitest run path/to/file.test.tsx`), and **the whole suite before you call
  the work finished.**

## Stack

Use whatever runner the project already has. Don't add a second one alongside
it; match the existing setup instead.

- **PHP / Laravel: Pest** (Pest 5 recommended). Feature tests hit real routes,
  with `RefreshDatabase` for a clean database per test:

  ```php
  use App\Models\User;
  use Illuminate\Foundation\Testing\RefreshDatabase;

  uses(RefreshDatabase::class);

  it('creates a project', function () {
      $user = User::factory()->create();

      $this->actingAs($user)
          ->post('/projects', ['name' => 'Atlas'])
          ->assertRedirect();

      expect($user->projects()->where('name', 'Atlas')->exists())->toBeTrue();
  });
  ```

- **TypeScript / React: vitest + jsdom.** Render with `createRoot` inside
  `act`, or use Testing Library if the project already has it.
- **Browser tests: the Pest browser plugin** (`pestphp/pest-plugin-browser`,
  driven by Playwright). It needs Pest `^5.2.1`. Install with
  `composer require pestphp/pest-plugin-browser --dev`, then
  `npm install playwright@latest` and `npx playwright install`.

  ```php
  use App\Models\User;

  it('has an accessible, error-free dashboard', function () {
      $this->actingAs(User::factory()->create());

      visit('/dashboard')
          ->assertNoJavaScriptErrors()
          ->assertNoAccessibilityIssues();
  });
  ```

## Testing Fancy UI components

Fancy's interactive components are **controlled** (`value` + `onChange`, or
`checked` + `onCheckedChange` for toggles) and carry **stable handles**: an `id`
you pass in, `data-*` attributes, and ARIA roles with accessible names.

- **Select by those handles**, never by CSS paths (`div > div:nth-child(3)`) or
  visible text. Both break the moment the page is restyled or translated.
- **Assert on state and handles**: the value your `onChange` received, an
  `aria-checked`, a `data-state`. Agents drive the same surfaces through MCP
  bridges using the same handles, so a handle-based test also covers the
  agent's path through the UI.

```tsx
// @vitest-environment jsdom
import { act, useState } from "react";
import { createRoot } from "react-dom/client";
import { expect, it } from "vitest";
import { Switch } from "@particle-academy/react-fancy";

(globalThis as { IS_REACT_ACT_ENVIRONMENT?: boolean }).IS_REACT_ACT_ENVIRONMENT = true;

it("turns notifications on", () => {
  const seen: boolean[] = [];
  function Settings() {
    const [on, setOn] = useState(false);
    return (
      <Switch id="notify" label="Email me" checked={on}
        onCheckedChange={(v) => { seen.push(v); setOn(v); }} />
    );
  }

  const host = document.body.appendChild(document.createElement("div"));
  const root = createRoot(host);
  act(() => root.render(<Settings />));

  const toggle = document.getElementById("notify")!;
  expect(toggle.getAttribute("role")).toBe("switch");
  act(() => toggle.click());

  expect(seen).toEqual([true]);
  expect(toggle.getAttribute("aria-checked")).toBe("true");
  act(() => root.unmount());
});
```

If a component you need to test has no stable handle, give it one (`id`,
`data-*`, an accessible name) rather than reaching for a brittle selector.

## Checks that lie

A green tick is only worth something if the check could have gone red.

- **Never read a suite's result through a pipe.** `php artisan test | tail`
  reports the exit code of `tail`, not of the tests, so it can say 0 while tests
  fail. Redirect to a file and read the real exit code:

  ```bash
  php artisan test > test.log 2>&1; code=$?
  grep -E "FAILED|Tests:" test.log; echo "exit=$code"
  ```

- **Prove a listener works with one planted failure.** Before trusting
  `assertNoJavaScriptErrors()`, `assertNoAccessibilityIssues()` or a
  console-error spy, plant one real problem (a `console.error`, an unlabelled
  input), watch the test go red, then remove it.
- **A green run with zero tests is not a pass.** Make sure the project has a
  `test` script (`"test": "vitest run"` in `package.json`; `php artisan test` or
  a `test` script in `composer.json`). `npm test --if-present` with no script
  "passes" by doing nothing, and so does a filter that matches no tests. Check
  the test count in the output.

## Lite CI

One workflow, on every pull request and every push to `main`: install, test,
typecheck, build. For a Laravel + Inertia + React app, save this as
`.github/workflows/ci.yml`:

```yaml
name: CI

on:
  pull_request:
  push:
    branches: [main]

jobs:
  test:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v7

      - uses: shivammathur/setup-php@v2
        with:
          php-version: "8.4"
          coverage: none

      - uses: actions/setup-node@v7
        with:
          node-version: 22
          cache: npm

      - run: composer install --no-interaction --prefer-dist
      - run: npm ci

      # Build first: Inertia pages need the Vite manifest during feature tests.
      - run: npm run build
      - run: npx tsc --noEmit

      - run: cp .env.example .env && php artisan key:generate
      - run: php artisan test
```

- **PHP-only project:** drop the Node steps (`setup-node`, `npm ci`, build,
  `tsc`). **JS-only project:** drop the PHP steps and run `npm test` instead of
  `php artisan test`.
- Add `npm test` as its own step if the app has a vitest suite too.
- Browser tests need the browsers in CI: add `npx playwright install --with-deps`
  before the test step.
- Keep actions pinned to their major versions as above, and use no other
  third-party actions.
- **Make it a required check.** Protect `main` (Settings, then Rules or
  Branches) and require the `test` job, so a red run blocks the merge instead of
  being a suggestion.

## Where each concern lives
- *Make a surface agent-driveable (bridges, handles)* → `human-plus` skill
- *SSR and hydration problems a test turns up* → `ssr` skill
- *Upgrading Fancy packages, then re-running the suite* → `update-fancy` skill
