## Lab 2: Prompt files
> [!NOTE]
> If you skipped the previous exercises, update the local copy of `common-agents` from branch `lab-1-solution` in [common-agents](https://github.com/decyjphr-org/common-agents) repo and local copy of `iphone-duo-viewer` from branch `lab-1-solution` in [iphone-duo-viewer](https://github.com/decyjphr-org/iphone-duo-viewer) repo.

**Goal:** Build reusable prompt files in `common-agents`, run them in VS Code, see where they stop working, and turn the most useful one into a skill that every repository can use.

**Repo:** `common-agents` · **Surface:** VS Code

> [!IMPORTANT]
> Prompt files work only in IDEs (VS Code, Visual Studio, JetBrains), and only in the workspace that contains them. The CLI, the Copilot app, and GitHub.com don't support them. VS Code has also deprecated prompt files for Agent Host sessions; they still work with the Local agent. This lab ends by moving a prompt file to a skill for that reason.

https://github.com/user-attachments/assets/dfaa9b9a-0bb1-4a71-830b-3ee242c882f8


Open `~/projects/common-agents` in VS Code before you start.

   ```bash
   cd ~/projects/common-agents && code .
   ```

   **Windows 11 Pro (Command Prompt):**

   ```cmd
   cd %USERPROFILE%\projects\common-agents && code .
   ```

### Exercise 2.1: Tour and fix the preloaded prompts

1. Type <command/ctrl+ p> and type `Run prompt files...`
1. Find the three preloaded prompts: `1-1-meeting-agenda`, `analyze-zendesk`, and `api-security-review`.
   <img width="603" height="146" alt="image" src="https://github.com/user-attachments/assets/2fef6035-50ac-4305-bea4-850358e3266e" />
1. Open `api-security-review.prompt.md`. It uses every common frontmatter field: `agent`, `tools`, `model`, and `argument-hint`. Check that the `model` value matches a model in your model picker. If it doesn't, the prompt uses the currently selected model.
1. Open `1-1-meeting-agenda.prompt.md`. It says `{{period}}`, which VS Code doesn't recognize as a variable. The model just sees the literal text. Fix it:

   ```markdown
   Create a meeting agenda for a 1-1 based on activity from ${input:period:date range, for example March 23 – April 1}. Look in Daily notes.
   ```

1. Run `/1-1-meeting-agenda` and confirm that VS Code asks you for the period.
1. `analyze-zendesk` says it isn't meant for Copilot Chat, yet it still shows up in the `/` menu. Discuss where reference-only prompts should live instead, for example a `docs/` folder.

The repository-wide `.github/copilot-instructions.md` is already preloaded. Read its **Authoring rules**; the review prompt in Exercise 2.2 checks changes against these existing standards. Don't replace the file.

1. Commit.

> [!IMPORTANT]
> ✅ **Checkpoint:** `/1-1-meeting-agenda` asks you for the period, and `common-agents` has the preloaded authoring rules.

### Exercise 2.2: Create a code review prompt

1. Create `.github/prompts/review-code.prompt.md`:

   ````markdown
   ---
   description: 'Review code changes against project standards'
   agent: 'agent'
   ---

   Review committed changes on this branch. DO NOT modify files.

   ## Find changes

   Use git to find changed files:

   ```bash
   git diff --name-only $(git merge-base main HEAD)...HEAD
   ```

   ```powershell
   git diff $(git merge-base main HEAD)...HEAD
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

 1.  Commit the new prompt on your original branch **before** creating the test branch:

   ```bash
   git add .github/prompts/review-code.prompt.md
   git commit -m "Add code review prompt"
   git branch --show-current
   git status --short
   ```

   Record the original branch name as `ORIGINAL-BRANCH` and ensure the working tree is clean. Use the same Git commands in Windows Command Prompt.
1. Create a branch:

   ```bash
   git checkout -b test-code
   ```

1. Commit a change that breaks one of the preloaded authoring rules from Exercise 2.1. For example, rename the `visualize` skill's directory to `visualissse` without changing its `name`:

   ```bash
   git mv .github/skills/visualize .github/skills/visualissse
   ```

1. Commit the deliberate bad rename:

   ```bash
   git commit -m "Test review with mismatched skill directory"
   ```

1. In Chat, type `/review-code` and press <kbd>Enter</kbd>.
1. Check the review, then return to your original branch and delete the test branch. Don't merge the broken rename:

   ```bash
   git status --short
   git switch ORIGINAL-BRANCH
   git branch -D test-code
   ```

   Proceed only if the working tree is clean; if the review unexpectedly edited files, inspect and save any work you want to keep before switching. Don't force the switch or discard unrelated work.

> [!IMPORTANT]
> ✅ **Checkpoint:** The review reports that the `name` and the directory don't match, and no files change. Back on your original branch, `review-code.prompt.md` remains, `visualize` has its original name, and `test-code` is deleted.

> [!TIP]
> Tool names vary between VS Code versions. If `terminal` isn't recognized, use the **Configure Tools** picker in the prompt file editor to insert valid tool names.

### 🚀 **Stretch goal:** Exercise 2.3: Create an issue summarizer with input variables

<details>
<summary>Show how to build an issue-summarizer prompt</summary>

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

> [!IMPORTANT]
> ✅ **Checkpoint:** You get one structured block per issue, each with all four fields.

</details>

### 🚀 **Stretch goal:** Exercise 2.4: Discuss the sort-order problem
<details>
 <summary>Look closely at the order of the issues in the output from Exercise 2.3.</summary>

1. Is the list really sorted by "needs attention"? Is a stale issue ranked above a hot one?
1. With your table, discuss why a prompt can't guarantee a deterministic sort. The model ranks the issues by reading them, not by calculating.
1. Sketch how a **skill** fixes this. A `scripts/rank_issues.py` script fetches the issues, computes a score from last-updated age and comment activity, and returns a sorted list. The model then only writes the summaries.

> [!IMPORTANT]
> ✅ **Checkpoint:** You can explain when a task needs a prompt file and when it needs a skill with a script. You'll build this skill in Exercise 3.2.

</details>

### 🚀 **Stretch goal:** Exercise 2.5: Turn the review prompt into a shared skill

<details>
<summary>Show how to share the review prompt as a skill</summary>

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

   **Windows 11 Pro (Command Prompt):**

   ```cmd
   cd %USERPROFILE%\projects\iphone-duo-viewer
   git switch -c test-review
   REM make and commit the change
   copilot
   ```

1. In the CLI, run:

   ```copilot
   /skills info code-review
   Use the /code-review skill on this branch.
   ```

> [!IMPORTANT]
> ✅ **Checkpoint:** `/skills info` shows the skill's location in `common-agents`, and the review flags the magic number using the *viewer's* `threejs.instructions.md`.

> [!TIP]
> VS Code has an experimental prompt-file migration that converts prompt files to skills. See [Migrate prompt files to skills](https://code.visualstudio.com/docs/agent-customization/overview#migrate-prompt-files-to-skills).

**Real-world pattern:** A platform team keeps `generate-unit-tests`, `review-code`, and `create-readme` in its central repository. These cover the three tasks new hires ask about most. Publishing them as skills, not prompt files, makes them work in every client the team uses.

</details>

---

[← Previous: Lab 1](lab-1.md) · [README ↑](../README.md#own-your-copilot-hands-on-lab-exercises) · [Next: Lab 3 →](lab-3.md)
