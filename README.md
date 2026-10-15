---
layout: default
title: "Own your Copilot: Hands-on lab exercises"
permalink: /workshop/
---

# Own your Copilot: Hands-on lab exercises

**Start here:** Follow the [participant setup guide](setup.md) before beginning Lab 0.
**Public setup site:** <https://decyjphr-org.github.io/copilot-workshop-lab/>

These exercises go with the *Own your Copilot* workshop sessions. You work in two repositories the whole time:

* **`common-agents`** is your team's central repository of prompt files, skills, and agents. You build it out in Labs 2–4. In Lab 7 you turn it into a plugin marketplace.
* **`iphone-duo-viewer`** is a product repository you create locally in Lab 1. It consumes what's in `common-agents`, adds its own MCP servers and hooks, and becomes the keystone app in Lab 8.

This split matches how most organizations work. A platform team curates shared customizations once, and product teams use them everywhere.

## How the pieces fit together

<img width="1650" height="847" alt="image" src="https://github.com/user-attachments/assets/7a993eba-8d2e-43a8-83b9-e2b55e40d67b" />

| Lab | Topic | Repo | Time | Primary surface | Plan needed |
|-----|-------|------|------|-----------------|-------------|
| [0](labs/lab-0.md) | Prerequisites and setup | `common-agents` | - | VS Code, CLI, Copilot app | Any paid plan |
| [1](labs/lab-1.md) | Custom instructions | `iphone-duo-viewer` | - | Copilot app, GitHub.com | Any paid plan (org exercise: Business or Enterprise) |
| [2](labs/lab-2.md) | Prompt files | `common-agents` | - | VS Code | Any paid plan |
| [3](labs/lab-3.md) | Agent skills | `common-agents` | - | CLI, Copilot app | Any paid plan |
| [4](labs/lab-4.md) | Custom agents and orchestration | `common-agents` | - | CLI, Copilot app, GitHub.com | Org exercise: Enterprise |
| [5](labs/lab-5.md) | MCP servers | `iphone-duo-viewer` | - | CLI, Copilot app | Any paid plan (org MCP policy must allow it) |
| [6](labs/lab-6.md) | Hooks | `iphone-duo-viewer` | - | CLI, cloud agent | Any paid plan |
| [7](labs/lab-7.md) | Plugins and marketplace | `common-agents` | - | CLI, Copilot app | Any paid plan |
| [8](labs/lab-8.md) | Keystone: iPhone Duo knolling viewer | `iphone-duo-viewer` | - | Copilot app | Any paid plan |

**Conventions used in this guide**

* Placeholders are in `CAPS`. Replace them with your own values:
  * `YOUR-ORG` is an organization or user account where you can push.
  * `YOUR-USER` is your local macOS, Linux, or Windows user name (for absolute paths).
  * `YOUR-HANDLE` is your GitHub username.
* Both repositories are assumed to live in `~/projects/`. Adjust the paths if you use a different folder.
* Commands are shown for macOS/Linux (`bash`) first. Where the Windows 11 Pro command differs, a **Windows 11 Pro** variant using the Command Prompt (`cmd.exe`) follows immediately after.
* ✅ **Checkpoint** marks how you confirm that a step worked. Don't skip these.
* 🧯 **Troubleshooting** lists the most common problems for that lab.
* 🚀 **Stretch goal** items are optional. Try them if you finish early.
* Commit after each lab. In the `common-agents` labs, push as well, because Lab 7 publishes from GitHub.

---

## Further reading

* [Customization cheat sheet](https://docs.github.com/en/copilot/reference/customization-cheat-sheet)
* [Hooks reference](https://docs.github.com/en/copilot/reference/hooks-reference)
* [Creating a plugin for Copilot CLI](https://docs.github.com/en/copilot/how-tos/copilot-cli/customize-copilot/plugins-creating)
* [Adding MCP servers for Copilot CLI](https://docs.github.com/en/copilot/how-tos/copilot-cli/customize-copilot/add-mcp-servers)
* [Adding agent skills for Copilot CLI](https://docs.github.com/en/copilot/how-tos/copilot-cli/customize-copilot/add-skills)
* [Using your own LLM models in Copilot CLI](https://docs.github.com/en/copilot/how-tos/copilot-cli/customize-copilot/use-byok-models)
