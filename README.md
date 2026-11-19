# Termfin

Termfin is a planned terminal-based personal finance application written in Go. Its design covers multiple accounts, income and expenses, transfers, categories, budgets, monthly reports, and CSV/JSON data exchange.

**Status: design and implementation guidance only.** This repository currently contains documentation. Application code, a Go module, persistent storage, and CI workflows have not been implemented. Checklists describe planned work, not available features.

## Planned capabilities

| Module | Scope |
|---|---|
| Accounts | Multiple accounts, opening balances, and current balances |
| Transactions | Income, expenses, and atomic transfers |
| Categories | Separate income and expense classifications |
| Budgets | Monthly category limits and spending summaries |
| Analytics | Monthly income, expenses, and category breakdowns |
| Import/export | Versioned CSV and JSON formats |
| Persistence | A file-based or SQLite adapter behind repository interfaces |

## Documentation

| Document | Purpose |
|---|---|
| [Implementation roadmap](docs/01-roadmap.md) | Scope decisions, milestones, and Go concepts |
| [Domain rules](docs/02-domain-rules.md) | Validation, balances, transfers, budgets, and reporting |
| [Architecture](docs/03-architecture.md) | Package responsibilities and repository contracts |
| [Money and persistence](docs/04-money-and-persistence.md) | Integer amounts, atomic writes, and storage options |
| [CLI and data exchange](docs/05-cli-and-data-exchange.md) | Command behavior and CSV/JSON contracts |
| [Testing strategy](docs/06-testing.md) | Forty proposed unit-test scenarios and integration coverage |
| [Definition of done](docs/07-definition-of-done.md) | Acceptance criteria, CI requirements, and release documentation |

Read the domain rules and architecture before implementing a feature. Design proposals are identified as such; unresolved decisions appear as unchecked tasks. Update the relevant specification and tests whenever a rule changes.

## Design goals

- Keep business rules independent of terminal input and storage technology.
- Represent money exactly and reject invalid or overflowing amounts.
- Commit transfers and import batches completely or leave committed state unchanged.
- Provide predictable errors, reproducible tests, and documented data formats.
- Keep the implementation small enough to understand and extend.

The project also provides a practical setting for Go fundamentals: functions, structs, methods, pointers, collections, interfaces, composition, error handling, packages, file I/O, JSON encoding, context propagation, and table-driven testing.

## Contributing

Documentation improvements, edge cases, and design feedback are useful at this stage. For a proposed change, describe the problem, affected rules, expected behavior, and acceptance criteria. Keep specifications and examples consistent across documents.

Implementation contributions should follow the documented layer boundaries and include tests for affected behavior. Installation instructions and executable examples should be added when a working application is available.

## Licensing

An open-source license has not yet been selected or added. Selecting a license and adding its text is a prerequisite for publishing the project as a licensed open-source release.
