# FastAPI branch review signals (session reference)

Context:
- Branch introduced readiness expansion, DB pool settings, and auth/dependency updates.

High-confidence review signals captured:
1. Readiness toggle vs eager dependency resolution
   - If route signature includes `engine: Engine = Depends(get_db_engine)` and `minio_client: Minio = Depends(get_minio_client)`, those dependencies resolve before branch logic.
   - Health toggles (`check_database/check_minio`) do not prevent initialization side effects unless dependencies are fetched lazily inside guarded blocks.

2. Import-time config freezing in auth dependencies
   - `OAuth2PasswordBearer(tokenUrl=EnvConfig().token_url)` at module import can freeze token URL and add import-time config side effects.
   - Prefer static token URL or app-construction-time wiring.

3. Test runner correctness
   - Raw `pytest` may fail in non-activated environment even when code is fine.
   - In `uv` projects, `uv run pytest -q` is the reliable validation command.

4. Documentation/packaging drift checks
   - After moving docs directories, verify `tool.setuptools.package-data` paths still exist.
   - Ensure README feature-state labels (e.g., "planned") match implemented code.

Use:
- Apply these as explicit review probes whenever reviewing FastAPI service-library branches with dependency wiring and config layers.