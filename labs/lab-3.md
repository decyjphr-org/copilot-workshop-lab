## Lab 3: Agent skills

**Goal:** Study a well-built skill that bundles instructions, a script, reference material, and a template. Then build your own with `/create-skill`, and install community skills.

**Repo:** `common-agents` · **Surface:** VS Code or CLI or Copilot app



https://github.com/user-attachments/assets/4c7877b1-bfa5-40c3-937e-fdf42ce8ab13


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

   **Windows 11 Pro (Command Prompt):**

   ```cmd
   cd %USERPROFILE%\projects\common-agents
   copilot
   ```

   ```copilot
   Use the /release-validator skill to check release v1.0.0. Testing and security checks passed, docs are done, and Product, Marketing, and Revenue approved.
   ```

1. Go to VS Code
1. Open the generated report, tick the Security box and fill in an approver, then ask Copilot to validate again.

> [!IMPORTANT]
> ✅ **Checkpoint:** The first run stops with `validate.py` failures for the Security approval. The second run prints `PASS`.

**Discussion:** Which parts of this skill would you trust to the model, and which to a script? Why is "every box is ticked" a script's job?

### Exercise 3.2: Build an `issue-triage` skill with `/create-skill`

**Stay in VS Code**
Now fix the sort-order problem discussed in optional [Exercise 2.4](lab-2.md#-stretch-goal-exercise-24-discuss-the-sort-order-problem) with a skill whose script does the ranking. You can do this without that stretch goal: a prompt alone cannot guarantee a deterministic attention order; the script computes the score and the model summarizes its sorted results.

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

1. Review the generated files against the preloaded authoring rules from [Exercise 2.1](lab-2.md#exercise-21-tour-and-fix-the-preloaded-prompts):

   * `name: issue-triage` matches the directory.
   * The `description` says when to use the skill.
   * `SKILL.md` links to the script, the reference file, and the template.
   * `rank_issues.py` runs by itself: `python3 .github/skills/issue-triage/scripts/rank_issues.py @microsoft/vscode`.

1. Run `/skills reload`, then:

   ```copilot
   Use the /issue-triage skill on @actions/starter-workflows
   ```

1. Commit and push.
1. Start a session in `~/projects/iphone-duo-viewer` and run `/skills info issue-triage`.

> [!IMPORTANT]
> ✅ **Checkpoint:** The summaries appear in the same order as the script's output, and the skill also loads from the viewer repo through `skillDirectories`.

### 🧯 **Troubleshooting (Lab 3)**

<details>
<summary>Show how to fix skill loading, authoring, and installation issues</summary>

* **The skill doesn't load.** The `name` must be 1–64 characters of lowercase letters, numbers, and hyphens, and must match the directory name exactly.
* **Copilot doesn't pick the skill automatically.** At first, only `name` and `description` load. Make the description specific about when to use the skill.
* **`SKILL.md` is getting long.** Keep it under 500 lines and move detail into `references/`. The detail loads only when the task needs it.
* **A skill has `tools` in its frontmatter.** The preloaded `root-instructions` skill does. The GitHub Docs list `name`, `description`, `license`, and `allowed-tools` as the `SKILL.md` fields. Use `allowed-tools` to pre-approve tools, and don't rely on the others.

</details>

### Exercise 3.3: Install community skills

1. Preview each skill and review its source and scripts, then use **Customize → Skills** in the Copilot app. If the installer asks where to put the skill, choose `~/projects/common-agents/.github/skills` so it lives in your team repo.
   1. Install **Grill me**
   1. Install **Web Design Guidelines**. You'll use Web Design Guidelines for the viewer's controls in Lab 8.
   1. Check whether the **impeccable design** skill is already installed or enable it

   ```copilot
   /skills list
   ```

1. Try them out:

   * `Grill me on my plan to roll out common-agents to 40 repositories.`
   * From the viewer repo: `Review the Knolling toggle against the web design guidelines.`
   * `Use impeccable design to suggest improvements to the viewer's controls.`

> [!IMPORTANT]
> ✅ **Checkpoint:** `/skills list` shows all three skills, and each one changes how Copilot responds.

> [!WARNING]
> Always preview third-party skills before you install them. Don't pre-approve `shell` or `bash` in `allowed-tools` for a skill unless you've reviewed its scripts and trust the source.

### 🚀 **Stretch goal:** Try the preloaded `js-to-typescript` skill

<details>
<summary>Show how to convert a small JavaScript project</summary>

Use the preloaded `js-to-typescript` skill (from `.claude/skills`) on a small JavaScript project, and check with `/skills info js-to-typescript` which location it loaded from.

</details>

### 🚀 **Stretch goal:** Search, preview, and install with GitHub CLI

<details>
<summary>Show the GitHub CLI skill commands</summary>

Use GitHub CLI as an alternative to the app installer. Preview the search result and replace `SKILL-NAME` with its identifier before installing; avoid installing duplicate copies. Choose the team repository location if prompted.

```bash
gh skill search grill
gh skill search web-design
gh skill preview SKILL-NAME
gh skill install SKILL-NAME
```

Always review third-party scripts and permissions before installation.

</details>

---

[← Previous: Lab 2](lab-2.md) · [README ↑](../README.md#own-your-copilot-hands-on-lab-exercises) · [Next: Lab 4 →](lab-4.md)
