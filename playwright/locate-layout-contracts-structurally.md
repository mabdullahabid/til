# Locate Layout Contracts Structurally in Playwright

Layout tests should not depend on CMS-controlled copy, URLs, or whether an action renders as a link or button. Scope the locator to a stable wrapper and select the relevant interactive element before asserting its viewport position.

```ts
const primaryCta = page
  .locator('.home-hero__actions')
  .locator('a, button')
  .first();
```

This keeps the layout contract stable while editors change the CTA implementation.
