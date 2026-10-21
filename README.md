Fix my MCP configuration so the Confluence server runs from this workspace only.

1. Replace the entire contents of .vscode/mcp.json in this workspace with ONLY a "confluence" server:
   - type: stdio
   - command: ${workspaceFolder}/.venv/Scripts/python.exe
   - args: ["${workspaceFolder}/confluence_mcp.py"]
   - env: CONFLUENCE_BASE_URL = https://confluence.agile.bns, CONFLUENCE_TOKEN = ${input:confluence-token}
   - inputs: one promptString input with id "confluence-token", description "Confluence Personal Access Token", password true
   Do NOT include the io.draw/mcp or com.atlassian servers, and do NOT include "gallery" or "version" fields.

2. In my user-level mcp.json (C:\Users\s4754194\AppData\Roaming\Code\User\mcp.json), remove ONLY the "confluence" server entry and the "confluence-token" input. Keep io.draw/mcp and com.atlassian/atlassian-mcp-server exactly as they are.

3. Check that both files are valid JSON with no errors in the Problems panel, then tell me what you changed. Never write or ask for my actual token; it is entered through the VS Code password prompt when the server starts.
