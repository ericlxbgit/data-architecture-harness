---
trigger: always_on
description: SQL Server conventions — naming, data types, DDL patterns
---

# SQL Server Conventions

The companion to `10-data-modeling-3nf.md`. Modeling decisions there;
SQL-Server-specific implementation here.

## Casing and language

- **PascalCase** for all object names: tables, columns, schemas, views,
  stored procedures, functions. This is the Microsoft / AdventureWorks
  convention and is consistent with most of the SQL Server ecosystem.
- **Singular** for table names: `Customer`, not `Customers`. The table
  represents a *kind of thing*; one row is one of them.
- No reserved words as identifiers (`User`, `Order`, `Group`, `Key`,
  `Date`). If you can't avoid `Order` for business reasons, brace it as
  `[Order]` in every reference and document the choice. Better: rename to
  `SalesOrder`, `PurchaseOrder`, etc., so the bracket isn't needed.
- No spaces, no hyphens, no special characters in identifiers.
- No Hungarian-style prefixes: **no** `tbl_`, `vw_`, `sp_`.
  - `tbl_` is redundant — `sys.objects` already tells you the type.
  - `vw_` is also redundant for the same reason.
  - `sp_` on stored procs is actively harmful — SQL Server treats that
    prefix as a hint to look in `master` first, slowing every call. Use
    `usp_` instead if a prefix is required by your team.

## Schemas

Use schemas to group related tables by **bounded context**, not by
infrastructure layer. Examples:

- `Sales`, `Inventory`, `Billing`, `HR`, `Identity` — good.
- `Staging`, `Reporting` — acceptable as separate, clearly purpose-specific
  schemas.
- Avoid dumping everything into `dbo`. `dbo` is a smell, not a default.

Create the schema explicitly:

```sql
CREATE SCHEMA Sales AUTHORIZATION dbo;
GO
```

## Table and column naming

| Object kind        | Pattern                              | Example                           |
|--------------------|--------------------------------------|-----------------------------------|
| Table              | `Schema.EntityName` (singular)       | `Sales.Customer`                  |
| Junction table     | `Schema.Parent1Parent2`              | `Sales.OrderProduct`              |
| Surrogate PK       | `<Entity>ID`                         | `CustomerID`                      |
| Foreign key column | Same name as the referenced PK       | `CustomerID` (in `Order` table)   |
| Self-referencing FK| `<Role><Entity>ID`                   | `ManagerEmployeeID`               |
| Boolean (`BIT`)    | `Is...` / `Has...`                   | `IsActive`, `HasSubscription`     |
| Timestamp          | `...At` (`DATETIME2`)                | `CreatedAt`, `UpdatedAt`          |
| Date               | `...Date` (`DATE`)                   | `OrderDate`, `BirthDate`          |
| Count              | `...Count`                           | `ItemCount`                       |
| Monetary           | `...Amount<Currency>` (`DECIMAL`)    | `TotalAmountUSD`                  |

## Object naming for constraints, indexes, etc.

| Object             | Pattern                                | Example                            |
|--------------------|----------------------------------------|------------------------------------|
| Primary key        | `PK_<Table>`                           | `PK_Customer`                      |
| Foreign key        | `FK_<Child>_<Parent>[_<Role>]`         | `FK_Order_Customer`                |
| Unique constraint  | `UQ_<Table>_<Columns>`                 | `UQ_Customer_Email`                |
| Check constraint   | `CK_<Table>_<Rule>`                    | `CK_Order_TotalAmountNonNegative`  |
| Default constraint | `DF_<Table>_<Column>`                  | `DF_Customer_CreatedAt`            |
| Index (non-clust.) | `IX_<Table>_<Columns>`                 | `IX_Order_CustomerID`              |
| Index (filtered)   | `IX_<Table>_<Columns>_<Predicate>`     | `IX_Order_OrderDate_NotDeleted`    |
| View               | `<Entity>View` or `v<Entity>`         | `ActiveCustomerView`               |
| Stored procedure   | `usp_<Action><Entity>`                 | `usp_CreateOrder`                  |
| Trigger            | `tr_<Table>_<Action>`                  | `tr_Order_AfterUpdate`             |

**Always name your constraints explicitly.** SQL Server will auto-generate
names like `PK__Customer__A4AE64D8...` if you don't — those are unstable
across environments, painful to script, and impossible to reference cleanly
in migrations.

