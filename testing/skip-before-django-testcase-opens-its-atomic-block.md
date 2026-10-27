# Skip Before Django TestCase Opens Its Atomic Block

Django `TestCase` opens a class-level atomic block in `setUpClass()`. If your class probes an optional external dependency *after* calling `super().setUpClass()` and then raises `SkipTest`, the test runner may skip `tearDownClass()`. That can leave the transaction open and break the next database test in the same process.

Probe first:

```python
import unittest
from django.test import TestCase


class BrowserDatabaseTest(TestCase):
    @classmethod
    def setUpClass(cls):
        if not browser_is_available():
            raise unittest.SkipTest("Browser unavailable")
        super().setUpClass()
```

Here `browser_is_available()` is your dependency probe. If the test never touches the database, inherit from `SimpleTestCase` instead and avoid the database transaction entirely.

To catch the leak, run a database test immediately after the skipped class in the same test process. Running the skipped class alone can hide the problem.
