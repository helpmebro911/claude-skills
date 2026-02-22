# Playwright Configuration Guide

## When to Create a Config

Only create `playwright.config.ts` if the project doesn't already have one. If one exists, respect its settings and only suggest additions relevant to UI testing.

## Adapting the Template

Copy from [assets/playwright.config.template.ts](../assets/playwright.config.template.ts) and adjust:

### Base URL

Set `baseURL` to the project's local dev server:

| Framework | Typical URL |
|-----------|------------|
| Vite / React / Vue | `http://localhost:5173` |
| Next.js | `http://localhost:3000` |
| Nuxt | `http://localhost:3000` |
| SvelteKit | `http://localhost:5173` |
| Astro | `http://localhost:4321` |
| Angular | `http://localhost:4200` |

### Web Server

If the project uses a dev server, configure `webServer` to start it automatically:

```typescript
webServer: {
  command: 'npm run dev',
  url: 'http://localhost:5173',
  reuseExistingServer: !process.env.CI,
  timeout: 30000,
},
```

Check `package.json` scripts to find the correct dev command.

### Test Directory

Place UI tests in a dedicated directory to separate them from unit/integration tests:

```typescript
testDir: './tests/ui',
```

### Projects (Browsers)

For UI component testing, desktop Chrome is the minimum. Add mobile viewport for responsive checks:

```typescript
projects: [
  { name: 'desktop-chrome', use: { ...devices['Desktop Chrome'] } },
  { name: 'mobile-chrome', use: { ...devices['Pixel 5'] } },
],
```

Add more browsers only if the user requests cross-browser coverage:

```typescript
{ name: 'desktop-firefox', use: { ...devices['Desktop Firefox'] } },
{ name: 'desktop-safari', use: { ...devices['Desktop Safari'] } },
{ name: 'mobile-safari', use: { ...devices['iPhone 13'] } },
```

### Authentication

If the app requires login, use Playwright's `storageState` for authenticated tests:

```typescript
// In a setup project:
{ name: 'auth-setup', testMatch: /.*\.setup\.ts/ },

// In test projects:
{
  name: 'authenticated',
  use: { storageState: '.auth/user.json' },
  dependencies: ['auth-setup'],
},
```

Create a setup file at `tests/ui/auth.setup.ts`:

```typescript
import { test as setup, expect } from '@playwright/test';

setup('authenticate', async ({ page }) => {
  await page.goto('/login');
  await page.getByLabel('Email').fill(process.env.TEST_USER!);
  await page.getByLabel('Password').fill(process.env.TEST_PASS!);
  await page.getByRole('button', { name: /log in/i }).click();
  await expect(page).toHaveURL('/dashboard');
  await page.context().storageState({ path: '.auth/user.json' });
});
```

### CI Configuration

For CI environments, always run headless and use a single worker to avoid flakiness:

```typescript
use: {
  trace: process.env.CI ? 'on-first-retry' : 'off',
  screenshot: 'only-on-failure',
},
workers: process.env.CI ? 1 : undefined,
retries: process.env.CI ? 2 : 0,
```

## Common Settings to Check

| Setting | Purpose | Recommendation |
|---------|---------|----------------|
| `timeout` | Per-test timeout | 30s default; increase for slow apps |
| `expect.timeout` | Assertion timeout | 5s default; increase for lazy-loaded content |
| `fullyParallel` | Run tests in parallel | `true` if tests are independent |
| `forbidOnly` | Fail CI on `.only` | `true` in CI |
| `reporter` | Test output format | `html` for local, `list` + `html` for CI |
