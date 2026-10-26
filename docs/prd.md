# Product Requirements Document

- **Product:** BudgetAI (working name)
- **Status:** Draft for MVP planning
- **Updated:** 2026-10-26
- **Implementation status:** Not started; this document describes intended behavior.

## 1. Summary

BudgetAI is a self-hosted personal desktop app inspired by the useful parts of Rocket Money: a clear spending overview, categorized transactions, simple budgets, recurring-charge awareness, and a conversational assistant. The owner installs and runs it on their own computer; v1 is not a network-accessible server or shared service. The first release is intentionally grounded in one source of data: statements the owner uploads for one credit-card account.

The owner imports Chase Sapphire Preferred statements for any month from January 2026 onward, reviews the parsed transactions, corrects categories, and sees where card spending went. Months may be uploaded in any order and gaps are allowed; the app must make missing coverage visible. The owner can then ask questions such as “How much did I spend on restaurants last month?” The assistant must calculate from the imported data and show its date range and supporting transactions or totals.

This is not a full financial picture. A single card statement cannot show a household’s income, cash balance, other account spending, net worth, or reliable upcoming bill schedule. The interface and assistant must say so clearly.

## 2. Product direction

### Vision

Make it easy for a person to understand and explore their own spending without handing their financial history to a budgeting service. The app should feel approachable and polished, while remaining understandable to run, back up, and extend.

### Product principles

1. **User-owned data:** The app runs locally; a basic installation needs no BudgetAI account, cloud database, network-accessible service, or bank-linking provider.
2. **Trust before cleverness:** Imported rows, totals, categorization, and assistant answers must be inspectable and correctable.
3. **Show the coverage:** Every spending view and answer makes clear which card and dates it covers.
4. **Automation stays reviewable:** Rules and AI can help organize data, but must not silently erase or rewrite financial records.
5. **One card is a product boundary for v1:** Support one card account per installation; keep internal identifiers extensible without building multi-card workflows.

## 3. Target user and primary job

### Primary user

A person or household that wants a private way to review one credit card’s spending on their own computer. They install a desktop app and upload statement files manually. They may not want to connect bank credentials or use a hosted budgeting provider.

### Primary job to be done

“When I upload my credit-card statements from this year, help me see where the money went, fix mistakes, compare the periods I have, and ask follow-up questions in plain language.”

### Assumptions for the first release

- One desktop installation is used by one person or household on one computer. Multi-user identity, sharing, LAN access, and multi-device sync are not part of v1.
- The first supported account is a Chase Sapphire Preferred credit card.
- The first desktop release targets macOS. The minimum supported macOS version is still to be selected.
- The initial reporting window begins January 1, 2026. The owner may import any available statement month from then onward, in any order, without needing every month. Later statements remain importable as they arrive; there is no two-statement or two-month history cap.
- The first importer will support one Chase statement layout. Whether the owner uploads a PDF statement or a CSV/other transaction export remains open until a representative file is available.
- All imported transaction amounts and budgets are treated as USD for MVP. Multi-currency conversion is out of scope.
- The app uses the transaction/posting dates from the statement. A statement’s payment due date or ending balance is not treated as spending.

## 4. Goals and non-goals

### Goals for MVP

- Import statement data for one credit card and make the result reviewable before it is saved.
- Reliably avoid duplicate transactions when importing overlapping or repeated statements.
- Categorize transactions automatically enough to make a useful first dashboard, while making corrections easy.
- Show spending over time, by category, and by merchant.
- Let the owner set simple monthly category budgets and see progress against the imported card data.
- Answer natural-language spending questions from the same calculations used by the dashboard.
- Make the limits of the card-only data obvious in the UI and assistant.
- Ship as a locally installed desktop app with data stored on the owner's computer; do not expose a server on the home network.

### Explicit non-goals for MVP

- Bank linking, credential storage, or automatic transaction syncing.
- More than one card/account per installation, account sharing, or hosted multi-tenant service.
- Access from other devices over a LAN, remote access, or multi-device sync.
- Full net-worth, checking-account cash-flow, income, savings, or debt-planning views.
- Canceling subscriptions, negotiating bills, moving money, or contacting merchants.
- Credit-score monitoring, credit reports, investment tracking, or financial advice.
- OCR for arbitrary scans or photos before the first statement source is chosen.
- Proactive notifications, mobile apps, or background monitoring.
- An agent with unrestricted SQL or write access to transaction records.

