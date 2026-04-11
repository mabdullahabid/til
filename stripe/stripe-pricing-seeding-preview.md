# Stripe Pricing Seeding for Preview Deployments

When using preview deployments for testing billing flows, Stripe pricing rules need to be automatically seeded or the billing UI will fail with missing price errors.

## The Problem

Preview environments spin up fresh databases, but Stripe pricing objects aren't automatically created. When a user hits the billing page, the application can't find the expected price IDs.

## Solutions

### Option 1: Auto-seed on Deploy

Run migration command automatically when preview deploys:

```yaml
# Example: GitHub Actions or CI pipeline
- name: Seed Stripe pricing
  run: |
    if [ "$ENVIRONMENT" = "preview" ]; then
      npm run stripe:seed-prices
    fi
  env:
    STRIPE_SECRET_KEY: ${{ secrets.STRIPE_TEST_SECRET_KEY }}
```

### Option 2: Manual Seeding

For one-off testing:

```bash
# Seed prices manually
npm run stripe:seed-prices -- --env=preview
```

### Option 3: Mock in Preview

Use Stripe's test mode with fixed price IDs that are pre-created in your test account.

## The Lesson

Preview environments need the same external service setup as production. Document or automate the seeding process - don't leave it as a manual step that gets forgotten.
