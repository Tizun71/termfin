# CLI and Data Exchange

## Command behavior

Define command inputs, outputs, and failure cases before selecting a CLI library. Command handlers delegate domain decisions to services.

| Operation | Main inputs | Expected output |
|---|---|---|
| Create/list accounts | Name and opening balance | ID, name, balance, and currency |
| Create/list categories | Name and kind | ID, name, and kind |
| Record income | Account, category, amount, timestamp | Transaction ID and details |
| Record expense | Account, category, amount, timestamp | Transaction details and resulting balance |
| Transfer funds | Source, destination, amount, timestamp | One transfer ID and both account references |
| List transactions | Optional account/month filters | Deterministically ordered records |
| Create/update budget | Expense category, month, limit | Limit, spent, and remaining |
| Monthly report | Month | Income, expenses, net amount, and category breakdown |
| Import/export | Format and path | Record counts or actionable errors |

Proposed input conventions: months use YYYY-MM; timestamps include date, time, and UTC offset; VND amounts use integer text without grouping separators.

Use standard output for requested results and standard error for diagnostics. Exit status 0 means success; document nonzero statuses. Sort listings deterministically, for example by timestamp followed by ID. Exact command syntax remains to be specified once the CLI design is selected.

### Tasks

- [ ] Specify command names, required arguments, and help text.
- [ ] Separate syntax errors from domain failures.
- [ ] Provide actionable error messages.
- [ ] Ensure every command delegates business rules to a testable service.

## Shared exchange contract

The proposed scope includes accounts, categories, transactions, and budgets in both formats. A transaction-only export must not be described as a complete backup.

Each format must define schema version, UTF-8 encoding, currency units, stable IDs, references, required and optional fields, unknown-field behavior, timestamps, timezone, duplicate handling, and import atomicity.

Proposed initial policy: reject unknown fields or columns to catch misspellings. Future compatibility changes should be versioned and tested.

## JSON format

Use a versioned root document with collections for accounts, categories, transactions, and budgets, plus the reporting timezone and currency metadata. Include authoritative fields required for restoration; derived display balances should not become a second source of truth.

The proposed exchange representation for money is a decimal integer string, preventing intermediary tools from silently rounding large amounts. This differs from the internal integer type. Decoding must not pass money through floating-point values.

Keep exchange models independent of adapter-specific storage structures so that changing storage does not change the public file contract.

## CSV format

The proposed complete export is a directory containing these files:

| File | Columns |
|---|---|
| metadata.csv | schema_version, currency, timezone |
| accounts.csv | id, name, currency, opening_balance |
| categories.csv | id, name, kind |
| transactions.csv | id, kind, account_id, source_account_id, destination_account_id, category_id, amount, occurred_at, note |
| budgets.csv | id, category_id, month, currency, limit |

metadata.csv contains one data row. Empty collections still have headers. Money fields contain unformatted integer text.

Income and expenses use account_id and category_id. Transfers use source_account_id and destination_account_id. Nonapplicable fields must be empty and validated as such.

Use a CSV parser/encoder rather than splitting lines on commas. Notes may contain commas, quotes, and line breaks. Proposed encoding policy: accept an optional UTF-8 BOM when reading and omit it when writing.

## Import process

1. Validate schema version, encoding, headers, and document structure.
2. Parse into intermediate records; errors identify the file, record/line, or field path where appropriate.
3. Detect duplicate IDs within the batch and compare against existing data.
4. Build candidate state and resolve references, including those introduced by the same batch.
5. Apply domain validation within the atomic scope.
6. Commit the entire batch once.

### Proposed conflict and validation policy

- Import adds data rather than silently replacing the existing store.
- Existing ID with equivalent domain content: skip and report separately.
- Existing ID with different content: reject the entire batch.
- Repeated ID within the batch: reject the batch.
- Do not overwrite opening balances or existing transactions.
- Validate balances against the combined candidate state, independent of row order. This follows the current-balance policy and does not guarantee nonnegative historical balances.
- A partial-history export may not be independently importable under nonnegative-balance rules. Complete backups include all authoritative data.
- Validate timezone metadata. Reject an import into a nonempty store if its reporting timezone conflicts; restoring into an empty store adopts the exported timezone.

Equivalent content means normalized domain values, not byte equality or JSON property order. A failed batch reports zero committed additions.

## Export process

- Read one consistent snapshot.
- Use deterministic record ordering.
- For CSV, prepare a new export directory and report success only after every required file is complete. Import rejects incomplete bundles.
- Do not silently overwrite existing output. Any overwrite option must be explicit and implemented safely.
- Verify export → import into an empty store by comparing domain data, references, balances, and reports.

## Remaining specification tasks

- [ ] Define field types, accepted enum values, length limits, and conditional requirements.
- [ ] Define file-size and record-count limits.
- [ ] Specify version compatibility and timezone restoration behavior.
- [ ] Cover Unicode names, quotes, commas, and multiline notes.
- [ ] Specify added/skipped counts and structured error details.
