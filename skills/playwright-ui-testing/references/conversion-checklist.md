# Conversion Optimisation Checklist

Test criteria for asserting that pages are optimised for user conversion. Generate Playwright tests against these items for landing pages, pricing pages, signup flows, and product pages.

## Above the Fold

The viewport area visible without scrolling (assume 720px height on desktop, 812px on mobile).

| Element | Assertion |
|---------|-----------|
| Primary headline | `h1` visible, communicates core value |
| Supporting subheadline | Clarifies the headline, visible below it |
| Primary CTA | Button visible, action-oriented text (not "Submit" or "Click here") |
| Hero image/visual | Relevant visual reinforcing the message |
| No clutter | Maximum 1 primary CTA above the fold — avoid competing actions |

```typescript
test('above-the-fold conversion elements', async ({ page }) => {
  await page.goto('/');
  const viewport = page.viewportSize()!;

  // Primary headline visible and above fold
  const h1 = page.getByRole('heading', { level: 1 });
  await expect(h1).toBeVisible();
  const h1Box = await h1.boundingBox();
  expect(h1Box!.y + h1Box!.height).toBeLessThan(viewport.height);

  // Primary CTA above fold
  const cta = page.getByRole('link', { name: /get started|sign up|try free/i });
  await expect(cta).toBeVisible();
  const ctaBox = await cta.boundingBox();
  expect(ctaBox!.y + ctaBox!.height).toBeLessThan(viewport.height);
});
```

## CTAs (Calls to Action)

| Criteria | What to Assert |
|----------|----------------|
| Action-oriented text | Button text uses verbs: "Start free trial", "Get started", "Book a demo" |
| Visual prominence | CTA has distinct colour, stands out from background |
| Single primary action | One dominant CTA per section, secondary actions visually subdued |
| Repeated strategically | CTA reappears after key content sections (features, testimonials, pricing) |
| Mobile: thumb-friendly | Minimum 44px height, full-width or centred on mobile |

```typescript
test('CTA buttons use action-oriented text', async ({ page }) => {
  await page.goto('/');

  const ctas = page.locator('[data-testid="cta"], .cta, a.btn-primary, button.btn-primary');
  const count = await ctas.count();
  expect(count).toBeGreaterThan(0);

  for (let i = 0; i < count; i++) {
    const text = await ctas.nth(i).textContent();
    // Should not be generic
    expect(text?.toLowerCase()).not.toMatch(/^(submit|click here|learn more)$/);
  }
});
```

## Trust Signals

| Element | Where |
|---------|-------|
| Customer logos | Near the hero or below the fold |
| Testimonials | Mid-page, with names/photos/roles |
| Star ratings or review counts | Near CTAs or product descriptions |
| Security badges | Near forms or payment sections |
| Social proof numbers | "10,000+ customers", "4.9 stars" |
| Case studies or results | Supporting detailed claims |

```typescript
test('trust signals are present on landing page', async ({ page }) => {
  await page.goto('/');

  // At least one trust signal type should be present
  const logos = page.locator('[data-testid="customer-logos"], .customer-logos, .trust-logos');
  const testimonials = page.locator('[data-testid="testimonials"], .testimonials, .reviews');
  const socialProof = page.locator('[data-testid="social-proof"], .social-proof');

  const hasLogos = await logos.count() > 0;
  const hasTestimonials = await testimonials.count() > 0;
  const hasSocialProof = await socialProof.count() > 0;

  expect(hasLogos || hasTestimonials || hasSocialProof).toBe(true);
});
```

## Forms

| Criteria | What to Assert |
|----------|----------------|
| Minimal fields | Only ask for what's needed — every field increases drop-off |
| Visible labels | Labels above or beside fields (not just placeholder text) |
| Inline validation | Errors appear next to the field, not just at the top |
| Button text | Describes what happens ("Create account"), not generic ("Submit") |
| Progress indicators | Multi-step forms show which step you're on |
| Error recovery | Filled data preserved after validation failure |

```typescript
test('form has visible labels and clear submit text', async ({ page }) => {
  await page.goto('/signup');

  // All inputs should have associated labels
  const inputs = page.locator('input:not([type="hidden"])');
  const count = await inputs.count();

  for (let i = 0; i < count; i++) {
    const input = inputs.nth(i);
    const id = await input.getAttribute('id');
    const ariaLabel = await input.getAttribute('aria-label');
    const placeholder = await input.getAttribute('placeholder');

    if (id) {
      const label = page.locator(`label[for="${id}"]`);
      const hasLabel = await label.count() > 0;
      const hasAria = !!ariaLabel;
      expect(hasLabel || hasAria).toBe(true);
    }
  }

  // Submit button has descriptive text
  const submit = page.getByRole('button', { name: /submit/i });
  // If it says just "Submit", flag it
  if (await submit.count() > 0) {
    const text = await submit.textContent();
    expect(text?.trim().toLowerCase()).not.toBe('submit');
  }
});
```

## Visual Hierarchy

| Criteria | What to Assert |
|----------|----------------|
| Heading structure | Single h1, logical h2→h3 progression |
| Content grouping | Related elements in distinct sections |
| Whitespace | Sections have adequate spacing (not crammed) |
| Contrast | Key elements visually distinct from background |
| Scannable | Short paragraphs, bullet lists, bold key phrases |

```typescript
test('page has correct heading hierarchy', async ({ page }) => {
  await page.goto('/');

  // Exactly one h1
  const h1s = page.locator('h1');
  expect(await h1s.count()).toBe(1);

  // h2s should exist for main sections
  const h2s = page.locator('h2');
  expect(await h2s.count()).toBeGreaterThan(0);
});
```

## Mobile-Specific Conversion

| Criteria | What to Assert |
|----------|----------------|
| Sticky CTA | CTA remains accessible as user scrolls |
| Tap targets | Buttons/links minimum 44x44px |
| No hidden CTAs | Primary CTA not inside a collapsed menu |
| Fast load | Key content visible within 3 seconds |
| Thumb zone | Primary actions in lower half of screen (reachable) |

```typescript
test('mobile: tap targets are large enough', async ({ page }) => {
  await page.setViewportSize({ width: 375, height: 812 });
  await page.goto('/');

  const buttons = page.locator('button, a[role="button"], [data-testid="cta"]');
  const count = await buttons.count();

  for (let i = 0; i < count; i++) {
    const box = await buttons.nth(i).boundingBox();
    if (box) {
      expect(box.height).toBeGreaterThanOrEqual(44);
    }
  }
});
```

## Pricing Page Specific

| Criteria | What to Assert |
|----------|----------------|
| Clear plan comparison | Plans side by side (desktop) or stacked with clear labels (mobile) |
| Recommended plan highlighted | One plan visually distinct (badge, colour, size) |
| Feature differentiation | Clear what each tier includes vs doesn't |
| Annual/monthly toggle | If offered, works correctly and updates prices |
| CTA per plan | Each plan has its own clear action button |

## Page Speed and Loading

| Criteria | What to Assert |
|----------|----------------|
| Content visible quickly | Key elements render within 3s |
| No layout shift | Elements don't jump after async content loads |
| Lazy images | Below-fold images load on scroll, not upfront |
| Loading indicators | Skeleton screens or spinners for async content |