## 5. MVP scope and release boundary

### P0: required for first usable release

1. Single-user desktop app with persistent local storage under the owner’s OS account; no network-accessible service is required.
2. Upload one or more Chase Sapphire Preferred statement files for any month from January 2026 onward, including gaps or out-of-order months.
3. Parse one agreed Chase source format, treat transaction amounts as USD, validate the rows, detect exact and likely duplicates, and show a preview before commit.
4. Store a normalized transaction ledger with search and filters.
5. Assign built-in categories; let the user correct a category and create a simple merchant-based rule.
6. Provide a spending dashboard with total spend, category breakdown, monthly/statement-period trend, and top merchants.
7. Let the user set or edit simple monthly budget amounts by category and see amount spent and amount remaining for the currently available data.
8. Provide in-app assistant chat for read-only questions about imported transactions, categories, merchants, periods, and budgets.
9. Show date coverage and supporting evidence for dashboard totals and assistant answers.
10. Provide a way to export transaction data and clear the installation’s data.

### P1: follow-up after the first usable release

- Candidate recurring-charge detection based on repeated transactions. Label results as “possible recurring” and show the observed dates and amounts; two occurrences are weak evidence and must not be represented as a confirmed subscription or guaranteed next bill.
- Assistant-proposed transaction category changes, rule creation, and budget edits, each shown as a preview and requiring explicit confirmation before applying.
- More statement formats and configurable merchant normalization.
- Optional local or external model-provider integrations beyond the first provider chosen for v1.

### Later, only if the product needs them

- Multi-card data, account sharing, bank sync, recurring-bill calendar, subscription cancellation workflows, financial goals, and richer household budgeting.

## 6. Primary user journeys

### Journey A: first setup and import

1. The owner launches the desktop app and sees that data stays on this computer and that only uploaded card statements are included.
2. The owner selects any available monthly Chase Sapphire Preferred statements from January 2026 onward and uploads them. They can import months out of order or skip months.
3. The app reports the detected statement period, transaction count, parsed totals, rows needing attention, and suspected duplicates.
4. The owner reviews the transaction preview, corrects invalid or ambiguous rows where possible, and confirms the import.
5. The app records the import and displays the coverage range. The original uploaded document is discarded after processing by default; stored transaction rows and import metadata remain.
6. The owner lands on an overview that explains how to add another statement later.

### Journey B: understand and correct spending

1. The owner sees total card spending for the selected period, a category breakdown, a trend, top merchants, and recent transactions.
2. The owner selects a chart segment or merchant to open a filtered transaction list.
3. The owner changes a transaction’s category or normalized merchant name.
4. The app offers to apply a simple rule to future matching transactions. Applying a rule is a separate, explicit choice.
5. Later imports apply the owner’s saved rules without overriding direct transaction corrections.

### Journey C: ask the assistant

1. The owner asks a question in natural language, such as “How much did I spend on groceries in September?” or “What were my five largest charges last month?”
2. The assistant determines whether the requested period and category/merchant are unambiguous. If not, it asks a short clarifying question.
3. It calls bounded, read-only application tools that use the same transaction semantics as the dashboard.
4. The answer states the result, currency, date range, data coverage, and a concise explanation.
5. A source panel links the answer to supporting transactions or aggregate breakdowns. If the data is missing or incomplete, the assistant says so rather than filling gaps.

## 7. Functional requirements and acceptance criteria

### FR-1: local desktop installation and first-run context

- The application installs and runs as a desktop app under the owner's OS account, with persistent local storage and no required BudgetAI account.
- Normal use must not require a network-accessible server. The app must not expose its local API to other devices on the network.
- First run identifies the app as a single-card spending view and explains that answers cover only imported statement data.
- Do not require full card number, CVV, bank password, or billing credentials. If an import contains sensitive card identifiers, do not persist them.
- Store the database in the OS per-user application data directory, not beside the executable or in the repository.
- If the desktop shell communicates with a local HTTP API, bind it to loopback only and restrict access to the app process.

