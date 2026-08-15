---
description: Reverse-engineer large SQL stored procedures and their
  surrounding codebase into multiple complementary views covering
  database business logic, data lineage, operational execution,
  intermediate objects, cursors, business rules, database side effects,
  invocation/trigger architecture, and dependencies. Use when analyzing
  unfamiliar or legacy stored procedures, especially when procedures are
  hundreds of lines long or interact with temporary tables, views, CTEs,
  cursors, functions, jobs, services, repositories, or database
  triggers.
name: sql-procedure-reverse-engineering
---

# SQL Procedure & Database Reverse Engineering

## Purpose

Analyze SQL stored procedures as executable business processes rather
than as isolated SQL text.

The goal is to make large or unfamiliar procedures understandable by
reconstructing:

1.  The database business/data model.
2.  The movement and transformation of data.
3.  The operational execution flow.
4.  The role and lifecycle of intermediate objects.
5.  Cursor and iterative processing.
6.  Business rules encoded in the SQL.
7.  Persistent database side effects.
8.  How the procedure is invoked by the surrounding system.
9.  Upstream and downstream dependencies.

Do not default to a line-by-line explanation. First build a structured
understanding, then present the requested views.

## Core principles

### Evidence over inference

Separate conclusions into:

-   **Confirmed** --- directly established by source code, schema,
    configuration, or an explicit call site.
-   **Inferred** --- a reasonable interpretation supported by names,
    relationships, or behavior.
-   **Unknown** --- cannot be established from the available material.

Never present an inferred business meaning as confirmed fact.

### Treat intermediate objects as first-class entities

Temporary tables, table variables, CTEs, views, cursors, and
materialized views are part of the procedure's data model during
execution. Track their creation, population, transformation,
consumption, and lifetime.

### Preserve two levels of understanding

Always distinguish:

-   **Implementation logic** --- what the SQL literally does.
-   **Business interpretation** --- what that behavior appears to mean.

### Analyze beyond the procedure when source is available

If the procedure belongs to a codebase, inspect surrounding application
code, database objects, configuration, jobs, schedulers, triggers,
ORM/repository mappings, SQL files, and other procedures/functions as
needed to determine invocation and dependencies.

Do not assume the procedure is called by the application merely because
a similarly named method exists.

------------------------------------------------------------------------

# Analysis workflow

Use a multi-pass analysis for large procedures.

## Pass 1 --- Structural inventory

Identify:

-   Procedure name and schema.
-   Parameters and their apparent roles.
-   Return values/result sets/output parameters.
-   Local variables.
-   Temporary tables.
-   Table variables.
-   CTEs.
-   Views referenced.
-   Tables referenced.
-   Functions referenced.
-   Other procedures called.
-   Cursors.
-   Loops.
-   Conditional branches.
-   Transactions.
-   Savepoints.
-   Error/exception handling.
-   Dynamic SQL.
-   Temporary object cleanup.
-   External calls if present.

Create an internal inventory before writing the explanation.

## Pass 2 --- Segment the procedure

Divide the procedure into logical sections based on behavior rather than
arbitrary line ranges.

Typical sections may include:

-   Initialization.
-   Parameter validation.
-   Data loading.
-   Intermediate data preparation.
-   Business-rule evaluation.
-   Calculation.
-   Cursor/loop processing.
-   Persistent writes.
-   Aggregation.
-   Cleanup.
-   Commit/rollback.
-   Error handling.

Record source line ranges when line numbers are available.

## Pass 3 --- Build the object graph

Create relationships between:

-   Tables.
-   Views.
-   Temporary tables.
-   CTEs.
-   Variables.
-   Cursors.
-   Functions.
-   Procedures.
-   Application components.
-   Jobs/schedulers.
-   Database triggers.

Represent both dependency direction and data-flow direction where
useful.

## Pass 4 --- Trace data

For important business fields and result values, trace:

Source → filter/join → transformation → intermediate object →
calculation → destination.

Prioritize fields involved in:

-   Amounts.
-   Statuses.
-   IDs/keys.
-   Dates.
-   Flags.
-   Counts.
-   Business classifications.
-   Persistent writes.

Do not attempt exhaustive column-level lineage if the procedure is too
large unless requested. Start with business-significant fields.

## Pass 5 --- Reconstruct execution

Reconstruct the runtime sequence, including:

-   Branches.
-   Loops.
-   Cursor fetches.
-   Nested procedure/function calls.
-   Transaction boundaries.
-   Error paths.
-   Early exits.
-   Cleanup paths.

Where branches exist, explicitly show alternate paths.

## Pass 6 --- Trace invocation

If a codebase or database metadata is available, search for:

-   Direct procedure calls.
-   `EXEC` / `CALL` statements.
-   JDBC/ADO/database-client calls.
-   ORM stored-procedure mappings.
-   Repository/DAO methods.
-   SQL mapping files.
-   Service-layer calls.
-   Controllers/API endpoints.
-   Scheduled jobs.
-   Batch jobs.
-   Queue/message consumers.
-   Database triggers.
-   Other procedures/functions.
-   Configuration-driven execution.

