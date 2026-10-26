# Backend application

- Backend implementation is planned in Python with FastAPI as a modular monolith.
- Keep route handlers focused on HTTP parsing, validation, and response mapping. Put import, categorization, analytics, and assistant behavior in reusable application services.
- Use Pydantic models at API boundaries and publish the API contract through OpenAPI.
- Use SQLAlchemy for persistence and Alembic migrations for schema changes. Persist the SQLite database only in the configured runtime data directory, never in the source tree or container image.
- Treat statement files as untrusted input: validate size/type, parse without side effects, normalize, detect duplicates, show a preview, and persist only after confirmation.
- Preserve source descriptions and dates alongside normalized merchant/category fields so a user can inspect and correct imported results.
- Keep agent-facing operations read-only and bounded. They must call application services, not execute model-authored SQL.
- Read the nearest nested `AGENTS.md` before adding a backend subdirectory.
