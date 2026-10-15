## Lab 1: Custom instructions

**Goal:** Create the `iphone-duo-viewer` repository in the Copilot app, give it a full instruction stack, and confirm which instructions Copilot loads. Add personal and organization instructions on top.

**Repo:** `iphone-duo-viewer` (new, local) · **Surface:** GitHub Copilot app

### Exercise 1.1: Create the local repository and open it in the Copilot app

1. Create an empty Git repository:

   ```bash
   mkdir -p ~/projects/iphone-duo-viewer
   cd ~/projects/iphone-duo-viewer
   git init -b main
   ```

   **Windows 11 Pro (Command Prompt):**

   ```cmd
   mkdir "%USERPROFILE%\projects\iphone-duo-viewer"
   cd /d "%USERPROFILE%\projects\iphone-duo-viewer"
   git init -b main
   ```

1. Open it in the Copilot app:

   ```bash
   copilot app
   ```

   `copilot app` opens the Copilot app in the current directory, straight into a new session. You can also click **+** next to **Projects** in the app and choose the folder on your machine.

1. In the dropdown under the prompt box, choose to run the session **in your local repository**. Set the mode to **Interactive**.

> [!IMPORTANT]
> ✅ **Checkpoint:** `iphone-duo-viewer` appears under **Projects** in the app sidebar, with an open session.

> [!NOTE]
> This repo stays local until Lab 6, when you publish it to GitHub for the cloud agent.

### Exercise 1.2: Write the repository-wide instructions

1. Click on the `Open in Visual Studio Code` button
2. Create a file `.github/copilot-instructions.md` with this content:

   ```markdown
   # iPhone Duo knolling viewer

   Interactive 3D viewer for the iPhone Duo with an exploded view and a knolling mode.

   ## Stack
   - TypeScript, Vite, and three.js. No UI framework.
   - npm for package management.

   ## Commands
   - `npm run dev`: start the dev server
   - `npm run build`: type-check and build
   - `npm test`: run the Playwright tests

   ## Conventions
   - Scene units are millimeters.
   - One mesh per physical component. Name every mesh after its part (for example, `inner-display`).
   - Read all dimensions from `src/specs.ts`. No magic numbers.
   - Keep modules small: scene setup, parts, layout (exploded and knolling), and UI.
   ```

### Exercise 1.3: Add path-specific instructions

1. Create `.github/instructions/threejs.instructions.md`:

   ```markdown
   ---
   name: three.js standards
   description: Scene, mesh, and animation conventions for the viewer's three.js code
   applyTo: "src/**/*.ts"
   ---

   - Create one `THREE.Group` per assembly and one `THREE.Mesh` per component.
   - Set `mesh.name` to the kebab-case part name.
   - Store each part's exploded and knolled transforms. Don't recompute them per frame.
   - Animate transitions with a single `requestAnimationFrame` loop and easing. No `setInterval`.
   - Dispose geometries and materials when you remove meshes.
   ```

1. Create `.github/instructions/playwright.instructions.md` manually with these team rules:

   ```markdown
   ---
   applyTo: "tests/**/*.spec.ts"
   ---

   ## Playwright test requirements

   - Use `page.getByRole` locators; avoid CSS selectors.
   - Assert on visible state with `toBeVisible()`, not DOM presence.
   - Each test must call `test.describe` with a feature name.
   - Mock external HTTP calls with `page.route()`.
   ```

1. Save all files and commit the repository-wide instructions and both path-specific files. Return to the Copilot app session for Exercise 1.4. The optional shared-skill comparison in Exercise 1.3.1 is at the end of this lab.

### Exercise 1.4: Scaffold the app and check that the instructions apply

1. In the app session, enter (Use `auto` model yolo!):

   ```text
   Scaffold the project described in the repository instructions. Render a single box
   sized to the closed iPhone Duo (84.1 × 117.8 × 11.3 mm) with orbit controls. Add a
   "Knolling" toggle button that does nothing yet, and one Playwright test that checks
   the button and the canvas are visible.
   ```

1. Review the changes. Look for `src/specs.ts`, a named mesh, millimeter units, a `tests/*.spec.ts` file that uses `test.describe` and `getByRole`, and no magic numbers.
1. In a terminal, list the instruction files Copilot discovers for this repo:

   ```bash
   cd ~/projects/iphone-duo-viewer
   copilot instruction list
   ```

   **Windows 11 Pro (Command Prompt):**

   ```cmd
   cd %USERPROFILE%\projects\iphone-duo-viewer
   copilot instruction list
   ```

> [!IMPORTANT]
> ✅ **Checkpoint:** `copilot instruction` lists `.github/copilot-instructions.md` and both `*.instructions.md` files, and the scaffold follows the rules in them.

### 🧯 **Troubleshooting** Exercise 1.6: If the app doesn't open in a browser, configure it to run the viewer

