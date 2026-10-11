## Lab 5: MCP servers

**Goal:** Give the viewer repo a shared MCP configuration so every contributor gets up-to-date three.js documentation, and use the CLI's built-in browser automation to check the viewer.

**Repo:** `iphone-duo-viewer` · **Surface:** CLI, Copilot app

### Exercise 5.1: Get a Context7 API key

1. Open the Context7 page in the GitHub MCP Registry: [github.com/mcp/upstash/context7](https://github.com/mcp/upstash/context7).
1. Follow the steps there to sign up and create an API key.
1. Add the key to your shell profile, not to the repo:

   ```bash
   export CONTEXT7_API_KEY=YOUR-API-KEY
   ```

   **Windows 11 Pro (Command Prompt, persists for new sessions):**

   ```cmd
   setx CONTEXT7_API_KEY "YOUR-API-KEY"
   ```

   > [!NOTE]
   > `setx` writes the variable for future Command Prompt sessions; it doesn't affect the one you ran it in. Open a new Command Prompt window afterward.

### Exercise 5.2: Add a repository MCP configuration

1. Create `~/projects/iphone-duo-viewer/.github/mcp.json`:

   ```json
   {
     "mcpServers": {
       "context7": {
         "type": "http",
         "url": "https://mcp.context7.com/mcp",
         "headers": { "CONTEXT7_API_KEY": "${CONTEXT7_API_KEY}" },
         "tools": ["*"]
       }
     }
   }
   ```

   The file references the environment variable, so it's safe to commit. The CLI expands variables in `headers`.

1. Commit the file.
1. Open a new terminal (so it has `CONTEXT7_API_KEY`), start `copilot` in the viewer repo, and confirm folder trust if prompted. The CLI skips project MCP servers in folders you haven't trusted.
1. Check the server:

   ```copilot
   /mcp show context7
   ```

1. Ask: `Use context7 to show the current three.js API for OrbitControls damping, then apply it to the viewer.`

> [!IMPORTANT]
> ✅ **Checkpoint:** `/mcp show context7` lists its tools, and the answer includes a Context7 tool call.

> [!TIP]
> MCP servers configured for a repository or for the CLI are also available in the Copilot app. If Context7 can't connect in the app, the app may not see your shell's environment variables. Start the app from a terminal with `copilot app`.

### 🧯 **Troubleshooting (Lab 5)**

<details>
<summary>Show how to fix MCP server connection and policy issues</summary>

* **The server doesn't appear.** Business and Enterprise organizations need the **MCP servers in Copilot** policy turned on. It's off by default.
* **The server is blocked.** Your organization may use an MCP registry allowlist. Only servers on the allowlist can run.
* **VS Code doesn't see the server.** The CLI doesn't read `.vscode/mcp.json`, and VS Code uses a different top-level key (`servers`). If you also want the server in VS Code, add a `.vscode/mcp.json` in the VS Code format.

</details>

### 🚀 **Stretch goal:** Exercise 5.3: Check the viewer with the built-in Playwright server

<details>
<summary>Show how to capture viewer screenshots with Playwright</summary>

The CLI includes a built-in `playwright` MCP server for browser automation. You don't need to configure it.

1. In the viewer repo session, ask:

   ```text
   Start the dev server, open the viewer with Playwright, take a screenshot, then click
   the Knolling button and take another. Stop the dev server when you're done.
   ```

> [!IMPORTANT]
> ✅ **Checkpoint:** You get two screenshots of the viewer. In Lab 8, you'll use the same technique to check that knolled parts don't overlap.

</details>

---

[← Previous: Lab 4](lab-4.md) · [README ↑](../README.md#own-your-copilot-hands-on-lab-exercises) · [Next: Lab 6 →](lab-6.md)
