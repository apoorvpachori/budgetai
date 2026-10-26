# Application directories

- Keep browser and server responsibilities separate: the web app calls the API and never accesses the database directly.
- Keep API contracts explicit and versioned under `/api/v1`.
- Shared product behavior belongs in the backend application/domain layer, not duplicated in React components or HTTP handlers.
- Read the local `AGENTS.md` in `api/` or `web/` before changing that application.
