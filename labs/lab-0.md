## Lab 0: Prerequisites and setup

**Goal:** Confirm your plan, sign in to every client, import and clone `common-agents`, and connect it to your personal Copilot configuration.

### Exercise 0.1: Verify your plan tier

1. Go to [**github.com → Settings → Billing and licensing**](https://github.com/settings/copilot/features).
1. Under "GitHub Copilot", note the plan name and the license source (personal or granted by an organization).
1. Your Copilot license would be granted by your GitHub Enterprise.

> [!IMPORTANT]
> ✅ **Checkpoint:** You know which plan you have, and which labs you can run end to end (see the [lab table in the README](../README.md#how-the-pieces-fit-together)).

### Exercise 0.2: Sign in to every client

Your workshop laptop already has VS Code (with the GitHub Copilot extension), GitHub CLI, the Copilot CLI, and the GitHub Copilot app installed. Sign in to each one with the account that holds your Copilot license.

#### VS Code

Open the Chat view (**View → Chat**, or <kbd>Cmd</kbd>+<kbd>Shift</kbd>+<kbd>I</kbd>). Sign in if prompted, then send `Hello`.

#### GitHub CLI

Check the sign-in status, and sign in if needed:

```bash
gh auth status || gh auth login
```

#### Copilot CLI

Start a session, sign in, and confirm the account:

```bash
copilot
```

```copilot
/login
/user
```

#### Copilot app

Open the app and click **Sign in to GitHub** if prompted. Skip repository selection during onboarding; you'll add projects in Exercise 0.3 and Lab 1.

> [!IMPORTANT]
> ✅ **Checkpoint:** VS Code Chat replies to `Hello`, `gh auth status` shows you're signed in, `/user` shows your GitHub username, and the Copilot app shows your account.

### Exercise 0.3: Get your copy of `common-agents`

`common-agents` comes preloaded with some prompt files, skills, and agents from your facilitator. You'll add to it in Labs 2–4 and publish from it in Lab 7, so you need **your own copy that you can push to**. Import <https://github.com/decyjphr-org/common-agents> into `YOUR-ORG`, naming your repository **`common-agents`**.

Create your projects folder before choosing a route:

```bash
mkdir -p ~/projects
```

**Windows 11 Pro (Command Prompt):**

```cmd
if not exist "%USERPROFILE%\projects" mkdir "%USERPROFILE%\projects"
```

Use the recommended import route below. Optional fork routes are at the end of this lab; all routes end with your copy cloned to `~/projects/common-agents`.

#### Option A: GitHub UI and Git (Recommended)

1. On GitHub.com, click **+ → Import repository**, or open <https://github.com/new/import>.
1. In **Your old repository's clone URL**, enter `https://github.com/decyjphr-org/common-agents`.
1. Under **Your new repository details**, choose `YOUR-ORG` as the owner and enter `common-agents` as the repository name.
1. Choose the visibility allowed by your organization, then click **Begin import**. Wait for the import to finish and open `YOUR-ORG/common-agents`.
1. Clone it:

   ```bash
   cd ~/projects
   git clone https://github.com/YOUR-ORG/common-agents.git
   cd common-agents
   ```

   **Windows 11 Pro (Command Prompt):**

   ```cmd
   cd %USERPROFILE%\projects
   git clone https://github.com/YOUR-ORG/common-agents.git
   cd common-agents
   ```

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

> [!IMPORTANT]
> ✅ **Checkpoint:** `ls ~/projects/common-agents/.github` shows `agents`, `prompts`, and `skills`.

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

1. Create the parent directory if needed and open the settings file in VS Code:

   ```bash
   mkdir -p ~/.copilot
   code ~/.copilot/settings.json
   ```

   **Windows 11 Pro (Command Prompt):**

   ```cmd
   if not exist "%USERPROFILE%\.copilot" mkdir "%USERPROFILE%\.copilot"
   code "%USERPROFILE%\.copilot\settings.json"
   ```

   You can also run `/settings` inside the CLI.
1. Add `skillDirectories`, keeping any settings that are already in the file. Merge these paths into an existing `skillDirectories` array rather than replacing its entries, and save:

   ```jsonc
   {
     "skillDirectories": [
       "/Users/YOUR-USER/projects/common-agents/.github/skills",
       "/Users/YOUR-USER/projects/common-agents/.claude/skills"
     ]
   }
   ```

   In Windows this would be:
   ```jsonc
   {
      "skillDirectories": [
       "C:\\Users\\YOUR-USER\\projects\\common-agents\\.github\\skills",
       "C:\\Users\\YOUR-USER\\projects\\common-agents\\.claude\\skills"
      ]
   }
   ```

   * Use absolute paths.
   * Point at the folder that **contains** the skill folders (`.github/skills`, plural). Don't point at a single skill.
   * List both folders. If you leave out `.claude/skills`, `js-to-typescript` works inside `common-agents` but nowhere else.
   * The environment variable `COPILOT_SKILLS_DIRS` (comma-separated) is an alternative.

#### Step 2: Link the shared agents folder

The Copilot CLI loads agents from two places: `WORKSPACE/.github/agents/` (project) and `~/.copilot/agents/` (personal). There's no setting that adds a third folder. Instead, make your personal agents folder a symbolic link to the `common-agents` agents folder. Every agent in `common-agents` then loads in every session, including agents you add later.

1. Create the link and check it. If `~/.copilot/agents` already exists, follow the **existing agents directory** instructions in Troubleshooting before running this command; don't create a link inside an existing directory.

   ```bash
   ln -s ~/projects/common-agents/.github/agents ~/.copilot/agents
   ls -l ~/.copilot/agents
   # ~/.copilot/agents -> /Users/YOUR-USER/projects/common-agents/.github/agents
   ```

   **Windows 11 Pro (Command Prompt, run as Administrator, or with Developer Mode enabled):**

   ```cmd
   mklink /D "%USERPROFILE%\.copilot\agents" "%USERPROFILE%\projects\common-agents\.github\agents"
   dir "%USERPROFILE%\.copilot"
   REM Look for a line like this in the directory listing for the parent ".copilot" folder:
   REM <SYMLINKD>     agents [C:\Users\YOUR-USER\projects\common-agents\.github\agents]
   ```

> [!NOTE]
> `mklink` only works from a `cmd.exe` prompt. Creating a symlink requires either an elevated (Administrator) prompt or Windows 11's Developer Mode turned on (**Settings → Privacy & security → For developers**).
>
> 1. Restart the CLI whenever you add or change an agent.

> [!WARNING]
> The docs list both agent locations, but they don't mention symbolic links. If the agents don't appear, remove the link (`rm ~/.copilot/agents`, or on Windows `rmdir "%USERPROFILE%\.copilot\agents"`), create a normal folder, and copy the `.agent.md` files into it instead. Lab 7 replaces this step with a plugin.

#### Step 4: Verify from outside the repository

Step 3 is optional and appears in the stretch goals at the end. You can verify skills and agents without sharing instruction files.

##### VS Code

VS Code reads personal skills and agents from `~/.copilot/skills` and `~/.copilot/agents`, so the linked agents appear there. It does **not** read the CLI's `skillDirectories` setting. Open `common-agents` itself to use its project skills in VS Code; for the `iphone-duo-viewer` labs, use the CLI or the Copilot app, which do read it.

##### GitHub CLI (+ Copilot CLI)

Check `gh auth status`, then start the Copilot CLI from a folder that has nothing to do with `common-agents`:

```bash
cd ~ && copilot
```

**Windows 11 Pro (Command Prompt):**

```cmd
cd %USERPROFILE% && copilot
```

Run these commands:

```copilot
/skills list
/agent
/env
```

Run `/skills info release-validator` and `/skills info js-to-typescript`. Both locations are inside `common-agents`. GitHub CLI provides GitHub authentication and repository commands; these slash commands run in **Copilot CLI**, not `gh`.

##### Copilot app

In the Copilot app, click **Customize**, then **Skills**. Skills configured for the CLI also show up in the app.

> [!IMPORTANT]
> ✅ **Checkpoint:** In a session started outside `common-agents`, `/skills list` includes `release-validator` and `js-to-typescript`, and `/agent` lists `code-explainer`.

### 🧯 **Troubleshooting (Lab 0)**

<details>
<summary>Show how to fix sign-in, token, and skill issues</summary>

* **VS Code says you have no Copilot license.** Sign out and sign in again with the account that holds the license.
* **The CLI rejects your token.** Classic personal access tokens (`ghp_`) aren't supported. Use `/login`, or a fine-grained PAT with the **Copilot Requests** permission in `COPILOT_GITHUB_TOKEN`.
* **The Copilot app shows an authorization error.** The app has its own policy toggle, separate from the CLI. Ask your admin to enable it.
* **`/skills list` doesn't show the common skills.** Check the path in `skillDirectories`: it must be absolute, it must exist, and it must end in `.github/skills` (or `.claude/skills` for `js-to-typescript`). Open `/settings` and look at the **Problems** tab for errors in the file.
* **The agents link was not created correctly.** On macOS/Linux, if `ls -l ~/.copilot/agents` lists files, the existing folder caused `ln -s` to create `~/.copilot/agents/agents`; first inspect that nested entry and remove it with `rm ~/.copilot/agents/agents` only if it is the mistakenly created symlink. Back up the real directory as below, then create the link again. On Windows, inspect the parent with `dir "%USERPROFILE%\.copilot"` and look for a `<SYMLINKD> agents` entry. If `mklink` reported that the file already exists, inspect and back up the real directory before rerunning it.
* **An existing agents directory blocks Step 2.** Inspect it first. If it is already a link to the correct shared folder, keep it and skip creation. If it is a link to a different folder, remove only the link, not its target. If it is a real directory, back it up without overwriting an existing backup:

   ```bash
   if [ -d ~/.copilot/agents ] && [ ! -L ~/.copilot/agents ]; then
     if [ -e ~/.copilot/agents.bak ] || [ -L ~/.copilot/agents.bak ]; then
       echo "agents.bak already exists; choose another unused backup name before continuing."
     else
       mv ~/.copilot/agents ~/.copilot/agents.bak
     fi
   fi
   ```

   **Windows 11 Pro (Command Prompt):**

   ```cmd
   if exist "%USERPROFILE%\.copilot\agents" (
     fsutil reparsepoint query "%USERPROFILE%\.copilot\agents" >nul 2>&1
     if errorlevel 1 (
       if exist "%USERPROFILE%\.copilot\agents.bak" (
         echo agents.bak already exists; choose another unused backup name before continuing.
       ) else (
         ren "%USERPROFILE%\.copilot\agents" "agents.bak"
       )
     )
   )
   ```

   Only rerun Step 2 after `agents` no longer exists. Keep the backup until you have restored any personal agents using the optional steps below.

</details>

### 🚀 **Stretch goals**

#### 🚀 **Stretch goal** Option B: Copilot CLI

<details>
<summary>Show CLI fork steps</summary>

Use this as an alternative to importing, not after cloning the imported copy.

1. Start the CLI in your projects folder:

   ```bash
   cd ~/projects
   copilot
   ```

   **Windows 11 Pro (Command Prompt):**

   ```cmd
   cd %USERPROFILE%\projects
   copilot
   ```

1. Give Copilot the task:

   ```copilot
   Fork decyjphr-org/common-agents into YOUR-ORG and clone the fork into
   ./common-agents. Then list the contents of common-agents/.github.
   ```

1. Read each tool call before you approve it. Approve each one once, and don't choose "always allow" for shell commands yet. Lab 6 covers guardrails.

> [!TIP]
> If the fork step fails, check that GitHub CLI is signed in (`gh auth status`), or fork in the GitHub UI and ask Copilot only to clone.

</details>

#### 🚀 **Stretch goal** Option C: GitHub Copilot app

<details>
<summary>Show app fork steps</summary>

Use this as an alternative to importing, not after cloning the imported copy.

1. Fork `decyjphr-org/common-agents` into `YOUR-ORG` using the GitHub UI.
1. In the app sidebar, click **+** next to **Projects**.
1. Choose the option to pick a project from GitHub, search for `YOUR-ORG/common-agents`, and clone it into `~/projects`.

</details>

#### 🚀 **Stretch goal** Explore further

<details>
<summary>Show what to look for</summary>

Spend five minutes reading the files. Look for:

* How `release-validator/SKILL.md` links to its `scripts/`, `references/`, and `assets/` files instead of copying them.
* How `nested-hub` and `nested-detail` work as a pair: the hub skill recommends topics, and the detail skill writes one file per topic.
* That the skills live in two locations. Both `.github/skills` and `.claude/skills` are valid project skill locations.

</details>

#### 🚀 **Stretch goal:** Keep personal agents out of the team repo.

<details>
<summary>Show how to keep local agents out of commits</summary>

Any agent you save to `~/.copilot/agents` is now written into `common-agents/.github/agents`. Give personal agents a `my-` prefix and tell Git to ignore them:

```bash
echo '.github/agents/my-*.agent.md' >> ~/projects/common-agents/.gitignore
```

**Windows 11 Pro (Command Prompt):**

```cmd
findstr /l /x /c:".github/agents/my-*.agent.md" "%USERPROFILE%\projects\common-agents\.gitignore" >nul 2>&1 || echo .github/agents/my-*.agent.md>>"%USERPROFILE%\projects\common-agents\.gitignore"
```

> [!NOTE]
> The `findstr /x /c:"..."` check keeps the command idempotent, so running it more than once won't duplicate the `.gitignore` entry.

If you backed up any agents using Troubleshooting, move them back with a `my-` prefix, for example `mv ~/.copilot/agents.bak/reviewer.agent.md ~/.copilot/agents/my-reviewer.agent.md` (Windows: `ren "%USERPROFILE%\.copilot\agents.bak\reviewer.agent.md" "my-reviewer.agent.md" && move "%USERPROFILE%\.copilot\agents.bak\my-reviewer.agent.md" "%USERPROFILE%\.copilot\agents\my-reviewer.agent.md"`). Use your chosen backup name if it differs from `agents.bak`, and check that the destination name isn't already in use.

Restart the CLI if you restore any agents.

</details>

#### 🚀 **Stretch goal:** Step 3 (optional): Share instruction files

<details>
<summary>Show how to configure instruction files</summary>

If `common-agents` has a folder of shared `*.instructions.md` files, add it to your shell profile:

```bash
export COPILOT_CUSTOM_INSTRUCTIONS_DIRS=/Users/YOUR-USER/projects/common-agents/PATH-TO-INSTRUCTIONS
```

**Windows 11 Pro (Command Prompt, persists for new sessions):**

```cmd
setx COPILOT_CUSTOM_INSTRUCTIONS_DIRS "C:\Users\YOUR-USER\projects\common-agents\PATH-TO-INSTRUCTIONS"
```

> [!NOTE]
> `setx` writes the variable for future Command Prompt sessions; it doesn't affect the one you ran it in. Open a new Command Prompt window afterward.

</details>

---

[← Previous: README](../README.md) · [README ↑](../README.md#own-your-copilot-hands-on-lab-exercises) · [Next: Lab 1 →](lab-1.md)
