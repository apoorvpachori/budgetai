# BudgetAI

BudgetAI is a self-hosted personal budgeting app built around a simple workflow: import a credit-card statement, review and categorize its transactions, explore spending, and ask questions about the data in natural language. The intended MVP is a desktop app that runs on the owner's computer and keeps its database there; it is not a shared LAN or hosted service.

The goal is a private, understandable alternative to services such as Mint or Rocket Money. Each installation belongs to the person or household running it; the project should not require a BudgetAI account, a BudgetAI-hosted backend, or a network-accessible server.

## Project status

This repository is at the planning and foundation stage. The application itself has not been implemented yet. The architecture and contribution instructions below are the current project decisions, not a claim that those features or commands already work. The detailed MVP requirements are in [`docs/prd.md`](docs/prd.md).

## MVP

The first useful release is deliberately narrow:

1. Import a statement for one credit-card account per installation.
2. Preview parsed transactions and catch duplicates before saving.
3. Categorize transactions, with a way for the user to correct categories and add simple rules.
4. Show spending by period, category, and merchant; let the owner set simple monthly category budgets.
5. Answer natural-language questions using the installation's transaction history and show the date range and transactions or totals behind each answer.

The first importer targets one Chase Sapphire Preferred card and assumes USD. Owners can import statements from any month beginning January 2026, in any order and with gaps; the app must show complete, partial, and missing date coverage rather than treating missing months as zero spending. The exact Chase statement format (PDF, CSV, or both) is still to be selected. Bank connections, multi-card workflows, shared hosted accounts, and financial advice are outside the first MVP. See the PRD for release boundaries, user journeys, requirements, and open decisions.

## Architecture direction

- **Monorepo:** `apps/desktop` contains the desktop shell, `apps/ui` contains its React interface, and `apps/api` contains the local Python backend. `deploy` contains desktop packaging guidance; `docs` contains product and operating guidance.
- **Desktop shell:** The first release targets macOS and uses Tauri to open the React/Vite interface in an app window; the owner does not need to open a browser or expose the app to the network.
- **Frontend:** React with TypeScript and Vite. Keep it as a client-rendered interface that can run in the desktop webview and during browser-based development.
- **Backend:** Python with FastAPI, launched as a local sidecar/process by the desktop app. Keep routes thin and product logic in application/domain services.
- **API contract:** REST endpoints under `/api/v1` for communication between the UI and local API. FastAPI's OpenAPI schema is the source for frontend API types.
- **Persistence:** SQLite for the one-account, single-installation MVP, stored in the operating system's per-user application data directory. Use SQLAlchemy for database access and Alembic for schema migrations.
- **Local-only operation:** The API must not listen on the LAN or public network. The app should work without an external account or hosted backend; any external AI provider is a separate opt-in connection.
- **Imports:** Separate file-specific parsing from transaction normalization, duplicate detection, preview, and persistence. The user confirms a preview before it changes stored data.
- **Categorization:** Deterministic built-in categories and user-editable rules/overrides come first. Model-generated category suggestions can be optional later.
- **Assistant:** Natural-language requests call a small set of read-only application services for things like spending by category, merchant search, and period comparisons. The assistant must not write transactions or issue arbitrary SQL.

For the full component boundaries, data flow, and open decisions, see [`docs/architecture.md`](docs/architecture.md).
For detailed product scope and acceptance criteria, see [`docs/prd.md`](docs/prd.md).

## Repository map

```text
.
├── AGENTS.md                 # Instructions for coding agents working in this repository
├── README.md                 # Product scope, status, architecture, and contributor entry point
├── apps/
│   ├── AGENTS.md             # Shared rules for applications
│   ├── api/
│   │   └── AGENTS.md         # Python/FastAPI local backend rules
│   ├── desktop/
│   │   └── AGENTS.md         # Desktop shell and packaging rules
│   └── ui/
│       └── AGENTS.md         # React/Vite desktop interface rules
├── deploy/
│   └── AGENTS.md             # Desktop packaging and release rules
└── docs/
    ├── AGENTS.md             # Documentation maintenance rules
    ├── architecture.md       # System design and current decisions
    └── prd.md                # MVP product requirements and acceptance criteria
```

Every tracked project directory has a scoped `AGENTS.md`. `.git` and future generated, dependency, cache, build, and runtime-data directories are intentionally excluded.

## Development

There is no runnable application yet, so development and packaging commands will be added when the desktop shell, frontend, and API are scaffolded. The intended developer workflow is to run Vite and FastAPI locally and launch the desktop shell in development mode. A release should install as a desktop app and store data in the user's OS application-data directory; Docker is not required for normal use.

Do not put real credit-card statements or transaction exports in the repository. Use synthetic or explicitly sanitized sample data for development documentation and examples.

## Privacy and local operation

- The app runs under the user's OS account; the user controls the database and backups on their computer.
- Financial data stays on that host unless the user explicitly configures an external model provider.
- External AI requests, when implemented, must be opt-in and documented with what data is sent.
- A basic local installation should not need a cloud account, bank-linking provider, or telemetry service.
- The local API must not be exposed to other devices on the network. Guidance must explain storage, backup/restore, desktop updates, and any optional external model-provider data flow.

## Decisions still to make

- Which Chase Sapphire Preferred statement format the first importer will support (PDF, CSV, or both).
- The minimum supported macOS version and packaging details. macOS is the first target; other operating systems can follow.
- Which local or external model providers, if any, will be supported by the first assistant implementation.
- The exact charting and UI component libraries.
- The default display locale for dates and numbers; transaction amounts are assumed to be USD.

Keep these decisions visible here until implementation settles them. Avoid introducing dependencies just to make an undecided feature look complete.

## Useful upstream documentation

- [React: Creating a React App](https://react.dev/learn/creating-a-react-app)
- [Vite: Getting Started](https://vite.dev/guide/)
- [Tauri: Frontend configuration](https://v2.tauri.app/start/frontend/)
- [Tauri: Bundling sidecars](https://v2.tauri.app/develop/sidecar/)
- [Tauri: Distribution and signing](https://v2.tauri.app/distribute/)
- [FastAPI: Features and automatic API documentation](https://fastapi.tiangolo.com/features/)
- [SQLAlchemy ORM](https://docs.sqlalchemy.org/en/20/orm/)
- [Alembic Tutorial](https://alembic.sqlalchemy.org/en/latest/tutorial.html)
