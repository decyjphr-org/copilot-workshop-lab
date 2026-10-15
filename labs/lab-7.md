## Lab 7: Plugins

<details>
<summary>🚀 Stretch goal: show the optional plugins lab</summary>

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

> [!IMPORTANT]
> ✅ **Checkpoint:** `copilot plugin list` shows the plugin, and `/skills list` inside a session shows its skills.

### Exercise 7.2: Move skills and agents into a plugin

This optional lab needs the `code-review` skill from optional [Exercise 2.5](lab-2.md#-stretch-goal-exercise-25-turn-the-review-prompt-into-a-shared-skill) and the `readme-specialist` agent from optional [Exercise 4.2](lab-4.md#-stretch-goal-exercise-42-create-a-readme-specialist-team-agent). Complete those stretch goals before the move commands below. The coordinator is needed only if you want its optional workflow.

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

   **Windows 11 Pro (Command Prompt):**

   ```cmd
   cd %USERPROFILE%\projects\common-agents
   mkdir plugins\workshop-kit\skills plugins\workshop-kit\com.github.copilot\agents

   git mv .github/skills/release-validator plugins/workshop-kit/skills/
   git mv .github/skills/code-review       plugins/workshop-kit/skills/
   git mv .github/skills/issue-triage      plugins/workshop-kit/skills/
   git mv .github/agents/readme-specialist.agent.md plugins/workshop-kit/com.github.copilot/agents/
   git mv .github/agents/code-explainer.agent.md    plugins/workshop-kit/com.github.copilot/agents/
   ```

   > [!NOTE]
   > `git mv` accepts forward slashes on Windows too, so the paths above don't need to change. Windows `mkdir` creates intermediate directories by default, so no `-p` flag is needed.
   
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

   > [!NOTE]
   > You should've completed exercises 2.5 to see `issue-triage` skill and exercise 4.2 to see `readme-specialist` custom agent.

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

> [!IMPORTANT]
> ✅ **Checkpoint:** While the local plugin was installed, the three skills and both agents came from `YOUR-HANDLE-workshop-kit`.

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

> [!IMPORTANT]
> ✅ **Checkpoint:** In the viewer repo, `/skills info code-review` shows the plugin as its source, and your teammate's `copilot plugin list` shows `YOUR-HANDLE-workshop-kit`.

**Discussion:** `common-agents` now works in two ways. `.github/skills` and `.github/agents` are where you incubate new work, loaded for you through `skillDirectories` and links. `plugins/` holds reviewed, versioned releases that anyone installs from the marketplace. When an incubating skill is ready, move it into the plugin, bump `version`, and push. Subscribers pick it up with `copilot plugin update`.

### 🚀 **Stretch goals**

<details>
<summary>Show how to publish a second plugin or auto-install it for contributors</summary>

* **Publish a second plugin.** Package the four instruction-generator skills as `instructions-kit`: `root-instructions`, `area-instructions`, `nested-hub`, and `nested-detail`. Move them into `plugins/instructions-kit/skills/`, add a `plugin.json`, and add a second entry to the `plugins` array in `marketplace.json`. Subscribers can then install just the kit they need. Before you publish, check the skills against the Exercise 2.1 authoring rules. For example, `root-instructions` has a `tools` field and mentions VS Code-only tools.
* **Auto-install for contributors.** Add the marketplace and plugin to the viewer repo's `.github/copilot/settings.json` with `extraKnownMarketplaces` and `enabledPlugins`, so every contributor gets the kit automatically. See [Configuration file settings](https://docs.github.com/en/copilot/reference/copilot-cli-reference/cli-config-dir-reference#configuration-file-settings) for the exact format.

</details>

</details>

---

[← Previous: Lab 6](lab-6.md) · [README ↑](../README.md#own-your-copilot-hands-on-lab-exercises) · [Next: Lab 8 →](lab-8.md)
