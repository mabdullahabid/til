# PR Review Apps Isolation

PR review apps provide isolated environments essential for debugging Sentry errors per environment. Each PR gets its own namespace with isolated resources.

## Environment Isolation

```
main branch → production namespace
feature/pr-123 → pr-123 namespace
feature/pr-124 → pr-124 namespace
```

This lets you:
- Trace errors to specific PRs via Sentry's environment tags
- Test configuration changes safely
- Reproduce bugs in isolated conditions

## Configuration Management

ConfigMaps handle environment variables, but there's a catch:

```yaml
# ConfigMap defines env vars
apiVersion: v1
kind: ConfigMap
metadata:
  name: app-config
data:
  API_URL: "https://api-preview.example.com"
```

**Warning:** New deployments override manual namespace changes. If you edit env vars directly in a namespace, the next deployment will reset them to ConfigMap values.

## Secrets Best Practice

Keep secrets in GitHub Actions environment secrets, not in:
- Manual namespace edits (will be overwritten)
- ConfigMaps (not encrypted)
- Source code (obviously)

```yaml
# .github/workflows/preview.yml
- name: Deploy
  env:
    DATABASE_URL: ${{ secrets.PREVIEW_DATABASE_URL }}
    API_KEY: ${{ secrets.PREVIEW_API_KEY }}
```

## The Lesson

Review apps are powerful for testing, but their ephemeral nature means state doesn't persist. Design your deployment pipeline to be the source of truth for all configuration - manual changes are temporary by design.
