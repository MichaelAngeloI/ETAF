# Copilot instructions for this repository

This workspace contains a Playwright-based browser smoke test suite for storefront locator flows. Most of the active code lives under `playwright-locator-smoke-test/`.

## Working directory and project layout

Run commands from the project folder unless you are intentionally editing repo-level CI files:

```bash
cd playwright-locator-smoke-test
```

Key areas:

- `playwright.config.js` – Playwright runtime config (browser project, timeouts, reporters, permissions)
- `fixtures/fixtures.js` – custom `test` fixture that wires in page objects and helpers
- `pages/nuanceUS/` – page-object classes (`BasePage`, `StoreLocatorPage`)
- `utils/webUtils.js` – shared browser utilities (cookies, geolocation, map drag, random pin selection)
- `utils/ValidationHelper.js` – reusable validation helpers for page loads, iframe checks, API responses, search results
- `data/config.json` – site/country/city matrix used by the tests
- `tests/` – the actual Playwright specs

## Build, test, and validation commands

Install dependencies:

```bash
cd playwright-locator-smoke-test
npm ci
npx playwright install --with-deps
```

Default test commands:

```bash
npm test
npm run test:headed
npm run test:debug
npm run test:ui
npm run report
```

Single-test execution (preferred when debugging a failure):

```bash
cd playwright-locator-smoke-test
npx playwright test tests/smoke-test.spec.js --project=chromium
npx playwright test tests/smoke-test.spec.js -g "form interaction test"
```

CI is configured to run the suite with GitHub Actions (`.github/workflows/playwright.yml`) using:

```bash
npm ci
npx playwright install --with-deps
npx playwright test
```

There is no dedicated lint script in this repo; browser validation is done via Playwright test execution and reporters (`html` + `allure-playwright`).

## High-level architecture

This project is not a traditional app codebase; it is a browser-automation suite that exercises live storefront locator pages across multiple geographies. The big-picture flow is:

1. A spec in `tests/` loads a locale/site URL from `data/config.json`.
2. The custom fixture in `fixtures/fixtures.js` injects `BasePage`, `StoreLocatorPage`, `testHelpers`, and `validationHelper` into each test.
3. Page-object classes encapsulate DOM interaction details instead of scattering selectors throughout the specs.
4. `webUtils.js` centralizes retries, random-city selection, map interactions, geolocation denial, and cookie handling. This is the main place to add cross-cutting browser behavior.
5. `ValidationHelper.js` checks page loads, visible UI, and network responses so tests assert app state instead of only clicking buttons.

The suite is intentionally site-aware and selector-heavy: many flows depend on live vendor markup, region-specific URLs, and locale-specific map controls.

## Key conventions

- Prefer the shared `test` fixture from `fixtures/fixtures.js` instead of creating ad hoc page objects in individual specs.
- Keep DOM selectors inside page-object classes (`pages/nuanceUS/*.js`) and avoid sprinkling raw CSS selectors across test files.
- Use the `data/config.json` site matrix for URLs and city lists rather than hard-coding locale-specific values in each test.
- Use Playwright waits (`waitFor`, explicit `timeout`) around locator and map interactions; this suite is sensitive to async UI timing and lazy-loaded map content.
- When working with global browser setup, prefer `webUtils` helpers (`handleCookieConsent`, `setupGeolocationDenial`, `mapDrag`, `randomPin`) over copy-pasted logic in tests.
- `playwright.config.js` is configured for Chromium by default and intentionally disables the other browsers in the active project config.
- Existing tests often use `logger.step(...)` or `webUtils.step(...)` for traceable debugging output; keep that convention when adding new steps.
- Follow the repository’s pattern of using `page.goto(...)`, `locator.waitFor(...)`, and `testInfo`/screenshots for debugging when a test fails.

## When making changes

- Treat `playwright-locator-smoke-test` as the source of truth for runtime behavior and tests.
- Preserve the site-specific selectors and helper layering; these are important to the project’s live-website automation model.
- If you add a new test flow, mirror the structure used by the existing specs and page objects instead of inventing a separate pattern.
- Changes to dependencies should be reflected in `package.json` and validated by rerunning the relevant Playwright test(s).
