Write a Python MCP server for Confluence using the official MCP Python SDK (FastMCP).

Target: https://confluence.agile.bns/ (self-hosted Confluence Server/Data Center, REST API under /rest/api).

Tools to expose:
- search_pages(query, space_key=None, limit=10): search content via CQL
- get_page(page_id): return title, space, version number, and body (storage format)
- create_page(space_key, title, body, parent_id=None): create a new page
- update_page(page_id, title=None, body=None): fetch the current version first, then update with version+1
- delete_page(page_id): delete a page

Requirements:
- Authentication: Personal Access Token sent as "Authorization: Bearer <token>". Read the token from the CONFLUENCE_TOKEN environment variable. Never hardcode it or log it.
- Read the base URL from CONFLUENCE_BASE_URL, defaulting to https://confluence.agile.bns
- Use the requests library with timeouts, and return clear error messages for 401, 403, 404, and 409 (version conflict).
- Handle SSL properly (allow an optional CONFLUENCE_CA_BUNDLE env var for a corporate CA certificate).
- Run over stdio transport.
- Keep it in a single file, confluence_mcp.py, plus a requirements.txt.

Also give me:
1. The exact mcp.json entry to register this server in VS Code, using an "inputs" prompt with password: true for the token instead of storing it in the file.
2. Steps to test it (e.g., a quick search_pages call) before using it with the Confluence agent.
