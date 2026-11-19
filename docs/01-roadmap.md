# Implementation Roadmap

This roadmap organizes the planned implementation into independently verifiable milestones. It does not imply that any milestone is complete.

## Proposed initial scope

- One local user and a single currency: VND, represented in whole dong.
- Nonnegative account balances; no debt, exchange rates, interest, or bank integrations.
- Recorded transactions are immutable in the initial version. Corrections require a separate design.
- Budgets inform users about spending; they do not block expenses.
- An in-memory repository supports domain development and unit tests. One durable adapter, JSON or SQLite, is required for the initial release.
- Writes are serialized. An enforced single-writer policy is acceptable for the initial version.

These are proposed constraints, not implemented capabilities. General-purpose currency support can be considered separately without changing the integer-money requirement.

### Scope decisions

- [ ] Confirm the initial currency and transaction-editing policy.
- [ ] Select the first durable storage adapter.
- [ ] Select the reporting timezone and its configuration mechanism.
- [ ] Confirm the domain rules before implementing services.
- [ ] Document how concurrent writes will be prevented or coordinated.

## Milestone 1: Domain foundations

- [ ] Establish the Go module and minimal package structure.
- [ ] Define money, account, and identifier types.
- [ ] Define repository contracts and error categories.
- [ ] Implement account creation with an in-memory repository.
- [ ] Add a thin CLI entry point and account validation tests.

Acceptance: an account can be created through a service independently of the CLI. Durable storage is not required at this milestone.

## Milestone 2: Core transactions

- [ ] Implement income and expense categories.
- [ ] Implement income and expense recording.
- [ ] Implement transfers within one atomic storage operation.
- [ ] Cover validation, insufficient funds, overflow, and commit failure.

Acceptance: successful transfers preserve total funds; failed transfers leave balances and transaction history unchanged.

## Milestone 3: Durable storage

- [ ] Implement a JSON or SQLite adapter against existing contracts.
- [ ] Restore committed state after restarting.
- [ ] Handle storage errors and corrupted or unsupported data explicitly.
- [ ] Enforce the selected writer-concurrency policy.

Acceptance: persistence does not require changing business rules, and failed writes do not expose partial updates.

## Milestone 4: Reporting and data exchange

- [ ] Implement monthly budgets and spending summaries.
- [ ] Implement monthly income and expense reports.
- [ ] Implement versioned JSON and CSV import/export.
- [ ] Document CLI inputs, outputs, and error behavior.

Acceptance: transfers are excluded from income and expense totals, and exports restore equivalent domain data into an empty store.

## Milestone 5: Release readiness

- [ ] Complete at least 30 meaningful unit-test cases.
- [ ] Complete storage integration tests and a CLI acceptance scenario.
- [ ] Add a GitHub Actions workflow that runs tests.
- [ ] Verify installation and usage documentation on a clean environment.
- [ ] Complete the definition-of-done checklist and select a license.

## Go concepts exercised

| Concept | Application |
|---|---|
| Variables, constants, functions | Input normalization and validation |
| Structs and methods | Accounts, transactions, budgets, and domain behavior |
| Pointers | Mutation boundaries and ownership |
| Arrays, slices, maps | Fixed-size values, transaction lists, and identifier lookup |
| Interfaces and composition | Repository contracts and replaceable adapters |
| Error handling | Distinguishing domain, parsing, cancellation, and storage failures |
| Package management | Cohesive packages and explicit dependencies |
| File I/O and JSON | Durable snapshots and exchange formats |
| context.Context | Cancellation across service and I/O boundaries |
| Table-driven tests | Valid, invalid, and boundary input cases |

Each milestone should make its inputs, outputs, failure behavior, and state transitions explicit. Review package dependencies and atomicity boundaries before expanding scope.