<details>
<summary>Show how to configure the app to start the viewer</summary>

The app reads project settings from `.github/github-app.yml`. This file is for the Copilot app only; it isn't a custom instructions file.

1. Create `.github/github-app.yml`:

   ```yaml
   scripts:
     - name: Setup
       command: npm install
       triggers:
         - session.create
     - name: Run
       command: npm run dev

   server_ready_pattern: '(?i)Local:\s+(https?://\S+)'
   auto_open_in_browser: true
   ```

1. The app asks you to review and accept the configuration because you created it outside the app. Read each command, then accept.
1. Run the **Run** script.

> [!IMPORTANT]
> ✅ **Checkpoint:** The viewer opens in the app's integrated browser and shows the box.

**Discussion:** `github-app.yml` also has an `instructions:` key. How is it different from `.github/copilot-instructions.md`? The `instructions:` key applies only in the Copilot app. The instructions file also works in the CLI, VS Code, and the cloud agent. Put conventions in the file.

</details>

### 🧯 **Troubleshooting (Lab 1)**

<details>
<summary>Show how to fix instruction loading and scope issues</summary>

* **A path-specific file never loads.** Without `applyTo`, the file isn't applied automatically. Check that the glob matches the file being edited.
* **Copilot ignores part of a long file.** Instruction files over about 1,000 lines risk silent truncation. Split them by domain.
* **`@relative/path` references don't resolve.** They work in `.github/copilot-instructions.md`, `AGENTS.md`, and `CLAUDE.md`, but not in `*.instructions.md` files.
* **Looking for VS Code's "chat locations" settings?** The `chat.*Locations` settings are deprecated and apply only to the VS Code Local agent. Use the standard `.github/` and `~/.copilot/` locations instead.
* **`root-instructions` or `area-instructions` mentions a tool it can't find.** These skills were written for VS Code and mention VS Code-only tools such as `emit_file_content`. In the CLI or the Copilot app, ask Copilot to write the file with its normal edit tools.

</details>

### 🚀 **Stretch goals**

#### 🚀 **Stretch goal** Use Copilot to create the file

<details>
<summary>Show the Copilot prompt and file content</summary>

There's no code yet at Exercise 1.2, so `/init create a custom instruction with the following` has nothing to analyze. As an alternative to the manual step there, write the instructions first to describe the project you *intend* to build. If you already scaffolded the app, compare this example with the existing file rather than overwriting it.

1. In the app session, ask Copilot to create `.github/copilot-instructions.md` with this content:

   ```markdown
   # iPhone Duo knolling viewer

   Interactive 3D viewer for the iPhone Duo with an exploded view and a knolling mode.

   ## Stack
   - TypeScript, Vite, and three.js. No UI framework.
   - npm for package management.

   ## Commands
   - `npm run dev`: start the dev server
   - `npm run build`: type-check and build
   - `npm test`: run the Playwright tests

   ## Conventions
   - Scene units are millimeters.
   - One mesh per physical component. Name every mesh after its part (for example, `inner-display`).
   - Read all dimensions from `src/specs.ts`. No magic numbers.
   - Keep modules small: scene setup, parts, layout (exploded and knolling), and UI.
   ```

1. Commit the file.

</details>

### 🚀 **Stretch goal:** Exercise 1.3.1: Create instructions using Copilot

<details>
<summary>Show how to compare manual instructions with the shared skill</summary>

1. Open or return to a Copilot app session in `iphone-duo-viewer`. You don't need to have VS Code open; if it is open, save any edits first.
1. Generate a comparison with the shared `area-instructions` skill from `common-agents`. In the app session, enter:

   ```text
   Use the area-instructions skill to propose .github/instructions/playwright.instructions.md
   for files matching tests/**/*.spec.ts. Playwright is the test framework.
   Compare your proposal with the existing file from Exercise 1.3; don't overwrite it.
   ```

1. Compare the result with these team rules and merge in anything that's missing:

   ```markdown
   ---
   applyTo: "tests/**/*.spec.ts"
   ---

   ## Playwright test requirements

   - Use `page.getByRole` locators; avoid CSS selectors.
   - Assert on visible state with `toBeVisible()`, not DOM presence.
   - Each test must call `test.describe` with a feature name.
   - Mock external HTTP calls with `page.route()`.
   ```

1. Check both path-specific files and commit any changes.

> [!IMPORTANT]
> ✅ **Checkpoint:** A skill from `common-agents` produced an instruction proposal in a different repository, and your saved file still contains the team rules. That's the Lab 0 setup working.

</details>

### 🚀 **Stretch goal:** Exercise 1.5: Improve the instructions with `/init` and `root-instructions`

<details>
<summary>Show how to compare /init with root-instructions</summary>

