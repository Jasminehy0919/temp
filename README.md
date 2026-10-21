Please do the following and tell me what you changed:

1. Fix the malformed frontmatter in .github/agents/confluence.agent.md (the opening "---" is broken, it currently shows something like "deve---"). It must start with a clean "---" line, then the YAML fields, then a closing "---". Do not change the rest of the content.

2. Move my workspace MCP config to the new location. Create .mcp.json in the workspace root (C:\Jasmine's Agent\.mcp.json) with ONLY the "confluence" server, using the format VS Code currently expects for that file:
   - command: ${workspaceFolder}/.venv/Scripts/python.exe
   - args: ["${workspaceFolder}/confluence_mcp.py"]
   - env: CONFLUENCE_BASE_URL = https://confluence.agile.bns, CONFLUENCE_TOKEN = ${input:confluence-token}
   - an input of type promptString with id "confluence-token", description "Confluence Personal Access Token", password true
   Then delete the deprecated .vscode/mcp.json so there is only one workspace config.

3. Check that the files have no errors in the Problems panel.

4. Never write, ask for, or log my real token. It is entered only through the VS Code password prompt when the server starts.
