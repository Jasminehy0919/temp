MCP is disabled by my organization (chat.mcp.access = none), so do not use MCP. Refactor confluence_mcp.py into a plain Python CLI script, confluence_cli.py, in this workspace. Keep confluence_mcp.py untouched.

Target: https://confluence.agile.bns (read from CONFLUENCE_BASE_URL, default to this). Auth: Bearer token from the CONFLUENCE_TOKEN environment variable. Never hardcode, print, or log the token. Support optional CONFLUENCE_CA_BUNDLE for corporate certificates.

argparse subcommands with JSON output:
- search --query "<CQL>" [--limit 10]
- get --page-id <id>
- create --space <key> --title <t> --body-file <path> [--parent-id <id>]
- update --page-id <id> --body-file <path> [--title <t>]  (fetch the current version first, then write version+1)
- delete --page-id <id> --confirm  (without --confirm, only print what would be deleted)

Use requests with timeouts and clear error messages for 401, 403, 404, 409.

Then update .github/agents/confluence.agent.md so the agent uses these terminal commands instead of MCP tools. Fix its frontmatter if malformed (it must start with a clean "---" line). Keep the rule that create, update, and delete require my explicit confirmation first, and never ask me to paste the token in chat.

Finally, tell me how to set CONFLUENCE_TOKEN in the PowerShell terminal for the current session only.
