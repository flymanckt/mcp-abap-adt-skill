# Setup Reference

Use this reference when installing, configuring, or troubleshooting `fr0ster/mcp-abap-adt`.

## Package

- npm package: `@mcp-abap-adt/core`
- binary: `mcp-abap-adt`
- Node requirement from package metadata: Node `>=22.0.0`, npm `>=9.0.0`
- Verify latest before installation:
  ```powershell
  npm view @mcp-abap-adt/core version bin engines --json
  ```

## Installation Choices

Prefer `npx` for MCP client configs when the user does not need a global command:

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

Use global install when the user wants the command available in shells:

```powershell
npm install -g @mcp-abap-adt/core
mcp-abap-adt --help
```

Install the configurator only when the user wants assisted client config generation:

```powershell
npm install -g @mcp-abap-adt/configurator
mcp-conf --client cline --name abap --mcp TRIAL
```

## Transports

- `stdio`: default and best for local MCP clients.
- `http` or `streamable-http`: endpoint defaults to `http://localhost:3000/mcp/stream/http`.
- `sse`: defaults to port `3001`, SSE path `/sse`, post path `/messages`.

Examples:

```powershell
mcp-abap-adt --transport=stdio --mcp=TRIAL --exposition=readonly
mcp-abap-adt --transport=http --port=3000 --mcp=TRIAL --exposition=readonly
mcp-abap-adt --transport=sse --port=3001 --mcp=TRIAL --exposition=readonly
```

Bind to `127.0.0.1` unless the user explicitly needs remote access. If using `--host=0.0.0.0`, require clients to supply full SAP connection headers and avoid relying on local default credentials.

## Destination Service Keys

Destination-based auth is the recommended model. A destination is the service-key filename without `.json`.

Windows:

```text
%USERPROFILE%\Documents\mcp-abap-adt\service-keys\TRIAL.json
%USERPROFILE%\Documents\mcp-abap-adt\sessions\TRIAL.env
```

Linux/macOS:

```text
~/.config/mcp-abap-adt/service-keys/TRIAL.json
~/.config/mcp-abap-adt/sessions/TRIAL.env
```

Use:

```powershell
mcp-abap-adt --transport=stdio --mcp=TRIAL
```

Notes:

- `--mcp=<destination>` skips automatic `./.env` loading.
- Session storage is in-memory by default.
- `--unsafe` persists session tokens to disk; use only with explicit user acceptance.
- Override auth-broker storage with `--auth-broker-path=<path>`.

## .env Configuration

Use `.env` for single-system local testing or when service keys are unavailable.

On-premise basic auth:

```env
SAP_URL=https://your-onprem-system.example.com:8000
SAP_CLIENT=100
SAP_AUTH_TYPE=basic
SAP_USERNAME=DEVELOPER
SAP_PASSWORD=secret
SAP_SYSTEM_TYPE=onprem
SAP_MASTER_SYSTEM=DEV
SAP_RESPONSIBLE=DEVELOPER
```

BTP or ABAP Cloud JWT:

```env
SAP_URL=https://your-system.abap.us10.hana.ondemand.com
SAP_AUTH_TYPE=jwt
SAP_JWT_TOKEN=access-token
SAP_REFRESH_TOKEN=refresh-token
SAP_SYSTEM_TYPE=cloud
```

RFC:

```env
SAP_CONNECTION_TYPE=rfc
SAP_SYSTEM_TYPE=onprem
```

RFC requires SAP NW RFC SDK prerequisites.

Important parser rule: only full-line comments are supported in `.env`; avoid inline comments.

## Config Precedence

For env files:

1. `--env-path=<path>` or `MCP_ENV_PATH`
2. `--env=<destination>` from sessions store
3. `--mcp=<destination>` auth broker
4. `.env` in current directory
5. `--auth-broker`
6. auth broker fallback when no `.env` is found

Command-line options override YAML config and environment defaults.

## YAML Config

Use YAML when command lines get long:

```yaml
transport: stdio
mcp: TRIAL
connection-type: http
unsafe: false
auth-broker: false
exposition:
  - readonly
```

Run:

```powershell
mcp-abap-adt --conf=config.yaml
```

If the YAML file does not exist, the server can generate a template.

## Troubleshooting

- Verify Node and npm versions first.
- Run `mcp-abap-adt --help` or `npx -y --package @mcp-abap-adt/core mcp-abap-adt --help`.
- If auth opens a browser on a busy callback port, set `--browser-auth-port=<port>`.
- If on-premise create/update fails, check `SAP_SYSTEM_TYPE=onprem`, `SAP_MASTER_SYSTEM`, and responsible user settings.
- If tools are missing, check `SAP_SYSTEM_TYPE` and `--exposition`.
- If an unexpected `.env` is loaded, use `--env-path` or `--mcp` explicitly.