**Acceptance:** The owner can launch the installed desktop app on a supported macOS version, retain data across restart, and see an empty state that guides them to import a statement. No LAN service is required or started.

### FR-2: statement upload, parsing, and preview

- The first implementation supports one Chase Sapphire Preferred source layout chosen from a representative statement export.
- Accept statement periods from January 2026 onward, including months uploaded out of order and gaps in coverage. Accept later statement periods as they occur.
- Accept multiple statement files for the same card in one import flow where practical; otherwise permit sequential imports with the same outcome.
- Extract at minimum: posted/transaction date, original description, USD amount, and statement period/source metadata.
- Normalize rows into a canonical transaction model without writing them to the database during parsing.
- Show parsed, invalid, and suspected-duplicate row counts. Do not silently drop rows.
- Show statement period and transaction totals. If the file has summary balances, label them separately from transaction spending.
- Provide a clear unsupported-file or parse-error message without persisting partial data.

**Acceptance:** The owner can inspect every row that will be imported before confirmation. A parse failure cannot partially commit an import. The preview makes clear how many rows are ready, skipped by choice, invalid, or potentially duplicated.

### FR-3: deduplication and import history

- Re-importing the exact same file must not create duplicate transactions.
- For overlapping statements, high-confidence duplicate candidates are identified using source metadata plus transaction attributes.
- Two otherwise identical purchases can be legitimate; ambiguous candidates are shown for review, not automatically discarded.
- Keep import time, source format, covered date range, and import summary for troubleshooting.

**Acceptance:** Confirming the same statement twice leaves the transaction count and totals unchanged. When the app cannot safely decide whether similar rows are duplicates, the owner chooses.

### FR-4: transaction ledger and corrections

- The ledger supports date range, category, merchant, and amount filters, search, and sorting.
- A transaction detail view shows the posted date, amount/currency, original description, normalized merchant, assigned category, and source statement/import.
- The user can edit category and normalized merchant without changing the original source description or raw imported amount.
- User edits persist and have higher precedence than default categorization and merchant rules.
- Support a simple notes field only if it can be implemented without expanding the first vertical slice materially; notes are not a release blocker.

**Acceptance:** A correction remains after reload and subsequent imports. The user can get from a dashboard aggregate to the transactions that contribute to it.

### FR-5: categorization and rules

- Include a practical starter set such as Dining, Groceries, Transportation, Shopping, Bills & Utilities, Travel, Health, Entertainment, Fees & Interest, and Other.
- Represent payments/transfers separately from purchases so card payments do not inflate spending.
- Categories may be grouped as spending, credit/refund, or excluded/non-spending.
- The owner can assign a transaction category and add a basic rule matching a merchant name or normalized merchant.
- Rule order/precedence is visible enough to explain why a category was assigned.
- Do not silently replace a manual correction with an automatic suggestion.

**Acceptance:** The same rule yields the same result on a later matching import. The owner can identify and override a mistaken category.

### FR-6: spending dashboard

- Show total net card spending in USD for a selected calendar month or statement period, with the date range and coverage visible.
- Support year-to-date reporting from January 1, 2026 onward and month selection for any available month in that range.
- Derive coverage from imported statement-period dates. Show missing or partially covered calendar months as such; never treat an unimported month as zero spending or a partially covered month as complete. Comparisons must identify the exact periods and coverage being compared.
- Show month-over-month or statement-period comparison when two comparable periods are present. Do not fabricate a comparison when only one period exists.
- Show category breakdown, spending trend, top merchants, and recent transactions.
- Chart sections link to the underlying filtered transactions.
- Use consistent definitions across dashboard, exports, and assistant.
- Card payment transactions and statement balances are excluded from purchase spending. Refunds/credits are visible and reduce net spend when attributable; otherwise show them distinctly.
- When the user has only one card, label results as card spending rather than total household spending.

**Acceptance:** A user can trace each dashboard total to included transactions and understand whether the value is gross charges, credits/refunds, or net spend.

### FR-7: simple budgets

