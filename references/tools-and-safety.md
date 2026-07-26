# Tools And Safety Reference

Use this reference before selecting handler groups or calling mutating SAP ABAP ADT tools.

## Handler Groups

`search` is always included. `system` is included when `readonly` is included. The default exposition is `readonly,high`, but for production or unknown systems prefer `readonly`.

Common exposure combinations:

- `readonly`: safest; repository inspection and system/read tools.
- `readonly,high`: development mode; ADT-backed create/update/delete/check/activation-related high-level tools.
- `readonly,low`: direct low-level operations; use only with explicit approval.
- `readonly,high,low`: technically accepted by current CLI help/parser, but avoid by default because it exposes both high-level and low-level mutation surfaces.
- `high`: high-level write group without readonly context; rarely ideal.
- `low`: low-level only; avoid unless requested.
- `compact`: compact facade only; cannot be combined with anything else.

Upstream documentation has inconsistent wording about `high` and `low` being mutually exclusive; current 7.1.3 CLI help advertises `readonly,high,low`, and the parser accepts any listed set. Apply the safety policy above even when the server accepts a broad exposure.

Avoid combining `compact` with other groups; it is intended as a standalone facade.

## Recommended Defaults

- Unknown target: `--exposition=readonly`
- Production: `--exposition=readonly`
- Development with approved edits: `--exposition=readonly,high`
- Admin/testing direct operations: `--exposition=readonly,low` only after explicit risk acceptance

## Tool Families

Read-only inspection:

- Objects and packages: `SearchObject`, `GetObjectsList`, `GetObjectsByType`, `GetPackageContents`, `ReadPackage`
- Object metadata: `GetObjectInfo`, `GetObjectStructure`, `GetTypeInfo`, `GetAdtTypes`
- Source and structure: `ReadClass`, `ReadProgram`, `ReadInterface`, `ReadTable`, `ReadView`, `ReadStructure`
- Dependency and impact: `GetWhereUsed`, `DescribeByList`, `GetIncludesList`
- ABAP understanding: `GetAbapAST`, `GetAbapSemanticAnalysis`, `SearchSource`
- Runtime read diagnostics: runtime feed, profiler, dump, gateway error log, and system message readers

High-level development:

- ABAP OO: `CreateClass`, `UpdateClass`, `GetClass`, `DeleteClass`, local definitions/macros/types/test-class handlers
- RAP: behavior definition/implementation, service definition/binding, metadata extension handlers
- DDIC and CDS/view: domain, data element, table, structure, view handlers
- Classic ABAP: program, function group, function module, function include handlers
- Checks and tests: `Check*`, `CreateUnitTest`, `RunUnitTest`, status/result handlers
- Lifecycle and transport: `ActivateObjects`, `CreateTransport`, transport readers

Low-level direct operations:

- Tools with `Low` suffix can lock, unlock, validate, activate, create, update, and delete at a more direct level. Treat these as dangerous and use only when high-level tools cannot do the job.

Compact facade:

- `HandlerCreate`, `HandlerGet`, `HandlerUpdate`, `HandlerDelete`
- `HandlerValidate`, `HandlerActivate`, `HandlerLock`, `HandlerUnlock`, `HandlerCheckRun`
- `HandlerUnitTestRun`, `HandlerUnitTestStatus`, `HandlerUnitTestResult`
- `HandlerCdsUnitTestStatus`, `HandlerCdsUnitTestResult`
- `HandlerProfileRun`, `HandlerProfileList`, `HandlerProfileView`
- `HandlerDumpList`, `HandlerDumpView`
- `HandlerServiceBindingListTypes`, `HandlerServiceBindingValidate`
- `HandlerTransportCreate`

## Safe Change Workflow

Before mutation:

1. Confirm SAP target, client, system type, and package or object namespace.
2. Prefer `readonly,high`; avoid `low`.
3. Read the existing object and metadata.
4. Run where-used or dependency checks when the change may affect callers.
5. Identify transport needs for on-premise development.

During mutation:

1. Use high-level `Create*` or `Update*` tools.
2. Avoid delete unless the user directly asked for deletion and dependencies are checked.
3. Avoid activation until checks are acceptable.

After mutation:

1. Run `Check*` or validation handlers.
2. Run relevant unit tests or CDS unit-test status/result handlers.
3. Activate only when requested or clearly part of the approved workflow.
4. Report object names, checks run, results, and any transport created or used.

## Red Lines

Ask before proceeding when:

- The target appears to be production.
- The action is delete, activate, transport creation/release, SQL execution, runtime execution, profiling, lock/unlock, or any `Low` tool.
- The request includes secrets in chat; tell the user to store them in service-key files, env vars, or client config instead.
- The SAP system type is unknown and the requested tool may be system-type-specific.
- The user wants remote HTTP/SSE exposure, `--host=0.0.0.0`, or `--unsafe`.

## ABAP Cloud Limitation

Direct ADT data preview of database tables is blocked by SAP BTP backend policies. Expect a descriptive error for table data preview in ABAP Cloud; on-premise systems can support data preview.
