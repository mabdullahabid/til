# Shopify Webhook Date Trap

When handling Shopify webhooks for order automation, always validate using the original creation date - not the last modified date.

## The Bug

A historical order from 2018 was merged/updated by a merchant, which triggered a webhook. The automation system processed it as a "new" order and attempted to place it at a factory for a product that no longer existed.

## The Fix

Validate `created_at` from the Shopify payload before processing:

```python
def handle_order_webhook(payload):
    created_at = datetime.fromisoformat(payload['created_at'])
    
    # Reject old orders that are just being updated
    if is_older_than_threshold(created_at, days=7):
        logger.info(f"Skipping old order from {created_at}")
        return
    
    # Process legitimate new orders
    process_order(payload)
```

## Why This Happens

Shopify webhooks fire on any order change - not just new orders. Merges, edits, refunds, and tag updates all trigger the same webhook. The `updated_at` timestamp changes, but `created_at` remains the true indicator of when the order was originally placed.

## The Lesson

Webhooks tell you *something* changed. They don't tell you *what* matters for your use case. Always validate payload age before acting on it.