- The owner can set a monthly limit per category, edit it, or remove it.
- Show spent, limit, and remaining amount for the current calendar month using imported card transactions.
- The UI states that these budgets cover only this card’s imported transactions and do not account for cash, debit cards, income, or other cards.
- Budget calculations use the same category and refund/payment rules as the dashboard.

**Acceptance:** Changing a transaction category updates the relevant category budget progress. Payments/transfers do not count toward category spending.

### FR-8: natural-language assistant

- Chat supports follow-up questions within the active conversation.
- MVP assistant questions are read-only; the assistant cannot change transactions, categories, budgets, or imports.
- Supported question types include spending totals, breakdowns by category/merchant, period comparisons, largest/smallest transactions, transaction search, and budget progress.
- Each answer includes currency, exact date range, coverage caveat when relevant, and evidence linking to source transactions or a named aggregate.
- Arithmetic and grouping come from deterministic application queries, not from the language model’s mental math.
- The assistant must state when the data is insufficient, an import is incomplete, or the question requires information outside the selected card.
- If an external provider is used, it is opt-in/configurable and the user is told what data is sent. A local provider may be supported as an alternative; provider selection remains open.

**Acceptance:** For a supported question, the displayed answer matches the corresponding dashboard/API aggregate and opens its supporting transactions. Unsupported whole-household questions receive an honest limitation response.

### FR-9: export and delete

- The owner can export normalized transactions to CSV, including dates, merchant, amount, currency, category, and source period.
- The owner can clear all imported financial data and categorization/budget settings from the installation.
- Neither operation requires sending data to a remote service.

**Acceptance:** The exported data can be read without the app. Clearing data returns the app to the first-run empty state.

## 8. Data and calculation rules

- Store monetary values as integer minor units and an explicit currency; do not use floating-point values for money. MVP imports and budget limits use USD.
- Use the statement’s posted date for spending views; keep import time separate.
- Preserve original descriptions alongside normalized merchant names.
- Store statement period start/end independently from transaction dates.
- Define positive amounts as purchases/fees and negative amounts as refunds/credits in the canonical ledger; normalize each source format to this convention.
- Credit-card payments are non-spending transfers and excluded from spend totals. The statement’s new balance/minimum payment/due amount is not a measure of purchases for that period.
- Net spend is purchases and spend-related fees less attributable refunds/credits. Show refunds separately where attribution is uncertain.
- Budgets use calendar months and posted dates. A statement-cycle view is separately labeled.
- Multi-currency conversion is outside MVP. If the source includes a foreign-currency original amount as well as the posted USD amount, use the USD posted amount for budgeting and spending totals.

## 9. Experience and navigation

Initial desktop navigation:

- **Overview:** spending summary, period selector, budget progress, category/merchant insights, and recent activity.
- **Transactions:** searchable ledger with category and date filters.
- **Budgets:** monthly category limits and progress.
- **Ask:** conversational assistant and evidence panel.
- **Settings:** import another statement, manage categories/rules, export/delete data, and configure assistant provider.

The first empty state should direct the owner to import. The first populated state should show the covered dates and distinguish imported card activity from whole-household finances. Mobile-friendly layout is desirable, but a native mobile app is out of scope.

## 10. Privacy, security, and reliability requirements

- The computer owner controls storage and backups; financial data is not sent to analytics or telemetry by default.
- Do not include real credit-card numbers or full descriptions in logs.
- Process statement files as untrusted input: enforce a documented size limit, validate type/structure, and avoid unsafe file paths or content execution.
- Delete temporary upload files after processing by default. Persist normalized transactions and minimal import metadata, not source PDFs, unless the owner explicitly enables retention later.
- Protect the local database through OS per-user file permissions and document backup handling.
- Require user confirmation before any future assistant action that changes financial records or settings.
- Do not expose the local API to LAN or internet interfaces. If the app uses a loopback HTTP API, protect it from unrelated local processes and web pages with scoped access controls.
- Failed parsing, model-provider errors, or unavailable charts must never corrupt saved transactions.

## 11. Success criteria

Because a privacy-first self-hosted app should not require product analytics, initial success is evaluated through acceptance and usability checks rather than default telemetry:

- A new user can import any available statement month from January 2026 onward, including months out of order, and understand which date ranges are complete, partial, or missing.
- No parsed row disappears silently; the import summary accounts for all rows.
- Reimporting a statement does not silently double-count it.
- A user can correct a category and see that correction reflected in the ledger, dashboard, and budget.
- A user can answer common spending questions without manually summing transactions.
- Assistant answers can be independently verified from displayed evidence.
- A user can quit and relaunch the desktop app without losing data and can find the backup/restore instructions.

## 12. Risks and mitigations

| Risk | Mitigation |
| --- | --- |
| Statement layouts vary or change | Start with one source; version its parser; expose row-level errors and keep fixtures synthetic/sanitized. |
| PDF extraction silently misreads a decimal/date | Preview every row and totals; block confirmation on structural errors; reconcile parsed rows against the statement detail where possible. |
| Duplicate-looking charges are legitimate | Auto-deduplicate only exact file repeats/high-confidence fingerprints; ask about ambiguous rows. |
| Missing months are mistaken for zero spending or complete coverage | Show statement coverage and gaps explicitly; never plot or answer an unimported month as zero. |
| Partial-year card history is mistaken for a complete financial picture | Show imported date coverage, year-to-date start date, and card-only scope in dashboards and chat. |
| Payments inflate spending | Classify payment/transfer rows separately and exclude them from spend aggregates by default. |
| Categorization feels confidently wrong | Make corrections visible, provide a rule option, and preserve manual overrides. |
| External AI leaks sensitive descriptions | No provider by default; make provider opt-in, disclose sent fields, minimize context, and offer local inference if practical. |
| Local API is accidentally exposed to the network | Bind only to loopback/private local transport, restrict access to the desktop shell, and include a release check that no LAN listener is enabled. |

## 13. Open decisions before implementation

1. **Chase statement format:** Should the first importer accept Chase Sapphire Preferred PDF statements, transaction CSV exports, or both? A representative file is needed to validate the import preview; a redacted copy is sufficient for planning.
2. **Statement retention:** Confirm the default of discarding source files after parsing while retaining normalized rows and import metadata.
3. **Desktop packaging:** macOS is the first target. Tauri is the current candidate for a React/Vite shell with the Python API bundled as a local sidecar; choose the shell and minimum macOS version.
4. **Assistant provider:** Should v1 require a configured external model, bundle/support a local model, or ship chat only when a provider is configured?
5. **Locale:** Use USD for amounts; choose the default date/number display locale and allow it to be configured.
These decisions should be resolved before the corresponding implementation slice. Simple monthly category budgets remain in P0 because they are a small, useful part of the Rocket Money-inspired experience; more advanced planning is out of scope. Other requirements can proceed independently.

## 14. Suggested implementation sequence

1. Confirm the Chase Sapphire Preferred file format and get a sanitized representative fixture.
2. Choose the desktop shell and minimum macOS version, then scaffold the React/Vite interface, local FastAPI process, SQLite persistence, and app-data location.
3. Build the import vertical slice: upload, parse, preview, duplicate handling, confirm, transaction ledger.
4. Add categories, manual corrections, merchant rules, and consistent spending calculations.
5. Add overview dashboard and transaction drill-down.
6. Add simple category budgets.
7. Add read-only assistant tools and evidence-backed responses over existing analytics services.
8. Add export/delete, backup guidance, desktop packaging, and a check that the API cannot be reached from the LAN before calling the release ready.

## 15. Product reference

Rocket Money is a product reference for spending dashboards, automatic transaction categorization, budgets, recurring-charge tracking, and conversational financial questions. Its broader capabilities also rely on linked accounts and services outside this MVP. BudgetAI uses these product patterns as inspiration but scopes v1 to uploaded card data and does not claim feature parity.

- [Rocket Money FAQ](https://www.rocketmoney.com/faq)
- [Rocket Money: How to get the most out of the app](https://help.rocketmoney.com/en/articles/1940551-how-to-get-the-most-out-of-rocket-money)
- [Rocket Money: What is Rowan?](https://help.rocketmoney.com/en/articles/16760933-what-is-rowan)
- [Rocket Money: Managing bills and subscriptions](https://help.rocketmoney.com/en/articles/2185531-managing-your-bills-and-subscriptions)
