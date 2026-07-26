# MCP ABAP ADT Skill

A Codex skill for configuring and safely operating the [`fr0ster/mcp-abap-adt`](https://github.com/fr0ster/mcp-abap-adt) server against SAP ABAP systems.

It covers read-only repository inspection, safe development workflows, service-key and `.env` authentication, MCP client configuration, and SAP GUI compatibility guidance.

## Install

Clone this repository, then copy or link the `mcp-abap-adt` folder into your Codex skills directory. The resulting directory must contain `SKILL.md`, `agents/`, and `references/`.

For example on Windows:

```powershell
git clone https://github.com/flymanckt/mcp-abap-adt-skill.git
Copy-Item -Recurse .\mcp-abap-adt-skill "$env:USERPROFILE\.codex\skills\mcp-abap-adt"
```

Restart Codex after installation. Invoke it with `$mcp-abap-adt` or ask Codex to work with an SAP ABAP system.

## Safety

The skill defaults unknown or production targets to read-only access. It does not authorize credential sharing, object mutation, activation, transport work, SQL execution, or remote server exposure without explicit user intent.

## Sources

The integration guidance targets [`@mcp-abap-adt/core`](https://www.npmjs.com/package/@mcp-abap-adt/core). Validate package versions and upstream documentation before relying on version-specific behavior.

## License

[MIT](LICENSE)