Trace the call chain upward and downward.

Example:

Application endpoint → Controller → Service → Repository → Stored
Procedure

Also identify alternate invocation paths when more than one exists.

## Pass 7 --- Extract business rules

Convert important conditional logic into plain-language rules.

For every rule, preserve:

-   Condition.
-   Action/result.
-   Relevant tables/fields.
-   Source location.
-   Confidence level.

Example:

`customer_type = 'P'` → premium pricing path.

If `'P'` being "premium" is not explicitly documented, mark that
interpretation as inferred.

## Pass 8 --- Analyze impact

Determine:

-   Tables read.
-   Tables inserted into.
-   Tables updated.
-   Tables deleted from.
-   Columns changed where practical.
-   Temporary objects created/destroyed.
-   External side effects.
-   Transaction behavior.
-   Potential rollback boundaries.

------------------------------------------------------------------------

# Required analysis views

Unless the user requests a subset, provide these nine views.

## View 1 --- Database Business Model

Answer:

> What does the database represent in the context of this procedure?

Show:

-   Important entities/tables.
-   Primary/foreign-key relationships when known.
-   Parent/child relationships.
-   Important joins.
-   Views and their apparent purpose.
-   Central tables.
-   Relevant business concepts.

Prefer a compact relationship diagram when it improves comprehension.

Example:

``` text
Customer
   │
   ├── Account
   │      └── Transaction
   │
   └── Order
          ├── OrderItem
          │      └── Product
          └── Payment
```

Do not invent relationships that are not supported by the source.

## View 2 --- Data Flow / Lineage

Answer:

> Where does the important data come from, how is it transformed, and
> where does it go?

Show important paths such as:

``` text
orders
  ↓
#tmp_orders
  ↓
#tmp_calculated_orders
  ↓
cursor
  ↓
calculation
  ↓
orders.total_amount
```

For important values, identify:

-   Source.
-   Filters.
-   Joins.
-   Transformations.
-   Aggregations.
-   Intermediate objects.
-   Destination.

## View 3 --- Operational Execution Flow

Answer:

> What happens when the procedure runs?

Represent the procedure as an execution sequence.

Include:

-   Initialization.
-   Validation.
-   Reads.
-   Temporary-object creation.
-   Branches.
-   Loops.
-   Cursor processing.
-   Calls.
-   Writes.
-   Cleanup.
-   Commit/rollback.
-   Error paths.

Use a diagram when practical.

## View 4 --- Intermediate Object View

Answer:

> What purpose does each temporary or derived object serve?

For every significant temporary table, table variable, CTE, view, or
cursor, describe:

-   Purpose.
-   Created/defined at.
-   Populated from.
-   Important transformations.
-   Consumers.
-   Lifetime.
-   Final disposition.

Example:

``` text
#tmp_orders

Created: line 142
Populated from: orders, order_items
Modified: lines 180–230
Consumed by: cursor_order, UPDATE orders
Destroyed: line 390
```

## View 5 --- Cursor / Iterative Processing

Answer:

> Where does the procedure process individual records rather than
> operate set-wise?

For every cursor/loop:

-   Source query.
-   Rows/entities iterated over.
-   Per-row operations.
-   Nested calls.
-   Writes.
-   Exit condition.
-   Potential performance significance.

Explicitly call out row-by-row processing.

## View 6 --- Business Rules

Answer:

> What business decisions are encoded?

Translate complex conditions into concise rules.

For each rule:

``` text
Rule:
Condition:
Action:
Affected data:
Source:
Confidence:
```

Use `Confirmed`, `Inferred`, or `Unknown`.

## View 7 --- Database Impact / Side Effects

Answer:

> What persistent state can this procedure change?

Separate:

``` text
READ
WRITE
INSERT
UPDATE
DELETE
TEMPORARY
EXTERNAL SIDE EFFECTS
```

For updates, show important changed columns and their sources where
possible.

Example:

``` text
orders.total_amount
        ↑
#tmp_order_calculation.final_amount
        ↑
base_price + discount + tax
```

Also describe transaction and rollback behavior.

## View 8 --- Invocation / Trigger Architecture

Answer:

> What causes this procedure to execute?

If the codebase is available, identify all confirmed invocation paths.

Possible patterns:

``` text
HTTP Request
    ↓
Controller
    ↓
Service
    ↓
Repository
    ↓
Stored Procedure
```

``` text
Scheduler
    ↓
Batch Job
    ↓
Service
    ↓
Stored Procedure
```

``` text
INSERT
  ↓
Database Trigger
  ↓
Stored Procedure
```

``` text
Message Queue
    ↓
Consumer
    ↓
Service
    ↓
Stored Procedure
```

Search for direct and indirect invocation mechanisms.

Clearly distinguish confirmed call paths from likely or possible ones.

## View 9 --- Dependency Graph

Answer:

> What does this procedure depend on, and what depends on it?

Show downstream dependencies:

Procedure → Procedure/Function → View/Table

And upstream dependencies:

Endpoint/Job/Trigger → Application Component → Procedure

Include:

