# Readiness dependency-resolution probe (FastAPI)

When reviewing health/readiness toggles, verify that disabled checks do not still trigger dependency initialization.

## Why this matters
If route parameters include eager dependencies like:
- `engine: Engine = Depends(get_db_engine)`
- `minio_client: Minio = Depends(get_minio_client)`

FastAPI resolves them before entering function logic. Feature flags such as `settings.health.check_database=False` or `check_minio=False` will not prevent initialization side effects.

## Minimal repro pattern
1. Build app with health router.
2. Override settings to disable DB/MinIO readiness checks.
3. Override config with an intentionally invalid DB URL.
4. Call `GET /health/readiness`.

Expected behavior (correct lazy design):
- Endpoint should not fail due to DB initialization when DB check is disabled.

Observed failure signal (eager design):
- SQLAlchemy URL parsing / engine-creation error occurs before readiness logic runs.

## Review guidance
- If toggles exist, dependency acquisition for optional checks must be conditional/lazy.
- Prefer fetching optional resources inside guarded blocks rather than as unconditional route parameters.
