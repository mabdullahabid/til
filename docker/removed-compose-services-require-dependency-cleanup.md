# Removed Compose Services Require Dependency Cleanup

When a production Compose stack moves Postgres or Redis to external infrastructure, deleting the bundled service definitions is not enough. Remove every `depends_on` reference to those services as well, or Compose rejects the configuration before deployment.

```bash
docker compose -f docker-compose.production.yml config -q
```

Run this validation in CI to catch dangling service dependencies before deploy.
