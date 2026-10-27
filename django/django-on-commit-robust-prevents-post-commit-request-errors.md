# Django `on_commit(..., robust=True)` Contains Best-Effort Callback Failures

A default `transaction.on_commit()` callback runs after the database commits. If it raises, the write is already committed, but the exception can still make the request look like it failed. A retry might then repeat an operation that actually succeeded.

In Django 4.2+, use `robust=True` when the callback is genuinely best effort:

```python
from django.db import transaction


def save_record(record):
    with transaction.atomic():
        record.save()
        record_id = record.pk
        transaction.on_commit(
            lambda: send_optional_notification(record_id),
            robust=True,
        )
```

With `robust=True`, Django catches and logs exceptions derived from `Exception` and continues running later callbacks. This does not make delivery durable, retryable, or guaranteed. If the side effect is essential, use an outbox or another durable handoff instead. Test commit behavior with `captureOnCommitCallbacks(execute=True)` or a transaction-aware test, because `TestCase` normally rolls back rather than committing.
