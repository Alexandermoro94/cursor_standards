Context7 MCP setup notes

- Primary config: .cursor/mcp.json (adds the Context7 MCP server for Cursor clients).
- Uses local command `npx -y @upstash/context7-mcp`.
- Add your API key in the ENV value `CONTEXT7_API_KEY` inside the config for higher limits/private repos.
- No extra packages installed globally; Cursor will run npx on demand.
- Repository left untouched beyond config addition.
