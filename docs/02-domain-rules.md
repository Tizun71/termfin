# Domain Rules

**English** | [Tiếng Việt](vi/02-domain-rules.md) · [Home](../README.md)

This document proposes a consistent initial domain model. Changes to these rules should update examples and test expectations before implementation.

## Accounts

An account has a stable ID, a name, a currency, and an opening balance. Its current balance is derived from its opening balance and committed transactions.

- Trimmed names must not be empty.
- Proposed uniqueness: compare trimmed names without case distinctions. Preserve diacritics and document the Unicode comparison policy.
- References use stable IDs rather than names. Renaming must preserve relationships.
- Opening balances must be nonnegative and are not monthly income.
- Accounts referenced by transactions cannot be deleted in the initial version.

## Categories

A category has an ID, a name, and an income or expense kind.

- Names must be valid and unique within their kind under the chosen normalization policy.
- Income requires an income category; expenses require an expense category.
- Transfers have no income or expense category.
- Referenced categories cannot be deleted. Renaming preserves existing references.

## Transactions

Each transaction has an ID, a kind, a positive amount, a timestamp, an optional note, and the relevant account/category IDs. The initial currency is VND.

| Kind | Required references | Effect |
|---|---|---|
| Income | Receiving account and income category | Increases the account balance |
| Expense | Paying account and expense category | Decreases the account balance |
| Transfer | Distinct source and destination accounts | Decreases source and increases destination in one commit |

Common rules:

- Amounts must be positive and representable by the selected integer type.
- Referenced accounts and categories must exist.
- Expenses and transfers must not make the current source balance negative.
- Arithmetic must detect overflow, including additions to destination accounts.
- Timestamps must include timezone information. Proposed policy: reject future transactions using a replaceable clock for deterministic tests.
- Backdated entries are allowed under the proposed current-balance policy. This does not guarantee that every historical balance remains nonnegative.
- Enforcing nonnegative historical balances would require revalidating the affected transaction timeline and additional tests.

### Worked example

Wallet A opens with VND 1,000,000; Bank B opens with VND 0.

1. Record VND 2,000,000 income in B: A = 1,000,000; B = 2,000,000.
2. Record VND 100,000 food expense from A: A = 900,000; B = 2,000,000.
3. Transfer VND 300,000 from B to A: A = 1,200,000; B = 1,700,000.

Final total funds are VND 2,900,000. Reported income is VND 2,000,000 and expenses are VND 100,000. The transfer changes neither total.

## Budgets

The proposed budget scope is one expense category in one calendar month across all accounts.

- Unique key: expense category, month, and currency.
- Limits must be positive. Duplicate creation returns an error; changing a limit is a separate update operation.
- Spent equals qualifying expenses in the category and month.
- Remaining equals limit minus spent and may be negative.
- Spending equal to the limit reaches the limit; spending above it exceeds the limit.
- Display rounding must not determine whether a budget is exceeded.
- Income, transfers, and opening balances do not consume a budget.

## Monthly analytics

- Use one configured timezone to determine month membership.
- Include the start of the selected month and exclude the start of the following month.
- Return total income, total expenses, net income minus expenses, and expenses by category.
- Monthly net cash flow is distinct from current account balances.
- Empty months return zero totals and an empty category breakdown.
- All values in a report must come from one consistent data snapshot.

## Error categories

Distinguish invalid input, missing references, duplicates, insufficient funds, overflow, write conflicts, cancellation, corrupted persisted data, and I/O or database failure.

Services return errors without printing terminal output. CLI handlers translate errors into useful messages. Repositories must not report failed writes as successful operations.

## Open decisions

- [ ] Confirm name normalization and uniqueness semantics.
- [ ] Confirm future and backdated transaction policies.
- [ ] Specify timezone configuration and month input format.
- [ ] Confirm the initial editing and deletion restrictions.
- [ ] Add acceptance criteria for any scope extension.
