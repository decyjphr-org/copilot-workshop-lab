# Own your Copilot: Hands-on lab exercises

These exercises go with the *Own your Copilot* workshop sessions. You work in two repositories the whole time:

* **`common-agents`** is your team's central repository of prompt files, skills, and agents. You build it out in Labs 2–4. In Lab 7 you turn it into a plugin marketplace.
* **`iphone-duo-viewer`** is a product repository you create locally in Lab 1. It consumes what's in `common-agents`, adds its own MCP servers and hooks, and becomes the keystone app in Lab 8.

This split matches how most organizations work. A platform team curates shared customizations once, and product teams use them everywhere.

## How the pieces fit together

```text
common-agents  (platform repo)                       iphone-duo-viewer  (product repo)
───────────────────────────────                      ─────────────────────────────────
.github/prompts/  ← Lab 2 (VS Code only)             .github/copilot-instructions.md  ← Lab 1
.github/skills/   ← Labs 2–3 ── skillDirectories ──▶ available in every session
.claude/skills/   ← preloaded ── skillDirectories ──▶ available in every session
.github/agents/   ← Lab 4  ◀── ~/.copilot/agents (symlink) ──▶ available in every session
plugins/ + .github/plugin/marketplace.json ← Lab 7 ── marketplace ──▶ installed as a plugin
                                                     .github/mcp.json                 ← Lab 5
                                                     .github/hooks/                   ← Lab 6
                                                     the viewer app                   ← Lab 8
```

