# Testing Strategy

**English** | [Tiếng Việt](vi/06-testing.md) · [Home](../README.md)

## Unit-test target

The initial release requires at least 30 meaningful unit-test cases. The 40 proposed scenarios below provide a starting plan; none is implemented yet. Related cases may use named subtests in table-driven tests. Report whether counts refer to functions, subtests, or scenarios, and avoid duplicating cases solely to reach a number.

Unit tests should not require real files, SQLite, or a running CLI. Use an in-memory repository or fake with explicit failure injection. Test doubles must respect the repository's atomicity contract; otherwise, passing tests can conceal rollback defects.

## Proposed unit scenarios

| # | Area | Scenario and expected result |
|---|---|---|
| 01 | Money | Valid integer text parses to the correct whole-dong amount |
| 02 | Money | Fractional input is rejected under the initial VND scope |
| 03 | Money | Nonnumeric input is rejected |
| 04 | Money | Values outside the integer range are rejected |
| 05 | Accounts | Valid creation assigns an ID and preserves the opening balance |
| 06 | Accounts | Whitespace-only names are rejected |
| 07 | Accounts | Duplicate normalized names are rejected |
| 08 | Accounts | Negative opening balances are rejected |
| 09 | Categories | Valid income and expense categories can be created |
| 10 | Categories | Unsupported kinds are rejected |
| 11 | Categories | Duplicate normalized names within a kind are rejected |
| 12 | Income | Valid income increases the balance and records one transaction |
| 13 | Income | A missing account produces an error without a write |
| 14 | Income | An expense category is rejected without a write |
| 15 | Income | Balance overflow is rejected without a write |
| 16 | Transactions | Zero and negative amounts are rejected as separate inputs |
| 17 | Expenses | A valid expense reduces the balance and records one transaction |
| 18 | Expenses | Spending the exact available balance succeeds |
| 19 | Expenses | Insufficient funds leave balance and history unchanged |
| 20 | Expenses | A missing or incompatible category produces no write |
| 21 | Transfers | Success updates both balances and preserves total funds |
| 22 | Transfers | Identical source and destination are rejected |
| 23 | Transfers | A missing account leaves all state unchanged |
| 24 | Transfers | Insufficient source funds leave both balances and history unchanged |
| 25 | Transfers | Destination overflow leaves the source unchanged |
| 26 | Atomicity | Injected confirmed commit failure preserves all prior state |
| 27 | Context | Cancellation before commit returns a cancellation error without mutation |
| 28 | Budgets | A valid expense-category/month budget can be created |
| 29 | Budgets | Duplicate category/month/currency creation is rejected |
| 30 | Budgets | Nonpositive limits and income categories are rejected |
| 31 | Budgets | Spending exactly the limit leaves zero remaining without exceeding it |
| 32 | Budgets | Spending above the limit produces negative remaining and exceeded status |
| 33 | Analytics | An empty month returns zero totals and an empty breakdown |
| 34 | Analytics | Transfers and opening balances do not count as income or expenses |
| 35 | Analytics | A transaction at the month start is included |
| 36 | Analytics | A transaction at the following month start is excluded |
| 37 | Analytics | Timezone conversion assigns a boundary transaction to the correct month |
| 38 | Import | One invalid record prevents the entire batch from committing |
| 39 | Import | Reimporting an existing ID with equivalent content creates no duplicate |
| 40 | Import | An existing ID with different content rejects the entire batch |

For failures, assert the error category and the absence of unintended state changes. Checking only that an error exists is insufficient.

## Table-driven test structure

Define each case with a descriptive name, initial state, input, expected output/error, and expected final state.

Expense examples, in VND:

- Available 100,000; spend 40,000: remaining 60,000 and one added transaction.
- Available 100,000; spend 100,000: remaining 0 and one added transaction.
- Available 100,000; spend 100,001: insufficient-funds error, original balance, and no added transaction.

Each subtest creates isolated state. Avoid shared mutable maps, slices, repositories, or ID generators. Control clocks and identifiers so that results do not depend on wall time or execution order.

## Additional coverage

- [ ] Future timestamps are rejected against a controlled clock.
- [ ] Report aggregation detects overflow.
- [ ] Category renaming preserves transaction references.
- [ ] CSV handles quotes, commas, Unicode, and multiline notes.
- [ ] JSON preserves large integer money values exactly.
- [ ] Missing columns, unknown fields, and unsupported versions produce clear errors.
- [ ] Import resolves batch references independently of row order and rejects repeated IDs.
- [ ] Timezone metadata conflicts are rejected without mutation.
- [ ] Late cancellation does not misreport a known successful commit as unsaved.
- [ ] Uncertain commit outcomes follow the documented reconciliation policy.

## Integration coverage

These tests use real adapters and do not replace the unit-test requirement:

- [ ] Persist state, close storage, reopen, and verify equivalent data and reports.
- [ ] JSON: corrupted input is not treated as an empty store; write failure preserves committed data.
- [ ] SQLite: failure inside a transaction rolls back changes; foreign-key enforcement is verified.
- [ ] Competing expenses cannot overspend; rejecting a second writer is acceptable under an enforced single-writer policy.
- [ ] JSON export/import into an empty store preserves references, balances, and reports.
- [ ] CSV export/import into an empty store preserves references, balances, and reports.
- [ ] CLI acceptance covers accounts, income, expenses, transfers, budgets, reports, and restart.

Run adapter-specific tests only for implemented adapters. When a second adapter is added, run shared repository contract tests against both implementations.

## Completion evidence

- [ ] Record meaningful unit-case counts and integration results.
- [ ] Include invalid inputs, storage failures, month boundaries, and rollback cases.
- [ ] Verify that deliberately breaking a rule causes its corresponding test to fail.
- [ ] Use temporary fixtures without real financial data or machine-specific paths.
