# Playwright Test Patterns for UI Verification

Assertion patterns for component visibility, layout, timing, and interaction testing. Import from `@playwright/test`.

## Component Visibility

```typescript
import { test, expect } from '@playwright/test';

test.describe('Component Visibility', () => {
  test('essential elements are visible on load', async ({ page }) => {
    await page.goto('/');

    // Header and navigation
    await expect(page.locator('header')).toBeVisible();
    await expect(page.getByRole('navigation')).toBeVisible();

    // Main content area
    await expect(page.getByRole('main')).toBeVisible();

    // Specific components by role, text, or test-id
    await expect(page.getByRole('heading', { level: 1 })).toBeVisible();
    await expect(page.getByRole('button', { name: /get started/i })).toBeVisible();

    // Footer
    await expect(page.locator('footer')).toBeVisible();
  });

  test('element is hidden until interaction', async ({ page }) => {
    await page.goto('/');

    // Modal not visible initially
    await expect(page.getByRole('dialog')).not.toBeVisible();

    // Trigger the modal
    await page.getByRole('button', { name: /open menu/i }).click();

    // Now visible
    await expect(page.getByRole('dialog')).toBeVisible();
  });
});
```

**Selector priority** (most to least reliable):

1. `getByRole()` — semantic, resilient to markup changes
2. `getByText()` / `getByLabel()` — content-based
3. `getByTestId()` — explicit test hooks
4. `locator('css-selector')` — last resort

## Element Ordering and Layout

```typescript
test('hero appears above features section', async ({ page }) => {
  await page.goto('/');

  const hero = page.locator('[data-testid="hero"]');
  const features = page.locator('[data-testid="features"]');

  const heroBox = await hero.boundingBox();
  const featuresBox = await features.boundingBox();

  expect(heroBox).not.toBeNull();
  expect(featuresBox).not.toBeNull();
  expect(heroBox!.y).toBeLessThan(featuresBox!.y);
});

test('CTA button is above the fold', async ({ page }) => {
  await page.goto('/');

  const cta = page.getByRole('button', { name: /get started/i });
  const ctaBox = await cta.boundingBox();
  const viewport = page.viewportSize();

  expect(ctaBox).not.toBeNull();
  expect(viewport).not.toBeNull();
  // CTA fully visible without scrolling
  expect(ctaBox!.y + ctaBox!.height).toBeLessThan(viewport!.height);
});

test('sidebar is to the left of main content', async ({ page }) => {
  await page.goto('/dashboard');

  const sidebar = page.locator('[data-testid="sidebar"]');
  const main = page.getByRole('main');

  const sidebarBox = await sidebar.boundingBox();
  const mainBox = await main.boundingBox();

  expect(sidebarBox).not.toBeNull();
  expect(mainBox).not.toBeNull();
  expect(sidebarBox!.x).toBeLessThan(mainBox!.x);
});
```

## Viewport-Specific Tests

```typescript
const MOBILE = { width: 375, height: 812 };
const TABLET = { width: 768, height: 1024 };
const DESKTOP = { width: 1280, height: 720 };

test.describe('Responsive layout', () => {
  test('mobile: hamburger menu replaces nav links', async ({ page }) => {
    await page.setViewportSize(MOBILE);
    await page.goto('/');

    // Desktop nav hidden on mobile
    await expect(page.locator('nav.desktop-nav')).not.toBeVisible();
    // Hamburger visible
    await expect(page.getByRole('button', { name: /menu/i })).toBeVisible();
  });

  test('desktop: nav links visible, no hamburger', async ({ page }) => {
    await page.setViewportSize(DESKTOP);
    await page.goto('/');

    await expect(page.locator('nav.desktop-nav')).toBeVisible();
    await expect(page.getByRole('button', { name: /menu/i })).not.toBeVisible();
  });

  test('mobile: cards stack vertically', async ({ page }) => {
    await page.setViewportSize(MOBILE);
    await page.goto('/pricing');

    const cards = page.locator('[data-testid="pricing-card"]');
    const count = await cards.count();

    for (let i = 1; i < count; i++) {
      const prev = await cards.nth(i - 1).boundingBox();
      const curr = await cards.nth(i).boundingBox();
      // Each card below the previous one (stacked)
      expect(curr!.y).toBeGreaterThan(prev!.y);
    }
  });
});
```