| Lab | Topic | Repo | Time | Primary surface | Plan needed |
|-----|-------|------|------|-----------------|-------------|
| [0](#lab-0-prerequisites-and-setup) | Prerequisites and setup | `common-agents` | - | VS Code, CLI, Copilot app | Any paid plan |
| [1](#lab-1-custom-instructions) | Custom instructions | `iphone-duo-viewer` | - | Copilot app, GitHub.com | Any paid plan (org exercise: Business or Enterprise) |
| [2](#lab-2-prompt-files) | Prompt files | `common-agents` | - | VS Code | Any paid plan |
| [3](#lab-3-agent-skills) | Agent skills | `common-agents` | - | CLI, Copilot app | Any paid plan |
| [4](#lab-4-custom-agents) | Custom agents and orchestration | `common-agents` | - | CLI, Copilot app, GitHub.com | Org exercise: Enterprise |
| [5](#lab-5-mcp-servers) | MCP servers | `iphone-duo-viewer` | - | CLI, Copilot app | Any paid plan (org MCP policy must allow it) |
| [6](#lab-6-hooks) | Hooks | `iphone-duo-viewer` | - | CLI, cloud agent | Any paid plan |
| [7](#lab-7-plugins) | Plugins and marketplace | `common-agents` | - | CLI, Copilot app | Any paid plan |
| [8](#lab-8-keystone-iphone-duo-knolling-viewer) | Keystone: iPhone Duo knolling viewer | `iphone-duo-viewer` | - | Copilot app | Any paid plan |

**Conventions used in this guide**

> [!important]
> Replace `WORKSHOP-ORG` with `decyjphr-org`
>

* Placeholders are in `CAPS`. Replace them with your own values:
  * `YOUR-ORG` is an organization or user account where you can push.
  * `YOUR-USER` is your local macOS or Linux user name (for absolute paths).
  * `YOUR-HANDLE` is your GitHub username.
* Both repositories are assumed to live in `~/projects/`. Adjust the paths if you use a different folder.
* ✅ **Checkpoint** marks how you confirm that a step worked. Don't skip these.
* 🧯 **Troubleshooting** lists the most common problems for that lab.
* 🚀 **Stretch goal** items are optional. Try them if you finish early.
* Commit after each lab. In the `common-agents` labs, push as well, because Lab 7 publishes from GitHub.

---

## Lab 0: Prerequisites and setup

**Goal:** Confirm your plan, sign in to every client, clone `common-agents`, and connect it to your personal Copilot configuration.

### Exercise 0.1: Verify your plan tier

1. Go to [**github.com → Settings → Billing and licensing**](https://github.com/settings/copilot/features).
1. Under "GitHub Copilot", note the plan name and the license source (personal or granted by an organization).
1. Your Copilot license would be granted by your GitHub Enterprise.
   
✅ **Checkpoint:** You know which plan you have, and which labs you can run end to end (see the table at the top).

### Exercise 0.2: Sign in to every client

Your workshop laptop already has VS Code (with the GitHub Copilot extension), GitHub CLI, the Copilot CLI, and the GitHub Copilot app installed. Sign in to each one with the account that holds your Copilot license.

1. **VS Code:** Open the Chat view (**View → Chat**, or <kbd>Cmd</kbd>+<kbd>Shift</kbd>+<kbd>I</kbd>). Sign in if prompted, then send `Hello`.
1. **GitHub CLI:** Check the sign-in status, and sign in if needed:

   ```bash
   gh auth status || gh auth login
   ```

1. **Copilot CLI:** Start a session, sign in, and confirm the account:

   ```bash
   copilot
   ```

   ```copilot
   /login
   /user
   ```

1. **Copilot app:** Open the app and click **Sign in to GitHub** if prompted. Skip repository selection during onboarding; you'll add projects in Exercise 0.3 and Lab 1.

✅ **Checkpoint:** VS Code Chat replies to `Hello`, `gh auth status` shows you're signed in, `/user` shows your GitHub username, and the Copilot app shows your account.


### Exercise 0.3: Get your copy of `common-agents`

`common-agents` comes preloaded with some prompt files, skills, and agents from your facilitator. You'll add to it in Labs 2–4 and publish from it in Lab 7, so you need **your own copy that you can push to**. Fork it, or create a repository from it if it's a template.

Choose one of the three options below. They all end in the same state: your copy cloned to `~/projects/common-agents`.

#### Option A: GitHub UI and Git

1. Fork `WORKSHOP-ORG/common-agents` into `YOUR-ORG` using the GitHub UI.
1. Clone it:

   ```bash
   cd ~/projects
   git clone https://github.com/YOUR-ORG/common-agents.git
   cd common-agents
   ```

#### Option B: Copilot CLI

1. Start the CLI in your projects folder:

   ```bash
   cd ~/projects
   copilot
   ```

1. Give Copilot the task:

   ```copilot
   Fork WORKSHOP-ORG/common-agents into YOUR-ORG and clone the fork into
   ./common-agents. Then list the contents of common-agents/.github.
   ```

1. Read each tool call before you approve it. Approve each one once, and don't choose "always allow" for shell commands yet. Lab 6 covers guardrails.

> [!TIP]
> If the fork step fails, check that GitHub CLI is signed in (`gh auth status`), or fork in the GitHub UI and ask Copilot only to clone.

#### Option C: GitHub Copilot app

1. Fork `WORKSHOP-ORG/common-agents` into `YOUR-ORG` using the GitHub UI.
1. In the app sidebar, click **+** next to **Projects**.
1. Choose the option to pick a project from GitHub, search for `YOUR-ORG/common-agents`, and clone it into `~/projects`.

#### Look at what's included

Your copy uses the standard project folders, and comes preloaded with:

```text
common-agents/
├── .github/
│   ├── agents/
│   │   └── code-explainer.agent.md          # Lab 4: explains a codebase (needs hardening)
│   ├── prompts/
│   │   ├── 1-1-meeting-agenda.prompt.md     # Lab 2: has a bug to fix
│   │   ├── analyze-zendesk.prompt.md        # reference only, not for chat
│   │   └── api-security-review.prompt.md    # Lab 2: full frontmatter example
│   └── skills/
│       ├── root-instructions/               # Lab 1: generates copilot-instructions.md
│       ├── area-instructions/               # Lab 1: generates *.instructions.md for one area
│       ├── nested-hub/  nested-detail/      # Lab 1 stretch: AGENTS.md hub + detail files
│       ├── release-validator/               # Lab 3: reference skill with script, references, assets
│       ├── visualize/                       # Lab 8: diagrams of code logic and data flow
│       └── html-in-canvas/                  # Lab 8 stretch: HTML rendered into a canvas or three.js
└── .claude/
    └── skills/
        └── js-to-typescript/                # a skill in the Claude-compatible location
```

Spend five minutes reading the files. Look for:

* How `release-validator/SKILL.md` links to its `scripts/`, `references/`, and `assets/` files instead of copying them.
* How `nested-hub` and `nested-detail` work as a pair: the hub skill recommends topics, and the detail skill writes one file per topic.
* That the skills live in two locations. Both `.github/skills` and `.claude/skills` are valid project skill locations.

This repo uses the standard project folders on purpose. When you open `common-agents` itself in any client, every file loads as a normal repository customization, so you can test as you write. Exercise 0.4 makes the same files available in *other* repositories.

✅ **Checkpoint:** `ls ~/projects/common-agents/.github` shows `agents`, `prompts`, and `skills`.

### Exercise 0.4: Use `common-agents` from every repository

Each customization type is shared in a different way:

| Type | How to make it available everywhere | Documented? |
|------|-------------------------------------|-------------|
| Skills | `skillDirectories` in `~/.copilot/settings.json` | ✅ Yes |
| Custom instructions | `COPILOT_CUSTOM_INSTRUCTIONS_DIRS` environment variable | ✅ Yes |
| Agents | Link `~/.copilot/agents` to `common-agents/.github/agents` | ✅ Both locations are documented. Linking them is a workshop shortcut. |
| Prompt files | Not possible. The CLI and the Copilot app don't support prompt files. | — |

In Lab 7 you replace this manual setup with a plugin. That's the supported way to share agents and skills with a team.

#### Step 1: Add the skills directory

1. Open `~/.copilot/settings.json` in an editor, or run `/settings` inside the CLI.
1. Add `skillDirectories`, keeping any settings that are already in the file:

   ```jsonc
   {
     "skillDirectories": [
       "/Users/YOUR-USER/projects/common-agents/.github/skills",
       "/Users/YOUR-USER/projects/common-agents/.claude/skills"
     ]
   }
   ```

   * Use absolute paths.
   * Point at the folder that **contains** the skill folders (`.github/skills`, plural). Don't point at a single skill.
   * List both folders. If you leave out `.claude/skills`, `js-to-typescript` works inside `common-agents` but nowhere else.
   * The environment variable `COPILOT_SKILLS_DIRS` (comma-separated) is an alternative.

#### Step 2: Link the shared agents folder

The Copilot CLI loads agents from two places: `WORKSPACE/.github/agents/` (project) and `~/.copilot/agents/` (personal). There's no setting that adds a third folder. Instead, make your personal agents folder a symbolic link to the `common-agents` agents folder. Every agent in `common-agents` then loads in every session, including agents you add later.

1. If you already have a `~/.copilot/agents` folder, back it up:

   ```bash
   [ -d ~/.copilot/agents ] && [ ! -L ~/.copilot/agents ] && mv ~/.copilot/agents ~/.copilot/agents.bak
   ```

1. Create the link and check it:

   ```bash
   ln -s ~/projects/common-agents/.github/agents ~/.copilot/agents
   ls -l ~/.copilot/agents
   # ~/.copilot/agents -> /Users/YOUR-USER/projects/common-agents/.github/agents
   ```

1. Keep personal agents out of the team repo. Any agent you save to `~/.copilot/agents` is now written into `common-agents/.github/agents`. Give personal agents a `my-` prefix and tell Git to ignore them:

   ```bash
   echo '.github/agents/my-*.agent.md' >> ~/projects/common-agents/.gitignore
   ```

   If you backed up any agents in step 1, move them back with a `my-` prefix, for example `mv ~/.copilot/agents.bak/reviewer.agent.md ~/.copilot/agents/my-reviewer.agent.md`.

1. Restart the CLI whenever you add or change an agent.

> [!WARNING]
> The docs list both agent locations, but they don't mention symbolic links. If the agents don't appear, remove the link (`rm ~/.copilot/agents`), create a normal folder, and copy the `.agent.md` files into it instead. Lab 7 replaces this step with a plugin.

#### Step 3 (optional): Share instruction files

If `common-agents` has a folder of shared `*.instructions.md` files, add it to your shell profile:

```bash
export COPILOT_CUSTOM_INSTRUCTIONS_DIRS=/Users/YOUR-USER/projects/common-agents/PATH-TO-INSTRUCTIONS
```

#### Step 4: Verify from outside the repository

1. Start the CLI from a folder that has nothing to do with `common-agents`:

   ```bash
   cd ~ && copilot
   ```

1. Run these commands:

   ```copilot
   /skills list
   /agent
   /env
   ```

1. Run `/skills info release-validator` and `/skills info js-to-typescript`. Both locations are inside `common-agents`.
1. In the Copilot app, click **Customize**, then **Skills**. Skills configured for the CLI also show up in the app.

✅ **Checkpoint:** In a session started outside `common-agents`, `/skills list` includes `release-validator` and `js-to-typescript`, and `/agent` lists `code-explainer`.

> [!NOTE]
> VS Code reads personal skills and agents from `~/.copilot/skills` and `~/.copilot/agents`, so the linked agents appear there. The VS Code docs don't mention `skillDirectories`. For the `iphone-duo-viewer` labs, use the CLI or the Copilot app, which do read it.

🧯 **Troubleshooting (Lab 0)**

* **VS Code says you have no Copilot license.** Sign out and sign in again with the account that holds the license.
* **The CLI rejects your token.** Classic personal access tokens (`ghp_`) aren't supported. Use `/login`, or a fine-grained PAT with the **Copilot Requests** permission in `COPILOT_GITHUB_TOKEN`.
* **The Copilot app shows an authorization error.** The app has its own policy toggle, separate from the CLI. Ask your admin to enable it.
* **`/skills list` doesn't show the common skills.** Check the path in `skillDirectories`: it must be absolute, it must exist, and it must end in `.github/skills`. Open `/settings` and look at the **Problems** tab for errors in the file.
* **`ls -l ~/.copilot/agents` lists files instead of showing a link.** `~/.copilot/agents` was still a normal folder when you ran `ln -s`, so the link was created *inside* it, as `~/.copilot/agents/agents`. Remove that link (`rm ~/.copilot/agents/agents`), run the backup command from Step 2, then create the link again.

---

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

1. Open it in the Copilot app:

   ```bash
   copilot app
   ```

   `copilot app` opens the Copilot app in the current directory, straight into a new session. You can also click **+** next to **Projects** in the app and choose the folder on your machine.

1. In the dropdown under the prompt box, choose to run the session **in your local repository**. Set the mode to **Interactive**.

✅ **Checkpoint:** `iphone-duo-viewer` appears under **Projects** in the app sidebar, with an open session.

> [!NOTE]
> This repo stays local until Lab 6, when you publish it to GitHub for the cloud agent.

### Exercise 1.2: Write the repository-wide instructions

There's no code yet, so `/init` has nothing to analyze. Write the instructions first to describe the project you *intend* to build.

1. In the app session, ask Copilot to create `.github/copilot-instructions.md` with this content (or create the file yourself):

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

1. Generate the Playwright instructions with the shared `area-instructions` skill from `common-agents`. In the app session, enter:

   ```text
   Use the area-instructions skill to create .github/instructions/playwright.instructions.md
   for files matching tests/**/*.spec.ts. Playwright is the test framework.
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

1. Commit both files.

✅ **Checkpoint:** A skill from `common-agents` produced a file in a different repository. That's the Lab 0 setup working.

### Exercise 1.4: Scaffold the app and check that the instructions apply

1. In the app session, enter:

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
   copilot instruction
   ```

✅ **Checkpoint:** `copilot instruction` lists `.github/copilot-instructions.md` and both `*.instructions.md` files, and the scaffold follows the rules in them.

### Exercise 1.5: Improve the instructions with `/init` and `root-instructions`

Now that the repo has code, let Copilot compare the instructions with it. You'll try the built-in command and your team's skill, and compare the two.

1. In the app session, enter `/init`. Review the suggestions, accept the accurate ones (such as the real script names from `package.json`), and reject anything that contradicts your conventions. Commit.
1. Now ask the shared skill for a second opinion. Don't let it write anything yet:

   ```text
   Use the root-instructions skill to analyze this repository. Don't write any files.
   Show me what it would put in .github/copilot-instructions.md that my current file
   is missing.
   ```

1. Apply anything useful, and commit.

✅ **Checkpoint:** `.github/copilot-instructions.md` matches the real build and test commands.

**Discussion:** When should a team keep its own instruction-generator skill instead of relying on `/init`? For example, when it wants every repository's instructions to have the same sections, or to follow house rules that `/init` doesn't know about.

🚀 **Stretch goal:** Use the `nested-hub` skill to generate a lean `AGENTS.md` hub for the viewer, then `nested-detail` to write one of the detail files it recommends (for example, "animation and transitions").

### Exercise 1.6: Configure the app to run the viewer

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

✅ **Checkpoint:** The viewer opens in the app's integrated browser and shows the box.

**Discussion:** `github-app.yml` also has an `instructions:` key. How is it different from `.github/copilot-instructions.md`? The `instructions:` key applies only in the Copilot app. The instructions file also works in the CLI, VS Code, and the cloud agent. Put conventions in the file.

### Exercise 1.7: Add personal custom instructions

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

✅ **Checkpoint:** The answer is in TypeScript, runs from the command line, and contains none of the banned phrases. You'll reuse these instructions in [Exercise 4.5](#exercise-45-see-personal-instructions-shape-agent-output).

> [!TIP]
> The Copilot app also has global **App instructions** (app settings → **Sessions** → "Instructions"). They apply only in the app.

### Exercise 1.8: Add organization custom instructions (organization owners)

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

✅ **Checkpoint:** The script uses Octokit with rate-limit handling and GitHub App authentication.

> [!NOTE]
> The workshop slides describe organization custom instructions as Enterprise-only. The current GitHub Docs article lists them for organizations on **Copilot Business or Copilot Enterprise**. Confirm with your admin. Source: [Adding organization custom instructions](https://docs.github.com/en/copilot/how-tos/copilot-on-github/customize-copilot/add-custom-instructions/add-organization-instructions).

**Discussion:** If your personal instructions say "use TypeScript" and the org instructions say something else, which wins? Personal instructions take precedence over repository instructions, which take precedence over organization instructions.

🧯 **Troubleshooting (Lab 1)**

* **A path-specific file never loads.** Without `applyTo`, the file isn't applied automatically. Check that the glob matches the file being edited.
* **Copilot ignores part of a long file.** Instruction files over about 1,000 lines risk silent truncation. Split them by domain.
* **`@relative/path` references don't resolve.** They work in `.github/copilot-instructions.md`, `AGENTS.md`, and `CLAUDE.md`, but not in `*.instructions.md` files.
* **Looking for VS Code's "chat locations" settings?** The `chat.*Locations` settings are deprecated and apply only to the VS Code Local agent. Use the standard `.github/` and `~/.copilot/` locations instead.
* **`root-instructions` or `area-instructions` mentions a tool it can't find.** These skills were written for VS Code and mention VS Code-only tools such as `emit_file_content`. In the CLI or the Copilot app, ask Copilot to write the file with its normal edit tools.

---

## Lab 2: Prompt files

**Goal:** Build reusable prompt files in `common-agents`, run them in VS Code, see where they stop working, and turn the most useful one into a skill that every repository can use.

**Repo:** `common-agents` · **Surface:** VS Code

> [!IMPORTANT]
> Prompt files work only in IDEs (VS Code, Visual Studio, JetBrains), and only in the workspace that contains them. The CLI, the Copilot app, and GitHub.com don't support them. VS Code has also deprecated prompt files for Agent Host sessions; they still work with the Local agent. This lab ends by moving a prompt file to a skill for that reason.

Open `~/projects/common-agents` in VS Code before you start.

### Exercise 2.1: Tour and fix the preloaded prompts

1. In Chat, type `/` and find the three preloaded prompts: `1-1-meeting-agenda`, `analyze-zendesk`, and `api-security-review`.
1. Open `api-security-review.prompt.md`. It uses every common frontmatter field: `agent`, `tools`, `model`, and `argument-hint`. Check that the `model` value matches a model in your model picker. If it doesn't, the prompt uses the currently selected model.
1. Open `1-1-meeting-agenda.prompt.md`. It says `{{period}}`, which VS Code doesn't recognize as a variable. The model just sees the literal text. Fix it:

   ```markdown
   Create a meeting agenda for a 1-1 based on activity from ${input:period:date range, for example March 23 – April 1}. Look in Daily notes.
   ```

1. Run `/1-1-meeting-agenda` and confirm that VS Code asks you for the period.
1. `analyze-zendesk` says it isn't meant for Copilot Chat, yet it still shows up in the `/` menu. Discuss where reference-only prompts should live instead, for example a `docs/` folder.
1. Create `.github/copilot-instructions.md` for `common-agents` itself, so the review prompt in the next exercise has standards to check against:

   ```markdown
   # common-agents

   Central repository of team prompt files, skills, and custom agents.

   ## Authoring rules
   - Skills: the frontmatter `name` must match the directory name (lowercase, hyphens).
   - Skills: the `description` must say when to use the skill ("Use when...").
   - Skills: link to files in `scripts/`, `references/`, and `assets/`; don't paste their content.
   - Skills: no links to files outside the repository, and no committed archives (`.zip`).
   - Agents: always set `tools` to the minimum required set.
   - Agents: the `description` says what the agent does and when to use it, not a persona.
   - Prompts: use `${input:name:placeholder}` for variables.
   ```

1. Commit.

✅ **Checkpoint:** `/1-1-meeting-agenda` asks you for the period, and `common-agents` has authoring rules.

### Exercise 2.2: Create a code review prompt

1. Create `.github/prompts/review-code.prompt.md`:

   ````markdown
   ---
   description: 'Review code changes against project standards'
   agent: 'agent'
   tools: ['terminal']
   ---

   Review committed changes on this branch. DO NOT modify files.

   ## Find changes

   Use git to find changed files:

   ```bash
   git diff --name-only $(git merge-base main HEAD)...HEAD
   ```

   ## Standards

   Read and apply `.github/copilot-instructions.md` and every
   `.github/instructions/*.instructions.md` file in this workspace whose
   `applyTo` pattern matches a changed file.

   ## Focus

   - Violations of coding standards
   - Security concerns
   - Error handling gaps
   - Defensive programming: null/undefined access
   ````

1. Create a branch and commit a change that breaks one of the authoring rules from Exercise 2.1. For example, rename the `visualize` skill's directory to `visualise` without changing its `name`.
1. In Chat, type `/review-code` and press <kbd>Enter</kbd>.

✅ **Checkpoint:** The review reports that the `name` and the directory don't match, and no files change. Undo the rename afterward.

> [!TIP]
> Tool names vary between VS Code versions. If `terminal` isn't recognized, use the **Configure Tools** picker in the prompt file editor to insert valid tool names.

### Exercise 2.3: Create an issue summarizer with input variables

This prompt reads issues through the GitHub MCP server, so make sure the server is enabled in VS Code.

1. Create `.github/prompts/issues-summarizer.prompt.md`:

   ```markdown
   ---
   name: issues-summarizer
   description: Summarize the most active issues in a repository for engineering review
   ---

   Summarize top 20 GitHub issues in repo ${input:repo:repo name as @owner/repo} filtered by activity date/last_updated and their comments for software engineers to review.
   Ignore recurring status report issues.
   Sort the ones that require more attention. If an issue has not seen an update for a while it should be prioritized higher but not as high as something that is a hot topic.
   Use all relevant URLs and links in the issue and comments.
   Always reference the users based on their GitHub usernames.
   You should not alter the facts.
   Your goal is to generate a comprehensive summary.
   Format your response with the following fields:
   **Summary**: "<PLACEHOLDER: an executive summary of the issue, and the outcomes of the investigations so far>"
   **Investigation Details**: "<PLACEHOLDER: a breakdown of all the investigation steps taken so far, include the users who conducted the investigations (comment authors), and the outcome of each. Use bulleted lists if necessary>"
   **Next Steps**: "<PLACEHOLDER: outcomes of the investigations and next steps needed to move the investigation forward or wrap up the work>"
   **Pending Items**: ["item1", "item2", ...] If there are no pending items, return an empty array.
   ```

1. Run it with `/issues-summarizer`. When prompted, enter a busy public repository, for example `@microsoft/vscode`. You can also pass the value inline: `/issues-summarizer repo=@microsoft/vscode`.

✅ **Checkpoint:** You get one structured block per issue, each with all four fields.

### Exercise 2.4: Discuss the sort-order problem

Look closely at the order of the issues in the output from Exercise 2.3.

1. Is the list really sorted by "needs attention"? Is a stale issue ranked above a hot one?
1. With your table, discuss why a prompt can't guarantee a deterministic sort. The model ranks the issues by reading them, not by calculating.
1. Sketch how a **skill** fixes this. A `scripts/rank_issues.py` script fetches the issues, computes a score from last-updated age and comment activity, and returns a sorted list. The model then only writes the summaries.

✅ **Checkpoint:** You can explain when a task needs a prompt file and when it needs a skill with a script. You'll build this skill in Exercise 3.2.

### Exercise 2.5: Turn the review prompt into a shared skill

`/review-code` only works in VS Code, and only while `common-agents` is the open workspace. As a skill, it works in the CLI, the Copilot app, and the cloud agent. Because of `skillDirectories`, it also works in every repository.

1. Create `.github/skills/code-review/SKILL.md`:

   ````markdown
   ---
   name: code-review
   description: Reviews committed changes on the current branch against the repository's custom instructions. Use when asked to review a branch, a diff, or changes before a pull request.
   ---

   Review committed changes on this branch. Do not modify files.

   1. Find changed files:

      ```bash
      git diff --name-only $(git merge-base main HEAD)...HEAD
      ```

   2. Read `.github/copilot-instructions.md` and each `.github/instructions/*.instructions.md`
      file whose `applyTo` pattern matches a changed file.
   3. Report, per file: standards violations, security concerns, error handling gaps,
      and possible null or undefined access. Include a suggested fix for each finding.
   ````

1. Commit and push `common-agents`.
1. Go to the viewer repo, create a branch, and commit a change that breaks a Lab 1 rule (for example, a hard-coded dimension in `src/`):

   ```bash
   cd ~/projects/iphone-duo-viewer
   git switch -c test-review
   # make and commit the change
   copilot
   ```

1. In the CLI, run:

   ```copilot
   /skills info code-review
   Use the /code-review skill on this branch.
   ```

✅ **Checkpoint:** `/skills info` shows the skill's location in `common-agents`, and the review flags the magic number using the *viewer's* `threejs.instructions.md`.

> [!TIP]
> VS Code has an experimental prompt-file migration that converts prompt files to skills. See [Migrate prompt files to skills](https://code.visualstudio.com/docs/agent-customization/overview#migrate-prompt-files-to-skills).

**Real-world pattern:** A platform team keeps `generate-unit-tests`, `review-code`, and `create-readme` in its central repository. These cover the three tasks new hires ask about most. Publishing them as skills, not prompt files, makes them work in every client the team uses.

---

## Lab 3: Agent skills

**Goal:** Study a well-built skill that bundles instructions, a script, reference material, and a template. Then build your own with `/create-skill`, and install community skills.

**Repo:** `common-agents` · **Surface:** CLI or Copilot app

### Exercise 3.1: Take apart the `release-validator` skill

`release-validator` came preloaded. It's the reference example of how to build a skill.

1. Look at its layout:

   ```text
   .github/skills/release-validator/
   ├── SKILL.md                    # when to use it, plus a five-step procedure
   ├── scripts/validate.py         # deterministic check: every box ticked, no placeholder approvers
   ├── references/release-rules.md # what counts as "done" and "approved"
   └── assets/release-template.md  # the report to fill in
   ```

1. Open `SKILL.md` and find these patterns:

   * The `description` lists the phrases that should trigger the skill ("validate a release", "is this release ready to ship").
   * `argument-hint` tells the user what to provide.
   * The body links to `references/` and `assets/` instead of copying them, so Copilot loads that detail only when the skill runs.
   * Step 5 says to *stop* if `validate.py` fails, so the model can't talk its way past the check.

1. Start a session in `common-agents` and test the skill with a release that's missing an approval:

   ```bash
   cd ~/projects/common-agents
   copilot
   ```

   ```copilot
   Use the /release-validator skill to check release v2.4.0. Testing and security checks passed, docs are done, and Product, Marketing, and Revenue approved.
   ```

1. Open the generated report, tick the Security box and fill in an approver, then ask Copilot to validate again.

✅ **Checkpoint:** The first run stops with `validate.py` failures for the Security approval. The second run prints `PASS`.

**Discussion:** Which parts of this skill would you trust to the model, and which to a script? Why is "every box is ticked" a script's job?

### Exercise 3.2: Build an `issue-triage` skill with `/create-skill`

Now fix the sort-order problem from Exercise 2.4 with a skill whose script does the ranking.

1. In the same session, run `/create-skill` and paste:

   ````markdown
   Create an issue-triage skill in .github/skills/. Follow the same pattern as the
   release-validator skill in this repository. Use this layout:

   ```text
   issue-triage/
   ├── SKILL.md
   ├── scripts/
   │   └── rank_issues.py
   ├── references/
   │   └── scoring.md
   └── assets/
       └── summary-template.md
   ```

   What the skill should do:
   - Take a repository as @owner/repo.
   - scripts/rank_issues.py fetches open issues updated in the last 30 days with
     `gh issue list --json number,title,updatedAt,comments,labels,url`, skips recurring
     status-report issues, and prints them sorted by an attention score. The score
     favors hot topics (many recent comments) first and stale issues second, as
     defined in references/scoring.md.
   - The model summarizes only the top 20, in the ranked order, using
     assets/summary-template.md (Summary, Investigation Details, Next Steps,
     Pending Items). It must not reorder them.
   - Always reference users by their GitHub usernames and never alter facts.
   ````

1. Review the generated files against the authoring rules from Exercise 2.1:

   * `name: issue-triage` matches the directory.
   * The `description` says when to use the skill.
   * `SKILL.md` links to the script, the reference file, and the template.
   * `rank_issues.py` runs by itself: `python3 .github/skills/issue-triage/scripts/rank_issues.py @microsoft/vscode`.

1. Run `/skills reload`, then:

   ```copilot
   Use the /issue-triage skill on @microsoft/vscode.
   ```

1. Commit and push.
1. Start a session in `~/projects/iphone-duo-viewer` and run `/skills info issue-triage`.

✅ **Checkpoint:** The summaries appear in the same order as the script's output, and the skill also loads from the viewer repo through `skillDirectories`.

🧯 **Troubleshooting (Lab 3)**

* **The skill doesn't load.** The `name` must be 1–64 characters of lowercase letters, numbers, and hyphens, and must match the directory name exactly.
* **Copilot doesn't pick the skill automatically.** At first, only `name` and `description` load. Make the description specific about when to use the skill.
* **`SKILL.md` is getting long.** Keep it under 500 lines and move detail into `references/`. The detail loads only when the task needs it.
* **A skill has `tools` in its frontmatter.** The preloaded `root-instructions` skill does. The GitHub Docs list `name`, `description`, `license`, and `allowed-tools` as the `SKILL.md` fields. Use `allowed-tools` to pre-approve tools, and don't rely on the others.

### Exercise 3.3: Install community skills

1. Search for and preview skills with GitHub CLI before you install them:

   ```bash
   gh skill search grill
   gh skill search web-design
   gh skill preview SKILL-NAME
   gh skill install SKILL-NAME
   ```

   Or use **Customize → Skills** in the Copilot app. If the installer asks where to put the skill, choose `~/projects/common-agents/.github/skills` so it lives in your team repo.

1. Install **Grill me** and **Web Design Guidelines**. You'll use Web Design Guidelines for the viewer's controls in Lab 8.
1. Check whether the **impeccable design** skill is already installed:

   ```copilot
   /skills list
   ```

1. Try them out:

   * `Grill me on my plan to roll out common-agents to 40 repositories.`
   * From the viewer repo: `Review the Knolling toggle against the web design guidelines.`

✅ **Checkpoint:** `/skills list` shows all three skills, and each one changes how Copilot responds.

> [!WARNING]
> Always preview third-party skills before you install them. Don't pre-approve `shell` or `bash` in `allowed-tools` for a skill unless you've reviewed its scripts and trust the source.

🚀 **Stretch goal:** Use the preloaded `js-to-typescript` skill (from `.claude/skills`) on a small JavaScript project, and check with `/skills info js-to-typescript` which location it loaded from.

---

## Lab 4: Custom agents

**Goal:** Harden the preloaded agent, build team agents in `common-agents`, use them from the viewer repo, add a personal agent, and design a multi-agent workflow.

**Repo:** `common-agents` · **Surface:** CLI, Copilot app, GitHub.com

### Exercise 4.1: Harden the `code-explainer` agent

The preloaded `code-explainer` agent says "Do not edit code", but it has no `tools` list, so it can use every tool, including edit and shell. Instructions ask for behavior; the `tools` list enforces it.

1. Open `~/projects/common-agents/.github/agents/code-explainer.agent.md` and look at three problems:

   * `tools` is commented out, so every tool is allowed.
   * The `description` is a persona ("You are an expert..."), not a statement of what the agent does and when to pick it. Copilot uses the description to choose agents.
   * The body tells the agent to use `#tool:vscode.mermaid-chat-features/renderMermaidDiagram`, which exists only in VS Code.

1. Fix the frontmatter:

   ```markdown
   ---
   name: code-explainer
   description: Explains an unfamiliar codebase, including its structure, architecture, domain model, build and test steps, and hotspots, without changing any code. Use when onboarding to a repository or asking how something works.
   argument-hint: A repository, folder, or file to explain, or a question about how the code works
   tools: ['read', 'search']
   ---
   ```

1. In the body, change the Mermaid line to: `Visualize the architecture with Mermaid diagrams in fenced code blocks.`
1. Commit and push. Restart the CLI. Because `~/.copilot/agents` links to this folder, the change applies everywhere.
1. In the viewer repo, run `/agent`, select **code-explainer**, and ask: `Explain how this app's scene, parts, and layout modules fit together.`
1. Ask it to `Fix any bugs you find.`

✅ **Checkpoint:** The explanation includes a Mermaid diagram, and the agent refuses, or isn't able, to edit files when you ask it to fix bugs.

### Exercise 4.2: Create a `readme-specialist` team agent

1. Create `~/projects/common-agents/.github/agents/readme-specialist.agent.md`:

   ```markdown
   ---
   name: readme-specialist
   description: Creates and improves repository README documentation
   tools: ['read', 'search', 'edit']
   ---

   Focus exclusively on README and supporting documentation.
   Do not modify source code. Prefer concise, task-oriented examples.
   ```

1. Add a persona, a workflow, and clear boundaries below the frontmatter.
1. Test it on its own repo. Start `copilot` in `common-agents`, run `/agent`, choose **readme-specialist**, and ask it to `Write a README that explains how to use this repository's skills and agents`.
1. Commit and push, then restart the CLI. You don't need to link anything; the new file is already in the linked folder.
1. Start `copilot` in `iphone-duo-viewer`, select **readme-specialist**, and ask it to `Write a README for this project`.

✅ **Checkpoint:** The agent works in both repos, edits only documentation, and refuses to change source code.

> [!NOTE]
> Keep `tools` to the minimum set. `tools: []` doesn't mean "read-only". It removes every tool.

**Optional (organization owners): promote it to the organization.** Copy the file to `agents/readme-specialist.agent.md` in your organization's `.github-private` repository and commit it to the default branch. The agent then appears for everyone in the organization on GitHub.com, with no local setup.

### Exercise 4.3: Create a personal agent

A personal agent is just for you. You save it to `~/.copilot/agents/`, which is linked to `common-agents/.github/agents`. The `my-` prefix matches the `.gitignore` rule from Exercise 0.4, so the file stays on your machine and never gets committed to the team repo.

1. Create `~/.copilot/agents/my-review-style.agent.md`:

   ```markdown
   ---
   name: my-review-style
   description: Reviews changes using my concise, fix-oriented format
   tools: ['read', 'search']
   ---

   Review only the requested changes. Every finding must include a
   specific suggested fix. Omit praise and general summaries.
   ```

1. Restart the CLI.
1. Confirm Git ignores the file:

   ```bash
   cd ~/projects/common-agents && git status --short .github/agents
   # No output for my-review-style.agent.md
   ```

1. In the viewer repo, run `/agent`, select **my-review-style**, and review the `test-review` branch from Exercise 2.5.

✅ **Checkpoint:** Every finding has a suggested fix, and there's no praise or summary.

**Precedence experiment:** In the viewer repo, create `.github/agents/my-review-style.agent.md` with the same name, but tell it to write long, detailed reviews. Restart the CLI and run it again. Which one ran?

> [!NOTE]
> The GitHub Docs disagree on this. The CLI configuration directory reference and the CLI command reference both say project agents (`.github/agents/`) take precedence over personal agents (`~/.copilot/agents/`). The CLI how-to for creating agents says the agent in your home directory wins. Record what your CLI version does, and give team and personal agents different names so it never matters.

For repository, organization, and enterprise agents, the repository agent overrides the organization agent, which overrides the enterprise agent:

```text
Enterprise:   .github-private/agents/reviewer.agent.md
Organization: .github-private/agents/reviewer.agent.md
Repository:   .github/agents/reviewer.agent.md  ← wins
```

### Exercise 4.4: Design a multi-agent handoff

1. Draw this workflow for a real viewer feature, such as "Add a hinge fold animation":

   ```text
   Coordinator
    ├─ Researcher ──┐
    ├─ Security  ───┼─ fan-out → consolidated plan
    └─ Architect ───┘
                           │ human approval
                           ▼
                      Implementer
                           ▼
                    Read-only Reviewer
   ```

1. For each stage, write the input and output contract (what it receives and what it must return).
1. Fan out only tasks that are independent.
1. Give each worker the minimum tools. For example, the Researcher and Reviewer get `read` and `search` only. Reuse your hardened `code-explainer` as the Researcher.
1. Create the agents in `common-agents/.github/agents/`:
   * `coordinator.agent.md`, whose `agents` field lists the workers it can delegate to.
   * One file for each worker. Set `user-invocable: false` to hide the workers from the picker; the coordinator can still delegate to them.
   * On the Implementer and Reviewer, add `include-custom-instructions: true`. Subagents don't follow the repository's custom instructions unless you add this.
1. Add a human approval step between the consolidated plan and the Implementer.
1. Commit, push, and restart the CLI.

✅ **Checkpoint:** Run the coordinator in the viewer repo. You see separate subagent runs for the fan-out, it stops for your approval before it edits anything, and the Implementer follows the viewer's `threejs.instructions.md`.

### Exercise 4.5: See personal instructions shape agent output

This exercise reuses the personal instructions from Exercise 1.7.

1. Open [Copilot Chat on GitHub.com](https://github.com/copilot) in your personal space.
1. Ask:

   ```text
   How can I trace Agent and Sub agent calls in a Copilot session in VSCode.
   ```

1. Follow up with:

   ```text
   Show me an example code that will parse the log and create a mermaid diagram of the calls
   ```

✅ **Checkpoint:** The parser is a TypeScript command-line program, and the response follows your plain-language rules. Run it on the log from your Exercise 4.4 coordinator run and paste the Mermaid output into a Markdown preview.

🧯 **Troubleshooting (Lab 4)**

* **Your agent doesn't show up.** Check that the file ends in `.agent.md`, that `ls -l ~/.copilot/agents` shows the link to `common-agents/.github/agents`, and that you restarted the CLI. For org agents, check that the file is on the default branch.
* **An agent is listed twice inside `common-agents`.** In that repo, the same files load as project agents and, through the link, as personal agents. The project copy takes precedence. Outside `common-agents`, each agent loads once.
* **A different agent runs than the one you expected.** A same-named agent at another level is shadowing yours. See the precedence experiment.
* **The agent does more than you allowed.** Check the `tools` list. It's an allowlist, and leaving it out allows every tool (as in the original `code-explainer`).

---

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

✅ **Checkpoint:** `/mcp show context7` lists its tools, and the answer includes a Context7 tool call.

> [!TIP]
> MCP servers configured for a repository or for the CLI are also available in the Copilot app. If Context7 can't connect in the app, the app may not see your shell's environment variables. Start the app from a terminal with `copilot app`.

### Exercise 5.3: Check the viewer with the built-in Playwright server

The CLI includes a built-in `playwright` MCP server for browser automation. You don't need to configure it.

1. In the viewer repo session, ask:

   ```text
   Start the dev server, open the viewer with Playwright, take a screenshot, then click
   the Knolling button and take another. Stop the dev server when you're done.
   ```

✅ **Checkpoint:** You get two screenshots of the viewer. In Lab 8, you'll use the same technique to check that knolled parts don't overlap.

🧯 **Troubleshooting (Lab 5)**

* **The server doesn't appear.** Business and Enterprise organizations need the **MCP servers in Copilot** policy turned on. It's off by default.
* **The server is blocked.** Your organization may use an MCP registry allowlist. Only servers on the allowlist can run.
* **VS Code doesn't see the server.** The CLI doesn't read `.vscode/mcp.json`, and VS Code uses a different top-level key (`servers`). If you also want the server in VS Code, add a `.vscode/mcp.json` in the VS Code format.

---

## Lab 6: Hooks

**Goal:** Enforce guardrails in the viewer repo with a `preToolUse` hook and format files automatically with a `postToolUse` hook. Hooks are deterministic: they run every time, whatever the model decides.

**Repo:** `iphone-duo-viewer` · **Surface:** CLI, cloud agent

**Requirements:** `jq` installed locally.

### Exercise 6.1: Publish the viewer repo to GitHub

The cloud agent (Exercise 6.5) and the pull request in Lab 8 need the repo on GitHub:

```bash
cd ~/projects/iphone-duo-viewer
gh repo create YOUR-ORG/iphone-duo-viewer --private --source=. --remote=origin --push
```

✅ **Checkpoint:** `git remote -v` shows `origin` pointing to `YOUR-ORG/iphone-duo-viewer`.

### Exercise 6.2: Create the hook configuration

1. Install Prettier for the formatting hook:

   ```bash
   npm install --save-dev prettier
   ```

1. Create `.github/hooks/guardrails.json`:

   ```json
   {
     "version": 1,
     "hooks": {
       "preToolUse": [{
         "type": "command",
         "matcher": "bash|edit|create",
         "bash": "./.github/hooks/guard.sh",
         "timeoutSec": 10
       }],
       "postToolUse": [{
         "type": "command",
         "matcher": "edit|create",
         "bash": "./.github/hooks/format.sh"
       }]
     }
   }
   ```

### Exercise 6.3: Write the hook scripts

1. Create `.github/hooks/guard.sh`. It reads `toolName` and `toolArgs` from the JSON payload on standard input:

   ```bash
   #!/usr/bin/env bash
   INPUT=$(cat)
   TOOL=$(echo "$INPUT" | jq -r '.toolName')
   ARGS=$(echo "$INPUT" | jq -r '.toolArgs | if type == "string" then . else tojson end')

   deny() {
     jq -cn --arg r "$1" '{permissionDecision: "deny", permissionDecisionReason: $r}'
     exit 0
   }

   case "$TOOL" in
     bash)
       echo "$ARGS" | grep -Eq 'rm -rf|git push (-f|--force)' \
         && deny "Blocked by guardrails: destructive shell command" ;;
     edit|create)
       echo "$ARGS" | grep -Eq '\.env|src/specs\.ts' \
         && deny "Blocked by guardrails: .env files and src/specs.ts are protected" ;;
   esac
   exit 0
   ```

   Empty output means "no decision", so the normal permission flow continues. Protecting `src/specs.ts` keeps the agent from "fixing" dimensions so they match a broken layout.

1. Create `.github/hooks/format.sh`:

   ```bash
   #!/usr/bin/env bash
   INPUT=$(cat)
   FILE=$(echo "$INPUT" | jq -r '.toolArgs | (if type == "string" then fromjson else . end) | .path // empty')
   if [ -n "$FILE" ] && [ -f "$FILE" ]; then
     npx --no-install prettier --write "$FILE" >/dev/null 2>&1
   fi
   exit 0
   ```

1. Make both scripts executable:

   ```bash
   chmod +x .github/hooks/guard.sh .github/hooks/format.sh
   ```

1. Test the guard locally before you use it with Copilot:

   ```bash
   echo '{"toolName":"bash","toolArgs":"{\"command\":\"rm -rf dist\"}"}' | ./.github/hooks/guard.sh
   # Expected: {"permissionDecision":"deny","permissionDecisionReason":"Blocked by guardrails: destructive shell command"}

   echo '{"toolName":"bash","toolArgs":"{\"command\":\"ls\"}"}' | ./.github/hooks/guard.sh
   # Expected: no output
   ```

1. Commit and push.

> [!TIP]
> Argument field names can differ between tools and CLI versions. To see the real payload, add `echo "$INPUT" >> /tmp/hook-payload.log` near the top of a script, then run one tool call.

### Exercise 6.4: Watch the hooks work in the CLI

1. Start `copilot` in the viewer repo.
1. Ask: `Clean up the build output with rm -rf dist`.
1. Ask: `Change the closed depth in src/specs.ts to 12 mm`.
1. Ask: `Add a comment to the top of src/main.ts`.

✅ **Checkpoint:** The first two requests are denied with your reasons. The third one edits the file, and the file is reformatted afterward.

> [!NOTE]
> The hooks documentation covers the Copilot CLI and the cloud agent. It doesn't say whether the Copilot app runs repository hooks, so test your guardrails in the CLI.

### Exercise 6.5: Run the hooks with the Copilot cloud agent

1. Make sure `.github/hooks/` is on the default branch. The cloud agent loads hooks only from `.github/hooks/*.json` in the cloned repository.
1. Create an issue in `YOUR-ORG/iphone-duo-viewer`, for example "Remove the `dist` directory and make the closed depth 12 mm", and assign it to Copilot.
1. Open the session logs.

✅ **Checkpoint:** The logs show the denied tool calls and your denial reasons.

🧯 **Troubleshooting (Lab 6)**

* **Hooks don't run.** Check that the file is in `.github/hooks/`, that it's valid JSON (`jq . .github/hooks/guardrails.json`), that it has `"version": 1`, and that the scripts are executable and have a shebang.
* **The guard didn't block anything.** Timeouts **fail open**: if the hook takes longer than `timeoutSec`, the tool call continues. Keep the guard well under 10 seconds.
* **Output is ignored.** Emit exactly one JSON object on stdout. Two `echo` calls with JSON produce invalid output, which is ignored.

> [!IMPORTANT]
> A crash or non-zero exit in a command `preToolUse` hook denies the tool call (fail-closed). A timeout doesn't (fail-open). Design your guard with both behaviors in mind.

🚀 **Stretch goal:** Add a `sessionStart` hook that logs the start time to `logs/session.log`, or an `agentStop` hook that plays a sound when the agent finishes.

---

## Lab 7: Plugins

**Goal:** Graduate the best skills and agents from `common-agents` into a plugin, publish `common-agents` as a marketplace, and install the plugin into the viewer workflow. This replaces the manual setup from Exercise 0.4.

**Repo:** `common-agents` · **Surface:** CLI, Copilot app

### Exercise 7.1: Discover and install a community plugin

```bash
# Discover
copilot plugin marketplace list
copilot plugin marketplace browse awesome-copilot

# Install and inspect
copilot plugin install database-data-management@awesome-copilot
copilot plugin list

# Maintain
copilot plugin update --all
copilot plugin disable database-data-management
```

✅ **Checkpoint:** `copilot plugin list` shows the plugin, and `/skills list` inside a session shows its skills.

### Exercise 7.2: Move skills and agents into a plugin

You'll **move** the files, not copy them. If the same skill is loaded from both `skillDirectories` and a plugin, you'll get duplicates.

1. Create the plugin layout (Agent Plugins 1.0 format) and move the files:

   ```bash
   cd ~/projects/common-agents
   mkdir -p plugins/workshop-kit/skills plugins/workshop-kit/com.github.copilot/agents

   git mv .github/skills/release-validator plugins/workshop-kit/skills/
   git mv .github/skills/code-review       plugins/workshop-kit/skills/
   git mv .github/skills/issue-triage      plugins/workshop-kit/skills/
   git mv .github/agents/readme-specialist.agent.md plugins/workshop-kit/com.github.copilot/agents/
   git mv .github/agents/code-explainer.agent.md    plugins/workshop-kit/com.github.copilot/agents/
   ```

   Because `~/.copilot/agents` links to `.github/agents`, moving the two agents out also removes them from your personal agents. From now on, the plugin provides them.

   The result:

   ```text
   common-agents/
   ├── .github/
   │   ├── skills/        # incubating skills (still loaded through skillDirectories)
   │   └── agents/        # incubating agents (coordinator and workers)
   └── plugins/
       └── workshop-kit/
           ├── plugin.json
           ├── skills/
           │   ├── release-validator/
           │   ├── code-review/
           │   └── issue-triage/
           └── com.github.copilot/
               └── agents/
                   ├── readme-specialist.agent.md
                   └── code-explainer.agent.md
   ```

1. Create `plugins/workshop-kit/plugin.json`:

   ```json
   {
     "$schema": "https://agent-plugins.org/schemas/1.0.0/plugin.schema.json",
     "name": "YOUR-HANDLE-workshop-kit",
     "description": "Release validation, code review, issue triage, and README and code-explainer agents",
     "version": "1.0.0",
     "author": { "name": "YOUR NAME" },
     "license": "MIT"
   }
   ```

1. Install the plugin from the local folder and check what loaded:

   ```bash
   copilot plugin install ./plugins/workshop-kit
   ```

   Then, inside a session in the viewer repo, run `/plugin list`, `/agent`, and `/skills info release-validator`.

1. Uninstall the local copy so you can install it from the marketplace next. Use the `name` from `plugin.json`, not the folder path:

   ```bash
   copilot plugin uninstall YOUR-HANDLE-workshop-kit
   ```

✅ **Checkpoint:** While the local plugin was installed, the three skills and both agents came from `YOUR-HANDLE-workshop-kit`.

> [!IMPORTANT]
> Installed plugins are cached. If you edit a plugin that you installed from a local folder, run `copilot plugin install` again to pick up the changes.

### Exercise 7.3: Publish `common-agents` as a marketplace

1. Create `common-agents/.github/plugin/marketplace.json`:

   ```json
   {
     "name": "YOUR-HANDLE-common-agents",
     "owner": { "name": "YOUR NAME" },
     "metadata": { "description": "Team skills and agents", "version": "1.0.0" },
     "plugins": [
       {
         "name": "YOUR-HANDLE-workshop-kit",
         "description": "Release validation, code review, issue triage, and README and code-explainer agents",
         "version": "1.0.0",
         "source": "plugins/workshop-kit"
       }
     ]
   }
   ```

1. Commit and push:

   ```bash
   git add -A && git commit -m "Publish workshop-kit plugin and marketplace" && git push
   ```

1. Add the marketplace and install the plugin:

   ```bash
   copilot plugin marketplace add YOUR-ORG/common-agents
   copilot plugin install YOUR-HANDLE-workshop-kit@YOUR-HANDLE-common-agents
   ```

1. In the Copilot app, click **Customize → Plugins**, click the gear icon next to the marketplace dropdown, and add `YOUR-ORG/common-agents`. Install the plugin there too.
1. Have a teammate add your marketplace and install the plugin, with no `skillDirectories` or links.

✅ **Checkpoint:** In the viewer repo, `/skills info code-review` shows the plugin as its source, and your teammate's `copilot plugin list` shows `YOUR-HANDLE-workshop-kit`.

**Discussion:** `common-agents` now works in two ways. `.github/skills` and `.github/agents` are where you incubate new work, loaded for you through `skillDirectories` and links. `plugins/` holds reviewed, versioned releases that anyone installs from the marketplace. When an incubating skill is ready, move it into the plugin, bump `version`, and push. Subscribers pick it up with `copilot plugin update`.

🚀 **Stretch goals**

* **Publish a second plugin.** Package the four instruction-generator skills as `instructions-kit`: `root-instructions`, `area-instructions`, `nested-hub`, and `nested-detail`. Move them into `plugins/instructions-kit/skills/`, add a `plugin.json`, and add a second entry to the `plugins` array in `marketplace.json`. Subscribers can then install just the kit they need. Before you publish, check the skills against the Exercise 2.1 authoring rules. For example, `root-instructions` has a `tools` field and mentions VS Code-only tools.
* **Auto-install for contributors.** Add the marketplace and plugin to the viewer repo's `.github/copilot/settings.json` with `extraKnownMarketplaces` and `enabledPlugins`, so every contributor gets the kit automatically. See [Configuration file settings](https://docs.github.com/en/copilot/reference/copilot-cli-reference/cli-config-dir-reference#configuration-file-settings) for the exact format.

---

## Lab 8: Keystone: iPhone Duo knolling viewer

**Goal:** Use everything from Labs 1–7 to turn the viewer scaffold into an interactive 3D model of the iPhone Duo. It has an exploded view and a *knolling* mode that lays every component flat in an aligned, non-overlapping grid.

**Repo:** `iphone-duo-viewer` · **Surface:** Copilot app (CLI for checks)

### Background

Apple announced the iPhone Duo at Apple Park on September 9, 2026, alongside the iPhone 18 Pro and iPhone 18 Pro Max. It's scheduled for release on October 23, 2026. Use the official Tech Specs page as the source of truth for dimensions:

| Measurement | Open | Closed |
|-------------|------|--------|
| Width | 164.6 mm (6.48 in) | 84.1 mm (3.31 in) |
| Height | 117.8 mm (4.64 in) | 117.8 mm |
| Depth | 5.2 mm (0.21 in) | 11.3 mm (0.44 in) |
| Weight | — | 254 g (8.96 oz) |

| Display | Size | Resolution | Density |
|---------|------|------------|---------|
| Inner folding OLED | 7.6 in | 1878 × 2670 | 430 ppi |
| Outer OLED | 5.4 in | 1398 × 2034 | 460 ppi |

The Tech Specs page also covers the chip, cameras, battery, modem, and a diagram of the buttons and connectors.

### Exercise 8.1: Check your setup (10 min)

Everything should already be in place. Confirm it in a CLI session in the viewer repo:

| From | What | Check |
|------|------|-------|
| Lab 1 | Repository, three.js, and Playwright instructions | `copilot instruction` |
| Lab 1 | App **Run** script | Run it in the Copilot app |
| Labs 2–3, 7 | `code-review`, `release-validator`, `issue-triage` (plugin) | `/skills list` |
| Lab 0 | `visualize` and `html-in-canvas` (preloaded) | `/skills list` |
| Lab 3 | Web Design Guidelines skill | `/skills list` |
| Labs 4, 7 | `code-explainer` and `readme-specialist` (plugin), coordinator, `my-review-style` | `/agent` |
| Lab 5 | Context7 and built-in Playwright | `/mcp` |
| Lab 6 | Guardrail and format hooks | `/env` |

✅ **Checkpoint:** Every row checks out.

### Exercise 8.2: Build the viewer (35 min)

1. In the Copilot app, start a new session for `iphone-duo-viewer` in **Plan** mode, and give it this prompt:

   ```text
   Turn this scaffold into an app to view the iPhone Duo. Do not stop until you have a
   good 3D model of the iPhone Duo.

   Use three.js and put all the parts in a grid.

   Render to an HTML canvas and wire up the Knolling toggle. When knolling is on, lay out
   every component in a non-overlapping, aligned grid with a straight-on view. When it's
   off, return smoothly to the exploded view. Add a setting to control the transition.
   Knolling means taking every piece of the phone, laying it out in a grid, and
   aligning everything perfectly.

   Use context7 for the current three.js API. Use the dimensions and display specs from
   https://www.apple.com/iphone-duo/specs/ and keep them in src/specs.ts.
   ```

1. Review the plan. Push back on anything that ignores the repository instructions. Then approve it.
1. Use the **Run** script to watch progress in the integrated browser. Iterate until the app meets the acceptance criteria.

> [!TIP]
> Your guard protects `src/specs.ts` from edits. If the agent needs to add new spec values, temporarily remove `src/specs\.ts` from the guard pattern, review the diff yourself, and then put it back.

### Acceptance criteria

* [ ] The model matches the open and closed dimensions in the tables above (within 1 mm).
* [ ] The inner (7.6 in) and outer (5.4 in) displays are separate components.
* [ ] The exploded view separates components along the device's depth axis.
* [ ] The **Knolling** toggle lays out every component flat in an aligned grid with no overlaps.
* [ ] Knolling mode switches to a straight-on (top-down) camera.
* [ ] Turning knolling off animates smoothly back to the exploded view.
* [ ] A settings control changes the transition (for example, duration or grid spacing).
* [ ] Every mesh is named after the part it represents.
* [ ] `npm test` passes.

### Exercise 8.3: Review and ship (15 min)

1. Run `/code-review` (plugin skill) and your `my-review-style` agent on the session's changes.
1. Ask `code-explainer` (plugin agent) to explain the finished architecture, and use the `visualize` skill to produce a data-flow diagram of the exploded-to-knolling transition.
1. Ask `readme-specialist` (plugin agent) to update the README with setup steps, the diagram, and screenshots. Use Playwright to capture the screenshots.
1. In the app, run `/pr-open` to open a pull request, then request a Copilot code review on it.

✅ **Checkpoint:** You demo the exploded view, the knolling toggle, and the transition setting to your table.

🚀 **Stretch goals**

* Add an `agentStop` hook that returns `{"decision": "block", "reason": "..."}` until an automated check passes (for example, a Playwright script that confirms no two knolled parts overlap). The CLI stops forcing more turns after 8 consecutive blocks.
* Add an open and closed fold animation using the hinge.
* Use the preloaded `html-in-canvas` skill to render live HTML labels (part name and dimensions) next to each knolled part inside the three.js scene. HTML-in-Canvas is an experimental Chromium API behind a flag, so keep a fallback path.
* Write a `knolling-layout` skill in `common-agents/.github/skills/`, graduate it into `workshop-kit`, bump the version, push, and run `copilot plugin update --all`.

### Sample app

Your facilitator will show screenshots of a finished sample app:


<img width="419" height="387" alt="image-20260921090605126" src="https://github.com/user-attachments/assets/cad553fe-5f60-452f-8b4c-7af082121731" />

<img width="678" height="626" alt="image-20260921090535726" src="https://github.com/user-attachments/assets/d299fcd1-c142-4d8f-82cc-051bbd3982ce" />


### References

* [Apple iPhone Duo Tech Specs](https://www.apple.com/iphone-duo/specs/)
* [iPhone Duo on Wikipedia](https://en.wikipedia.org/wiki/IPhone_Duo)
* [Phone Repair Guru: iPhone Duo](https://www.phonerepairguru.com/news/g331e94rfbdqe55zzrh4qzj9w36xyl)
* [Wccftech: Behold the Apple iPhone Duo](https://wccftech.com/behold-the-apple-iphone-duo-apples-first-2000-juggernaut-replete-with-a-7-6-inch-inner-screen-a-hinge-made-up-of-100-individual-components-and-a-dual-battery-architecture/)
* [Apple Headlines: iPhone Duo](https://www.appleheadlines.com/iphone-duo/)

---

## Further reading

* [Customization cheat sheet](https://docs.github.com/en/copilot/reference/customization-cheat-sheet)
* [Hooks reference](https://docs.github.com/en/copilot/reference/hooks-reference)
* [Creating a plugin for Copilot CLI](https://docs.github.com/en/copilot/how-tos/copilot-cli/customize-copilot/plugins-creating)
* [Adding MCP servers for Copilot CLI](https://docs.github.com/en/copilot/how-tos/copilot-cli/customize-copilot/add-mcp-servers)
* [Adding agent skills for Copilot CLI](https://docs.github.com/en/copilot/how-tos/copilot-cli/customize-copilot/add-skills)
* [Using your own LLM models in Copilot CLI](https://docs.github.com/en/copilot/how-tos/copilot-cli/customize-copilot/use-byok-models)
