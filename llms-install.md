# Installing the Runcrate MCP server

Runcrate's MCP server is hosted. There is nothing to clone, build or run locally, and no API key.

The current steps for every agent are at https://runcrate.ai/mcp.md. Fetch them and follow the section for the agent you are: `curl -fsSL "https://runcrate.ai/mcp.md"`. If you are Cline, these are the steps:

1. Add a remote server named `runcrate` with transport streamable HTTP and the URL `https://mcp.runcrate.ai/mcp`.
   With the Cline CLI: `cline mcp add runcrate https://mcp.runcrate.ai/mcp --transport streamableHttp --yes`.
   Or add this to `cline_mcp_settings.json`:

   ```json
   {
     "mcpServers": {
       "runcrate": {
         "type": "streamableHttp",
         "url": "https://mcp.runcrate.ai/mcp"
       }
     }
   }
   ```

2. Do not add an `Authorization` header or any env vars. The server uses OAuth with dynamic client registration: on the first tool call the client opens the user's browser to sign in to Runcrate and pick an environment.
3. Check it works by calling `list_gpu_types`, which returns live GPU prices and launches nothing.

Docs: https://runcrate.ai/docs/mcp/setup
