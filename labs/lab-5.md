## Lab 5: MCP servers

**Goal:** Give the viewer a global MCP configuration so every contributor gets up-to-date three.js documentation, and use the CLI's built-in browser automation to check the viewer.

**Repo:** `iphone-duo-viewer` · **Surface:** CLI, Copilot app

### Exercise 5.1: Setup Context7 MCP

1. Open the Context7 page in the GitHub MCP Registry: [github.com/mcp/upstash/context7](https://github.com/mcp/upstash/context7).
1. Follow the steps there to sign up and create an account
2. In `Github Copilot App` add `Context7` as MCP
3. Edit MCP Configuration and correct the URL to use OAuth
   ```diff
   - "url": "https://mcp.context7.com/mcp"
   + "url": "https://mcp.context7.com/mcp/oauth"
   ```
4. `Sign in` and confirm if prompted.
   
5. Check the server:

   ```copilot
   /mcp show context7
   ```

6. Ask: `Use context7 to show the current three.js API for OrbitControls damping, then apply it to the viewer.`

> [!IMPORTANT]
> ✅ **Checkpoint:** `/mcp show context7` lists its tools, and the answer includes a Context7 tool call.

> [!TIP]
> MCP servers configured for a repository or for the CLI are also available in the Copilot app. If Context7 cannot connect after using the OAuth endpoint, open its MCP settings and authenticate again.

### 🧯 **Troubleshooting (Lab 5)**

<details>
<summary>Show how to fix MCP server connection and policy issues</summary>

* **The server doesn't appear.** Business and Enterprise organizations need the **MCP servers in Copilot** policy turned on. It's off by default.
* **The server is blocked.** Your organization may use an MCP registry allowlist. Only servers on the allowlist can run.
* **VS Code doesn't see the server.** The CLI doesn't read `.vscode/mcp.json`, and VS Code uses a different top-level key (`servers`). If you also want the server in VS Code, add a `.vscode/mcp.json` in the VS Code format.

</details>

### 🚀 **Stretch goal:** Exercise 5.2: Check the viewer with the built-in Playwright server

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