-   Procedures.
-   Functions.
-   Tables.
-   Views.
-   Application classes/modules.
-   Repositories/DAOs.
-   Jobs.
-   Schedulers.
-   Message consumers.
-   Triggers.
-   Configuration.

------------------------------------------------------------------------

# Large-procedure handling

For procedures hundreds or thousands of lines long:

1.  Do not summarize the entire procedure in one pass.
2.  Segment it into logical blocks.
3.  Analyze each block.
4.  Maintain a shared object/dependency inventory.
5.  Reconcile the blocks into global data and execution flows.
6.  Resolve references between sections.
7.  Only then produce the final views.

Never lose object identity between sections. For example, `#tmp_orders`
discovered in one section must remain the same object when referenced
later.

When context is insufficient, explicitly state which sections or
dependencies could not be resolved.

------------------------------------------------------------------------

# Codebase analysis rules

When source code is available:

1.  Search for the exact stored procedure name.
2.  Search for likely aliases/wrappers.
3.  Search for direct SQL execution.
4.  Search repository/DAO methods.
5.  Search ORM mappings.
6.  Search configuration and SQL mapping files.
7.  Trace callers upward.
8.  Trace callees downward.
9.  Inspect scheduled/batch execution.
10. Inspect database triggers and other database-side callers.
11. Record source locations for confirmed relationships.

Do not stop at the first call site if multiple invocation paths are
possible.

For every invocation path, report:

``` text
Trigger
→ Caller
→ Intermediate components
→ Stored procedure
```

Include file/class/method and line number when available.

------------------------------------------------------------------------

# Confidence and evidence

Use the following labels:

### Confirmed

Directly supported by source code, database metadata, configuration, or
explicit documentation.

### Inferred

Strongly suggested by naming, joins, behavior, or architecture but not
explicitly established.

### Unknown

Insufficient evidence.

Example:

``` text
customer_type = 'P'
→ Premium pricing path

Status: Confirmed

Interpretation:
'P' likely means Premium.

Status: Inferred
```

Never silently convert inference into fact.

------------------------------------------------------------------------

# Output format

When asked to analyze a procedure, use this order unless the user
requests otherwise:

## Executive Summary

Explain in 5--10 sentences:

-   What the procedure appears to accomplish.
-   Main business process.
-   Main tables.
-   Major intermediate objects.
-   Whether it is set-based or cursor/loop-heavy.
-   Major side effects.
-   How it appears to be invoked.
-   Major uncertainties.

## 1. Database Business Model

## 2. Data Flow / Lineage

## 3. Operational Execution Flow

## 4. Intermediate Objects

## 5. Cursor / Iterative Processing

## 6. Business Rules

## 7. Database Impact / Side Effects

## 8. Invocation / Trigger Architecture

## 9. Dependency Graph

## Risks / Complexity Hotspots

Call out:

-   Dynamic SQL.
-   Cursor-heavy processing.
-   Nested loops.
-   Large temporary tables.
-   Repeated scans.
-   Complex branching.
-   Hidden side effects.
-   Transaction scope.
-   Error-handling gaps.
-   Dependencies that could not be resolved.
-   Ambiguous business rules.

## Open Questions

List only questions that materially prevent understanding.

------------------------------------------------------------------------

# Diagram conventions

Prefer diagrams for relationships and flows when the structure is
complex.

Use:

-   `→` for execution/data flow.
-   `↑` for source lineage.
-   `↓` for downstream flow.
-   `├──` / `└──` for hierarchy.
-   Labels such as `[READ]`, `[UPDATE]`, `[INSERT]`, `[CALL]`,
    `[CURSOR]`.

Do not create diagrams merely for decoration.

------------------------------------------------------------------------

# What not to do

Do not:

-   Produce only a line-by-line explanation.
-   Assume table names reveal business meaning.
-   Assume a repository method is definitely the invocation path without
    evidence.
-   Ignore temporary tables because they are not persistent.
-   Ignore views or CTEs because they are read-only.
-   Treat cursors as ordinary queries.
-   Hide uncertainty.
-   Claim a procedure is triggered by a scheduler, API, trigger, or job
    without evidence.
-   Invent missing schema relationships.
-   Overload the reader with every SQL keyword or syntactic detail.

The objective is **comprehension of the system**, not transcription of
the SQL.

------------------------------------------------------------------------

# Final quality check

Before finalizing an analysis, verify:

-   [ ] All major tables are identified.
-   [ ] Important table relationships are explained.
-   [ ] Major data flows are traced.
-   [ ] Temporary tables are tracked from creation to disposal.
-   [ ] CTEs/views are explained when materially relevant.
-   [ ] Cursors and loops are explicitly identified.
-   [ ] Major business rules are extracted.
-   [ ] Persistent writes and side effects are identified.
-   [ ] Transaction/error behavior is explained.
-   [ ] Invocation paths were searched for when a codebase is available.
-   [ ] Upstream and downstream dependencies are identified.
-   [ ] Confirmed facts are separated from inference.
-   [ ] Unresolved dependencies are explicitly stated.
-   [ ] The final explanation gives both a high-level picture and enough
    detail to navigate back to the source.
