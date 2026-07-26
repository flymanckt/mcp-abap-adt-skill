---
name: mcp-abap-adt
description: Use when Codex needs to work with SAP ABAP systems, including fr0ster/mcp-abap-adt, SAP GUI/SE38/SE11 development, ABAP repository inspection, object CRUD, transports, activation, syntax checks, runtime dumps, service-key auth, or MCP client configuration.
---

# MCP ABAP ADT

Use `mcp-abap-adt` to connect MCP clients to SAP ABAP repositories through ADT. Treat it as a live SAP system integration, not a local code helper: credentials, transports, activation, deletion, SQL, and transport creation all need explicit care.

## First Steps

1. Confirm the user's goal: read-only analysis, development changes, transport work, runtime diagnostics, or MCP client setup.
2. Confirm target SAP type: `cloud`, `onprem`, or `legacy`. On-premise needs `SAP_SYSTEM_TYPE=onprem`; legacy systems may need RFC.
3. Prefer safe exposure first:
   - Production or unknown system: `--exposition=readonly`.
   - Development with approved changes: `--exposition=readonly,high`.
   - Never expose `low` unless the user explicitly requests dangerous direct operations and accepts the risk.
   - Use `compact` only when the user wants the compact facade; it must be the only exposition group.
4. Verify current package details before installing or upgrading:
   ```powershell
   node --version
   npm --version
   npm view @mcp-abap-adt/core version bin engines --json
   ```
5. Read [setup.md](references/setup.md) when configuring installation, auth, client JSON, or troubleshooting. Read [tools-and-safety.md](references/tools-and-safety.md) before choosing handler groups or tool families.
6. Read [sapgui-abap-development.md](references/sapgui-abap-development.md) before changing ABAP through SAP GUI, SE38, SE11, SALV, BAPI calls, or when investigating dumps.

## Install And Configure

For local or client-managed execution, prefer `npx` unless the user asks for a global install:

```json
{
  "mcpServers": {
    "abap": {
      "command": "npx",
      "args": ["-y", "--package", "@mcp-abap-adt/core", "mcp-abap-adt", "--transport=stdio", "--mcp=TRIAL", "--exposition=readonly"]
    }
  }
}
```

For global installs:

```powershell
npm install -g @mcp-abap-adt/core
mcp-abap-adt --help
```

Use service-key destinations when available. On Windows, service keys live at:

```text
%USERPROFILE%\Documents\mcp-abap-adt\service-keys\<destination>.json
```

The destination name is the filename without `.json`; use it with `--mcp=<destination>`.

## Operating Rules

- Never put SAP passwords, JWTs, refresh tokens, service keys, or client secrets into chat, committed files, screenshots, or generated docs.
- Do not run create, update, delete, activate, lock, unlock, transport, SQL, runtime execution, or profiling tools unless the user asked for that action and the target system is identified.
- Before writes, gather object metadata and where-used or dependency context where possible.
- After writes, prefer ADT validation/check/unit-test tools before activation or transport.
- Bind HTTP and SSE to localhost by default. Treat `--host=0.0.0.0` as a security-sensitive choice.
- Avoid `--unsafe` unless the user wants file-based session persistence and accepts plain-text token storage risk.

## Common Workflows

Read-only ABAP impact analysis:

1. Start or configure MCP with `--exposition=readonly`.
2. Use search, package, object info, structure, type info, where-used, and source-read tools.
3. Summarize evidence from the SAP system and call out uncertainty when tool output is incomplete.

ABAP development change:

1. Confirm development system and target package/transport expectations.
2. Expose `readonly,high`, not `low`.
3. Inspect current object, dependencies, and inactive objects.
4. Create or update via high-level tools.
5. Run checks and relevant tests.
6. Activate only after checks are acceptable and user intent is clear.

SAP GUI ABAP development:

1. Read [sapgui-abap-development.md](references/sapgui-abap-development.md).
2. Treat clipboard paste and GUI automation as unreliable until the SAP editor screen itself proves the change.
3. Use the SE38 syntax-check button before activation when the user requests GUI syntax checks.

Authentication setup:

1. Prefer destination service keys and `--mcp=<destination>`.
2. Use `.env` only for single-system local testing or when the user cannot use service keys.
3. For on-premise basic auth, include `SAP_SYSTEM_TYPE=onprem` and `SAP_MASTER_SYSTEM` when create/update operations are needed.
4. For RFC, verify SAP NW RFC SDK prerequisites before recommending `SAP_CONNECTION_TYPE=rfc`.

## Source Anchors

Primary upstream project: https://github.com/fr0ster/mcp-abap-adt

Key docs to re-check when behavior matters:
- README: https://github.com/fr0ster/mcp-abap-adt/blob/main/README.md
- Authentication: https://github.com/fr0ster/mcp-abap-adt/blob/main/docs/user-guide/AUTHENTICATION.md
- Client configuration: https://github.com/fr0ster/mcp-abap-adt/blob/main/docs/user-guide/CLIENT_CONFIGURATION.md
- CLI options: https://github.com/fr0ster/mcp-abap-adt/blob/main/docs/user-guide/CLI_OPTIONS.md
- Handler management: https://github.com/fr0ster/mcp-abap-adt/blob/main/docs/user-guide/HANDLERS_MANAGEMENT.md