## Data types — the short list

Use these. Justify in a comment if you use anything else.

| Need                          | Use                            | Avoid                                       |
|-------------------------------|--------------------------------|---------------------------------------------|
| Surrogate key (small/medium)  | `INT IDENTITY(1,1)`            | `UNIQUEIDENTIFIER` as clustered PK          |
| Surrogate key (>2B rows)      | `BIGINT IDENTITY(1,1)`         |                                             |
| Distributed/merge-friendly ID | `UNIQUEIDENTIFIER` non-clust.  | `NEWID()` as the clustered key (page splits)|
| Short text (fixed alphabet)   | `VARCHAR(n)`                   | `VARCHAR(MAX)` when `n` would do            |
| User-facing text              | `NVARCHAR(n)`                  | `VARCHAR` (loses non-ASCII)                 |
| Large text                    | `NVARCHAR(MAX)`                | `TEXT`, `NTEXT` (deprecated)                |
| Money                         | `DECIMAL(19,4)`                | `MONEY`, `FLOAT`, `REAL`                    |
| Generic decimal               | `DECIMAL(p,s)` with stated p,s | `NUMERIC` (synonym; pick one)               |
| Date only                     | `DATE`                         | `DATETIME` truncated by convention          |
| Date + time (UTC)             | `DATETIME2(3)` or `DATETIME2(7)` | `DATETIME` (lower precision, larger)      |
| Time-zone-aware               | `DATETIMEOFFSET`               | `DATETIME2` + a string zone column          |
| Boolean                       | `BIT NOT NULL`                 | `CHAR(1)` with `'Y'/'N'`                    |
| JSON document                 | `NVARCHAR(MAX)` + `ISJSON` CHECK| Unconstrained `NVARCHAR(MAX)`              |
| Binary blob                   | `VARBINARY(MAX)` (or FILESTREAM)| `IMAGE` (deprecated)                       |

**Why these rules matter** (the non-obvious ones):

- `MONEY` rounds in surprising ways and has only 4 decimal places of
  precision — use `DECIMAL(19,4)` and be explicit.
- `FLOAT`/`REAL` are approximate. They will silently produce wrong totals.
- `DATETIME` has ~3.33ms precision and is 8 bytes; `DATETIME2(3)` has true
  millisecond precision in 7 bytes. There is no reason to choose `DATETIME`
  in new schemas.
- `VARCHAR` cannot store non-ASCII reliably. Use `NVARCHAR` for any
  human-entered text.
- `UNIQUEIDENTIFIER` as the clustered key causes massive page splits
  because `NEWID()` is unordered. If you must use GUIDs, either use
  `NEWSEQUENTIALID()` or keep the GUID as a non-clustered unique key
  alongside an `INT IDENTITY` clustered key.

## NULL handling

- Default to `NOT NULL`. Make nullability an explicit decision.
- For nullable columns, document the meaning of NULL in a comment — is it
  "unknown," "not applicable," or "not yet"? These are different things.
- `ISNULL(x, default)` is fine for SQL Server; `COALESCE(x, default)` is
  more portable. Pick one and stay consistent.
- Never compare with `= NULL`. Use `IS NULL`.

## Audit columns — required on every business table

```sql
CreatedAt   DATETIME2(3) NOT NULL CONSTRAINT DF_<Table>_CreatedAt   DEFAULT SYSUTCDATETIME(),
CreatedBy   NVARCHAR(128) NOT NULL CONSTRAINT DF_<Table>_CreatedBy  DEFAULT SUSER_SNAME(),
UpdatedAt   DATETIME2(3) NOT NULL CONSTRAINT DF_<Table>_UpdatedAt   DEFAULT SYSUTCDATETIME(),
UpdatedBy   NVARCHAR(128) NOT NULL CONSTRAINT DF_<Table>_UpdatedBy  DEFAULT SUSER_SNAME(),
```

`UpdatedAt` / `UpdatedBy` are maintained by an `AFTER UPDATE` trigger or by
the application layer — pick one approach per project and apply it
uniformly.

Exempted from audit columns: pure lookup / reference tables.

## Indexing — only the rules a modeler must follow

The DBA owns most index choices. You own these defaults:

- **Every PK is clustered** unless there's a documented reason otherwise.
  (Common reason: a GUID PK, where the clustered key should be a separate
  IDENTITY.)
- **Every FK column has a non-clustered index.** SQL Server does not
  create one automatically. Without it, deletes and joins against the
  parent are slow and prone to lock escalation.
- **Filtered index on `IsDeleted = 0`** for any soft-delete table that has
  hot queries on live rows.
- **Filtered unique index** to enforce conditional uniqueness (e.g. "one
  current row per natural key").
- Do not add indexes "just in case." Each index has a write cost.

## DDL template — a properly-formed table

```sql
CREATE TABLE Sales.Customer (
    CustomerID      INT             IDENTITY(1,1) NOT NULL,
    AccountNumber   VARCHAR(20)     NOT NULL,
    EmailAddress    NVARCHAR(254)   NOT NULL,
    FirstName       NVARCHAR(50)    NOT NULL,
    LastName        NVARCHAR(50)    NOT NULL,
    BirthDate       DATE            NULL,        -- NULL = not provided
    IsActive        BIT             NOT NULL CONSTRAINT DF_Customer_IsActive  DEFAULT 1,
    CreatedAt       DATETIME2(3)    NOT NULL CONSTRAINT DF_Customer_CreatedAt DEFAULT SYSUTCDATETIME(),
    CreatedBy       NVARCHAR(128)   NOT NULL CONSTRAINT DF_Customer_CreatedBy DEFAULT SUSER_SNAME(),
    UpdatedAt       DATETIME2(3)    NOT NULL CONSTRAINT DF_Customer_UpdatedAt DEFAULT SYSUTCDATETIME(),
    UpdatedBy       NVARCHAR(128)   NOT NULL CONSTRAINT DF_Customer_UpdatedBy DEFAULT SUSER_SNAME(),

    CONSTRAINT PK_Customer              PRIMARY KEY CLUSTERED (CustomerID),
    CONSTRAINT UQ_Customer_AccountNumber UNIQUE (AccountNumber),
    CONSTRAINT UQ_Customer_EmailAddress  UNIQUE (EmailAddress),
    CONSTRAINT CK_Customer_EmailFormat   CHECK (EmailAddress LIKE '%_@_%._%')
);
GO

-- Index every FK column (none here yet — add as relationships are introduced).

-- Document the entity at the database level.
EXEC sys.sp_addextendedproperty
    @name = N'MS_Description',
    @value = N'A customer who can place orders. Identified externally by AccountNumber, internally by CustomerID.',
    @level0type = N'SCHEMA', @level0name = N'Sales',
    @level1type = N'TABLE',  @level1name = N'Customer';
GO
```

Key points to copy when generating new tables:

- Explicit `CLUSTERED` on the PK.
- Every constraint named.
- `DATETIME2(3)`, not `DATETIME`.
- `NVARCHAR` for user-facing text.
- `SYSUTCDATETIME()`, not `GETDATE()` (which returns local server time).
- Extended properties for entity-level documentation — they're queryable
  and travel with the schema.

## Schema change patterns (SQL Server specifics)

- **Add column**: `ALTER TABLE ... ADD <col> ... NULL` is metadata-only and
  fast. `... NOT NULL DEFAULT <value>` is *also* metadata-only in modern
  SQL Server when the default is constant — but verify on your version.
- **Drop column**: `ALTER TABLE ... DROP COLUMN ...` is metadata-only but
  the space is not reclaimed until rebuild. Schedule a rebuild on big tables.
- **Rename**: use `sp_rename` only as the *third* phase of an
  expand–migrate–contract migration, never in place on a contracted table.
- **Change type**: never in place on a contracted table. Add a new column,
  backfill, deprecate, drop.

## What "done" looks like for a modeling change

- [ ] Entity definition stated in one sentence (in extended property or
      comment).
- [ ] PK declared, clustered.
- [ ] All FKs declared as constraints and indexed.
- [ ] Every constraint named explicitly.
- [ ] Audit columns present (or absence justified — lookup table).
- [ ] No deprecated types (`DATETIME`, `MONEY`, `TEXT`, `NTEXT`, `IMAGE`).
- [ ] CHECK constraints for every business invariant on the row.
- [ ] At least one example query and one example insert verified.
