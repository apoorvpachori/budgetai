# Application directories

- Keep desktop UI, local API, and persistence responsibilities separate: the UI calls the API and never accesses the database directly.
- Keep API contracts explicit and versioned under `/api/v1`.
- Shared product behavior belongs in the backend application/domain layer, not duplicated in React components or HTTP handlers.
- The intended app is single-user and single-device; do not add LAN serving, account sharing, or remote access to v1.
- Read the local `AGENTS.md` in `api/`, `desktop/`, or `ui/` before changing that application.