Now that the repo has code, let Copilot compare the instructions with it. You'll try the built-in command and your team's skill, and compare the two.

1. In the app session, enter `/init`. Review the suggestions, accept the accurate ones (such as the real script names from `package.json`), and reject anything that contradicts your conventions. Commit.
1. Now ask the shared skill for a second opinion. Don't let it write anything yet:

   ```text
   Use the root-instructions skill to analyze this repository. Don't write any files.
   Show me what it would put in .github/copilot-instructions.md that my current file
   is missing.
   ```

1. Apply anything useful, and commit.

> [!IMPORTANT]
> ✅ **Checkpoint:** `.github/copilot-instructions.md` matches the real build and test commands.

**Discussion:** When should a team keep its own instruction-generator skill instead of relying on `/init`? For example, when it wants every repository's instructions to have the same sections, or to follow house rules that `/init` doesn't know about.

</details>

### 🚀 **Stretch goal:** Use nested-hub and nested-detail

<details>
<summary>Show the hub-and-detail workflow</summary>

Use the `nested-hub` skill to generate a lean `AGENTS.md` hub for the viewer, then `nested-detail` to write one of the detail files it recommends (for example, "animation and transitions").

</details>

### 🚀 **Stretch goal:** Exercise 1.7: Add personal custom instructions

<details>
<summary>Show how to set instructions that follow you across repositories</summary>

Personal instructions follow you across repositories.

1. **On GitHub.com:** Open [Copilot Chat](https://github.com/copilot). Click your profile picture in the bottom left, then **Personal instructions**.
1. **For the CLI and the Copilot app:** Create `~/.copilot/copilot-instructions.md`.
1. Add these instructions to both:

   ```markdown
   1. Use TypeScript for coding examples.
   2. Prefer example programs that run on the command line.
   3. Make responses human-friendly:
      - Avoid superficial "-ing" phrases ("highlighting...", "ensuring...", "reflecting...",
        "showcasing...", "fostering..."). Delete them or expand them with real sources.
      - Avoid vague attributions ("Experts believe", "Industry reports suggest",
        "Some critics argue"). Name the source or delete the claim.
      - Avoid AI vocabulary: additionally, crucial, delve, enduring, enhance, fostering,
        garner, interplay, intricate, landscape (abstract), pivotal, showcase,
        tapestry (abstract), testament, underscore, vibrant. Use plain words.
      - Don't use fancy ways to say "is" ("serves as", "stands as", "boasts", "features").
        Say "is" or "has".
      - Remove chatbot phrases ("I hope this helps!", "Let me know if...", "Of course!",
        "Certainly!", "Found the smoking gun!").
      - Don't defer ("Great question! You're absolutely right!"). Respond directly.
   ```

1. On GitHub.com, click **Save**.
1. In Copilot Chat on GitHub.com, ask `Show me how to read a JSON file`.

> [!IMPORTANT]
> ✅ **Checkpoint:** The answer is in TypeScript, runs from the command line, and contains none of the banned phrases. You'll reuse these instructions in [Exercise 4.5](lab-4.md#-stretch-goal-exercise-45-see-personal-instructions-shape-agent-output).

> [!TIP]
> The Copilot app also has global **App instructions** (app settings → **Sessions** → "Instructions"). They apply only in the app.

</details>

### 🚀 **Stretch goal:** Exercise 1.8: Add organization custom instructions (organization owners)

<details>
<summary>Show how to set instructions for your organization</summary>

1. Go to your organization, then **Settings**.
1. In the left sidebar, click **Copilot**, then **Custom instructions**.
1. Under "Preferences and instructions", add:

   ```text
   Use Octokit with the throttling and retry plugins for GitHub API calls.
   Prefer GitHub App authentication over OAuth apps.
   When building MCP servers, use OAuth with a custom OAuth app for authentication.
   ```

1. Click **Save changes**.
1. In [Copilot Chat on GitHub.com](https://github.com/copilot), ask `Write a script that lists all repositories in my organization`.

> [!IMPORTANT]
> ✅ **Checkpoint:** The script uses Octokit with rate-limit handling and GitHub App authentication.

> [!NOTE]
> The workshop slides describe organization custom instructions as Enterprise-only. The current GitHub Docs article lists them for organizations on **Copilot Business or Copilot Enterprise**. Confirm with your admin. Source: [Adding organization custom instructions](https://docs.github.com/en/copilot/how-tos/copilot-on-github/customize-copilot/add-custom-instructions/add-organization-instructions).

**Discussion:** If your personal instructions say "use TypeScript" and the org instructions say something else, which wins? Personal instructions take precedence over repository instructions, which take precedence over organization instructions.

</details>

---

[← Previous: Lab 0](lab-0.md) · [README ↑](../README.md#own-your-copilot-hands-on-lab-exercises) · [Next: Lab 2 →](lab-2.md)
