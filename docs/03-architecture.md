# Architecture

**English** | [Tiếng Việt](vi/03-architecture.md) · [Home](../README.md)

## Layer boundaries

Request flow: terminal input → CLI handler → application service → repository contract → storage adapter.

Services own business rules and depend on the contracts they consume. Adapters implement those contracts. The entry point selects and wires dependencies. Replacing JSON with SQLite must not change transfer validation or budget calculations.

## Proposed package layout

| Path | Responsibility |
|---|---|
| cmd/termfin/main.go | Startup, configuration, root context, dependency wiring, and exit status |
| internal/cli/ | Argument parsing, input conversion, service calls, and presentation |
| internal/account/ | Account types, validation, and use cases |
| internal/category/ | Category types, validation, and reference rules |
| internal/transaction/ | Income, expenses, transfers, and atomic-operation contracts |
| internal/budget/ | Budget limits and spending evaluation |
| internal/analytics/ | Monthly aggregation and reporting boundaries |
| internal/transferio/ | Import/export coordination, separate from account transfers |
| internal/storage/ | In-memory and durable adapters, storage transactions, and data versions |
| internal/domain/ | Shared domain types where needed, such as money and identifiers |
| pkg/ | Optional packages intentionally supported for external use |
| migrations/ | Ordered schema changes if SQLite is selected |
| test/ | Integration and CLI acceptance tests; unit tests live beside their packages |
| go.mod | Module identity and dependency declarations |

This is a proposed layout, not an existing directory structure. Add packages as responsibilities emerge. Neither pkg nor migrations is required when it has no purpose.

## CLI responsibilities

CLI handlers may parse arguments, convert textual inputs, report command syntax errors, call services, and format results.

Services determine reference validity, category compatibility, sufficient funds, arithmetic safety, atomic update boundaries, and budget/report inclusion. These rules must remain enforceable when services are called directly without a terminal.

## Repository contracts

Define behavior before writing interfaces:

1. What data must each service read and write?
2. Which errors must callers distinguish?
3. Are returned values detached copies or mutable references?
4. What do missing records and empty lists mean?
5. Is ordering deterministic?
6. Who generates IDs and enforces uniqueness?
7. Which reads and writes must share an atomic scope?

Prefer small consumer-owned interfaces, or a shared contracts package when several consumers genuinely need it. Keep SQL types, file paths, and serialization details out of domain-service contracts. Import/export paths belong at the I/O boundary.

## Atomic operations

A proposed unit-of-work contract lets a service execute related operations within one atomic scope. The service reads accounts, checks balances, and records a transaction using repositories bound to that scope. The adapter handles isolation, locking, commit, and rollback.

Reading balances outside this scope and starting a transaction only for the write leaves a stale-read window. Source and destination updates must never be independent saves. Import batches should reuse the same atomic mechanism.

## Dependency direction

- Shared domain types should have few dependencies.
- Services must not import concrete storage adapters.
- Storage may use domain types and implement service-facing contracts.
- Analytics reads through contracts rather than accessing files or CLI handlers.
- Resolve import cycles by reviewing ownership and boundaries before adding another abstraction.

## Context propagation

Pass context explicitly from the entry point through services and I/O operations. Do not use it to carry ordinary business parameters or store a request context long term in a service.

Cancellation checks are useful around waits and long-running work. Cancellation does not provide rollback by itself. Cancellation before commit should discard tentative changes. Once commit is known to have succeeded, late cancellation must not be reported as proof that nothing was saved. An indeterminate commit outcome needs an explicit error and a retry/reconciliation policy.

## Design tasks

- [ ] Map expense and transfer flows, including atomic boundaries.
- [ ] Specify repository results, errors, ownership, and ordering.
- [ ] Confirm the balance source of truth.
- [ ] Verify that services can be tested without the CLI or durable storage.
- [ ] Review the dependency graph for cycles.
- [ ] Describe which components change when adding a second storage adapter.
