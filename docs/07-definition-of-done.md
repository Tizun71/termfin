# Definition of Done

**English** | [Tiếng Việt](vi/07-definition-of-done.md) · [Home](../README.md)

This checklist defines the proposed initial release criteria. All items remain pending until supported by implementation and verification evidence.

## Functional scope

- [ ] Multiple accounts can be created and listed with correct balances.
- [ ] Income and expense categories follow documented validation rules.
- [ ] Income and expenses are validated and recorded correctly.
- [ ] Transfers are atomic and do not inflate income or expense reports.
- [ ] Budgets can be created and updated, with spent and remaining amounts.
- [ ] Monthly reports respect configured timezone and boundary rules.
- [ ] JSON import/export follows the published exchange contract.
- [ ] CSV import/export follows the published exchange contract.
- [ ] Committed data survives restart.

## Architecture and correctness

- [ ] CLI handlers and main.go contain no business logic.
- [ ] Repository interfaces isolate services from concrete storage adapters.
- [ ] Money never passes through floating-point representations.
- [ ] Amounts, balances, references, category kinds, timestamps, and overflow are validated.
- [ ] Failures before commit do not expose partial changes.
- [ ] Invalid import batches commit no additions.
- [ ] Duplicate IDs, retries, and uncertain outcomes have documented behavior.
- [ ] Context propagation and rollback behavior are explicit.
- [ ] The selected writer-concurrency policy is enforced.

## Testing

- [ ] At least 30 distinct, meaningful unit-test cases have named expectations.
- [ ] Table-driven tests cover validation and domain boundaries.
- [ ] Transfers cover success, insufficient funds, overflow, and commit failure.
- [ ] Analytics tests exclude transfers and opening balances from income/expenses.
- [ ] Integration tests cover reopening storage and both exchange-format round trips.
- [ ] Tests run on isolated synthetic data without machine-specific paths.

## GitHub Actions requirements

No workflow currently exists. The implementation milestone should add a workflow that:

1. Runs for pushes and pull requests.
2. Checks out the repository and selects a Go version compatible with the module declaration.
3. Resolves declared dependencies.
4. Checks formatting, runs appropriate static analysis, and executes unit tests.
5. Runs integration tests for the selected adapter with explicit platform/driver requirements.
6. Fails when a required check fails and provides useful logs.

### CI tasks

- [ ] Add the workflow under .github/workflows when implementation begins.
- [ ] Verify a successful run on GitHub.
- [ ] Confirm a deliberately failing test causes a failed workflow, then restore the test.
- [ ] Test supported platforms where filesystem or driver behavior differs, including Windows if claimed as supported.

## Installation and usage documentation

Installation commands are intentionally absent while the application is unimplemented. Before release, document and verify:

- [ ] Required Go version, supported operating systems, and actual dependencies.
- [ ] Repository checkout, dependency resolution, build, and run steps in a clean environment.
- [ ] Storage selection, default data location, configuration, and timezone behavior.
- [ ] A complete example using synthetic financial data.
- [ ] Commands, arguments, money/time formats, and exit statuses.
- [ ] Import/export file requirements and duplicate-record policy.
- [ ] Backup, restoration, and corrupted-data handling.
- [ ] Limits for currency support, backdating, concurrency, transaction changes, and verified crash durability.
- [ ] Unit and integration test instructions.

Only advertise behavior that has been implemented and verified.

## Manual acceptance scenario

Use a fresh test store:

1. Create Wallet A with VND 1,000,000 and Bank B with VND 0.
2. Create Salary as an income category and Food as an expense category.
3. Record VND 2,000,000 income in B and VND 100,000 food expense from A in the same month.
4. Transfer VND 300,000 from B to A.
5. Verify A = 1,200,000; B = 1,700,000; monthly income = 2,000,000; expenses = 100,000.
6. Set the monthly Food budget to VND 500,000 and verify VND 400,000 remains.
7. Attempt a transfer exceeding B's balance; verify unchanged balances and history.
8. Restart and verify the same results.
9. Export/import JSON into an empty store; repeat CSV with another empty store.
10. Reimport identical data and verify no duplicates. Import an invalid batch and verify zero committed additions.

## Publication checklist

- [ ] Select an open-source license and add its full text in a LICENSE file.
- [ ] Ensure the README accurately distinguishes available and planned capabilities.
- [ ] Keep contribution guidance consistent with the repository's actual tooling.
- [ ] Publish only synthetic examples and fixtures; exclude financial data, credentials, and local configuration.
- [ ] Resolve contradictory specifications and document remaining limitations before release.

## Maintainability review

- [ ] Money representation and arithmetic limits are explainable from the documentation.
- [ ] Commit and rollback boundaries are identifiable in transfer and import flows.
- [ ] A validation rule can be changed with focused service tests.
- [ ] Adding a storage adapter does not require rewriting business rules.
- [ ] Parsing, domain, cancellation, and persistence failures remain distinguishable.
- [ ] A new report can use existing read contracts without bypassing layer boundaries.