## Loading and Timing

```typescript
test('content loads within acceptable time', async ({ page }) => {
  await page.goto('/', { waitUntil: 'networkidle' });

  // Key content visible after load
  await expect(page.getByRole('heading', { level: 1 })).toBeVisible({ timeout: 3000 });
});

test('skeleton loader shows then content replaces it', async ({ page }) => {
  await page.goto('/dashboard');

  // Skeleton appears first (may be brief)
  const skeleton = page.locator('[data-testid="skeleton"]');
  const content = page.locator('[data-testid="dashboard-content"]');

  // Content eventually appears
  await expect(content).toBeVisible({ timeout: 10000 });

  // Skeleton gone
  await expect(skeleton).not.toBeVisible();
});

test('no layout shift after images load', async ({ page }) => {
  await page.goto('/');

  // Capture position before images load
  const heading = page.getByRole('heading', { level: 1 });
  await expect(heading).toBeVisible();
  const positionBefore = await heading.boundingBox();

  // Wait for all images
  await page.waitForLoadState('networkidle');
  const positionAfter = await heading.boundingBox();

  // Position should not shift
  expect(positionAfter!.y).toBeCloseTo(positionBefore!.y, 0);
});
```

## Interaction Flows

```typescript
test('clicking CTA navigates to signup', async ({ page }) => {
  await page.goto('/');

  await page.getByRole('button', { name: /get started/i }).click();
  await expect(page).toHaveURL(/.*sign-?up/i);
});

test('form submission shows success message', async ({ page }) => {
  await page.goto('/contact');

  await page.getByLabel('Name').fill('Test User');
  await page.getByLabel('Email').fill('test@example.com');
  await page.getByLabel('Message').fill('Test message');
  await page.getByRole('button', { name: /send|submit/i }).click();

  await expect(page.getByText(/thank you|success|sent/i)).toBeVisible();
});

test('dropdown menu opens and shows options', async ({ page }) => {
  await page.goto('/dashboard');

  const trigger = page.getByRole('button', { name: /account/i });
  await trigger.click();

  const menu = page.getByRole('menu');
  await expect(menu).toBeVisible();
  await expect(menu.getByRole('menuitem', { name: /profile/i })).toBeVisible();
  await expect(menu.getByRole('menuitem', { name: /log out/i })).toBeVisible();
});
```

## Page-Level Assertions

```typescript
test('page has correct title and meta', async ({ page }) => {
  await page.goto('/pricing');

  await expect(page).toHaveTitle(/pricing/i);
});

test('no console errors on page load', async ({ page }) => {
  const errors: string[] = [];
  page.on('console', (msg) => {
    if (msg.type() === 'error') errors.push(msg.text());
  });

  await page.goto('/');
  await page.waitForLoadState('networkidle');

  expect(errors).toEqual([]);
});

test('no broken images', async ({ page }) => {
  await page.goto('/');
  await page.waitForLoadState('networkidle');

  const images = page.locator('img');
  const count = await images.count();

  for (let i = 0; i < count; i++) {
    const naturalWidth = await images.nth(i).evaluate(
      (img: HTMLImageElement) => img.naturalWidth
    );
    expect(naturalWidth).toBeGreaterThan(0);
  }
});
```

## Accessibility Quick Checks

```typescript
test('all images have alt text', async ({ page }) => {
  await page.goto('/');

  const images = page.locator('img');
  const count = await images.count();

  for (let i = 0; i < count; i++) {
    const alt = await images.nth(i).getAttribute('alt');
    expect(alt).toBeTruthy();
  }
});

test('focus order is logical through main content', async ({ page }) => {
  await page.goto('/');

  // Tab through interactive elements and check order
  const focusOrder: string[] = [];
  for (let i = 0; i < 10; i++) {
    await page.keyboard.press('Tab');
    const focused = await page.evaluate(() => {
      const el = document.activeElement;
      return el?.textContent?.trim().substring(0, 30) || el?.tagName || 'unknown';
    });
    focusOrder.push(focused);
  }

  // Verify no unexpected jumps (log for manual review)
  expect(focusOrder.length).toBeGreaterThan(0);
});
```
