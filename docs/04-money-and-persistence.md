# Money and Persistence

## Exact money representation

The proposed internal representation is a signed 64-bit integer in the smallest supported currency unit. For the initial whole-dong VND scope, 150000 means VND 150,000. Supporting fractional currencies later requires explicit currency and decimal-scale rules.

Parse input directly from decimal text to integer money. Do not convert through float64. Whole-dong input rejects fractions and unsupported grouping separators; grouping is a presentation concern.

Distinguish malformed input, values outside the representation range, and amounts rejected by domain rules. Guard addition, subtraction, and report aggregation against overflow before performing unsafe arithmetic. Taking the absolute value of the minimum signed integer also requires care.

### Tasks

- [ ] Define storage and display units centrally.
- [ ] Specify the accepted textual money format.
- [ ] Cover parsing, range limits, arithmetic overflow, and invalid transaction amounts.

## Balance source of truth

The proposed initial source of truth is opening balances plus committed transaction records. Current balances are derived. A transfer is one record containing both source and destination.

This avoids independently maintaining source balance, destination balance, and transaction history. Derived balances still require atomic read-check-write operations: concurrent expenses must not both spend the same available funds.

If cached balances are introduced later, update them alongside transaction history in one commit and provide reconciliation against the authoritative records.

## Atomic transfer sequence

1. Begin an atomic scope with the required write coordination.
2. Read source and destination state within that scope.
3. Validate references, distinct accounts, currency, amount, available funds, and overflow.
4. Prepare the complete transfer record.
5. Persist and commit once.
6. Report success only after commit succeeds.

Failure before commit must leave observable committed state unchanged. Saving the source first and compensating later is insufficient: the compensation can fail too.

## Storage choices

| Adapter | Main benefits | Responsibilities |
|---|---|---|
| In memory | Fast domain development and unit tests | Correct copy and atomicity semantics; no durability |
| JSON file | Simple inspection and file-I/O practice | Safe replacement, process locking, recovery, and data-version handling |
| SQLite | Structured queries, constraints, and database transactions | Driver selection, migrations, locking behavior, and integration coverage |

One durable adapter is sufficient for the initial release. An in-memory implementation alone does not satisfy persistence requirements.

## JSON adapter guidance

- Persist a consistent, versioned snapshot containing all state needed for the commit.
- Build candidate state separately from committed state. Shallow copies of nested maps or pointer-containing slices may still share mutable data.
- Write a temporary file on the same filesystem; complete writes, flush, and close before replacing the committed file with a platform-appropriate mechanism.
- Publish new in-memory state only after the file commit succeeds.
- Prevent competing processes from overwriting each other's state. An in-process mutex does not coordinate separate processes.
- Never delete the committed file first to make room for its replacement.
- Treat an existing corrupted file as an error, not an empty database.
- Define startup handling for temporary files, backups, and unsupported schema versions.

Atomic replacement and durability after power loss are separate properties. Verify replacement, flush, and filesystem metadata behavior on supported platforms before claiming crash safety. Injected write-failure tests alone do not establish power-loss durability.

## SQLite adapter guidance

- Read balances and record transactions within a database transaction using appropriate write coordination.
- Use constraints for references, unique IDs, and basic data validity; enable and verify foreign-key enforcement for relevant connections.
- Keep business validation in services. Database constraints provide additional protection.
- Roll back on pre-commit failure and check commit errors explicitly.
- Bound handling of busy/locked errors; avoid blindly retrying writes.
- Use ordered migrations, validate schema versions, and define backup/recovery procedures.

## Retries and uncertain outcomes

A process can stop after commit but before displaying success. Repeating the command may create a duplicate. A proposed mitigation is a stable operation ID: identical ID and equivalent content identify an existing operation; identical ID with different content is a conflict.

Generating a fresh ID for every retry does not solve duplication. Safe retry support requires a documented way to reuse the operation ID and inspect the outcome. Storage errors that leave commit status uncertain must be distinguishable from confirmed rollback.

## Acceptance tasks

- [ ] Failed writes preserve previously committed state.
- [ ] Restart restores equivalent data and reports.
- [ ] Corrupt or unsupported data is not silently overwritten.
- [ ] Competing expenses cannot both validate against stale funds.
- [ ] Adapter replacement preserves domain behavior.
