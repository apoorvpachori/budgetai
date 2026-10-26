# Repository instructions

## Start here

- Read `README.md` for the product goal, MVP boundary, current status, and local development entry points.
- Read `docs/prd.md` before changing product scope, user journeys, data coverage, or acceptance criteria.
- Read `docs/architecture.md` before changing project structure, persistence, imports, or the natural-language assistant.
- Follow the closest `AGENTS.md` for the area being changed, in addition to this file.

## Product constraints

- This is an open-source personal desktop app that users install and control themselves. Keep the default installation small, understandable, and local-first.
- MVP is single-user and single-device. Do not expose a server on the LAN or add remote access or sync without a scope change.
- MVP supports one credit-card account per installation. Keep the schema extendable to more accounts without building multi-account UI or workflows now.
- Prioritize correctness and reviewability of imported financial data over automation or visual polish.
- Do not imply that a feature exists until it has been implemented. Keep `README.md` and `docs/architecture.md` aligned with the code.

## Data and privacy rules

- Store money as integer minor units with an explicit currency. Never use floating point for financial amounts.
- Make statement imports previewable and idempotent. Preserve enough source information to explain how a normalized transaction was produced.
- Keep transaction access behind application services. Natural-language features must use bounded, read-only tools over those services; never give a model unrestricted database access or arbitrary SQL execution.
- Keep external model-provider use optional and clearly configurable. Do not add telemetry or send financial records to external services by default.
- Do not log statement contents, full transaction descriptions, or other financial details.

## Repository conventions

- Every tracked project directory has an `AGENTS.md`. `.git`, dependency folders, build output, caches, and runtime data are excluded. Add a short, area-specific file when adding a new project directory; do not copy this whole file into it.
- Prefer a modular monolith. Do not add separate services, queues, or infrastructure unless a concrete MVP need requires them.
- Keep API schemas as the contract between the frontend and backend. Prefer generating frontend types from the backend's OpenAPI schema over maintaining duplicate hand-written API types.
- When changing a user-facing flow, deployment command, environment variable, or feature status, update the relevant README or docs in the same change.
- Do not commit secrets, real statement files, or personal financial data.
