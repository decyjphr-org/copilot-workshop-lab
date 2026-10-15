## Lab 6: Hooks

<details>
<summary>🚀 Stretch goal: show the optional hooks lab</summary>

**Goal:** Enforce guardrails in the viewer repo with a `preToolUse` hook and format files automatically with a `postToolUse` hook. Hooks are deterministic: they run every time, whatever the model decides.

**Repo:** `iphone-duo-viewer` · **Surface:** CLI, cloud agent

**Requirements:** `jq` installed locally.

### Exercise 6.1: Publish the viewer repo to GitHub

The cloud agent (Exercise 6.5) and the pull request in Lab 8 need the repo on GitHub:

```bash
cd ~/projects/iphone-duo-viewer
gh repo create YOUR-ORG/iphone-duo-viewer --private --source=. --remote=origin --push
```

**Windows 11 Pro (Command Prompt):**

```cmd
cd %USERPROFILE%\projects\iphone-duo-viewer
gh repo create YOUR-ORG/iphone-duo-viewer --private --source=. --remote=origin --push
```

> [!IMPORTANT]
> ✅ **Checkpoint:** `git remote -v` shows `origin` pointing to `YOUR-ORG/iphone-duo-viewer`.

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
         "matcher": "bash|edit|create|apply_patch",
         "bash": "./.github/hooks/guard.sh",
         "timeoutSec": 10
       }],
       "postToolUse": [{
         "type": "command",
         "matcher": "edit|create|apply_patch",
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
     edit|create|apply_patch)
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

   > [!NOTE]
   > **Windows 11 Pro:** The hook scripts above are Bash scripts, and `guardrails.json` invokes them with the `"bash"` key, so they need a Bash interpreter on `PATH`. Use **Git Bash** (installed with Git for Windows) or **WSL**, both of which already have `chmod`, so this command works unchanged from a Git Bash or WSL prompt. There's no separate Windows `chmod` equivalent needed.

1. Test the guard locally before you use it with Copilot:

   ```bash
   echo '{"toolName":"bash","toolArgs":"{\"command\":\"rm -rf dist\"}"}' | ./.github/hooks/guard.sh
   # Expected: {"permissionDecision":"deny","permissionDecisionReason":"Blocked by guardrails: destructive shell command"}

   echo '{"toolName":"bash","toolArgs":"{\"command\":\"ls\"}"}' | ./.github/hooks/guard.sh
   # Expected: no output
   ```

   **Windows 11 Pro (Git Bash or WSL prompt):**

   ```bash
   echo '{"toolName":"bash","toolArgs":"{\"command\":\"rm -rf dist\"}"}' | ./.github/hooks/guard.sh
   # Expected: {"permissionDecision":"deny","permissionDecisionReason":"Blocked by guardrails: destructive shell command"}

   echo '{"toolName":"bash","toolArgs":"{\"command\":\"ls\"}"}' | ./.github/hooks/guard.sh
   # Expected: no output
   ```

   > [!NOTE]
   > Run these from Git Bash or WSL, not from PowerShell or `cmd.exe` — `guard.sh` and `format.sh` are Bash scripts and won't run directly under Windows' native shells.

1. Commit and push.

> [!TIP]
> Argument field names can differ between tools and CLI versions. To see the real payload, add `echo "$INPUT" >> /tmp/hook-payload.log` near the top of a script, then run one tool call.

### Exercise 6.4: Watch the hooks work in the CLI

1. Start `copilot` in the viewer repo.
1. Ask: `Clean up the build output with rm -rf dist`.
1. Ask: `Change the closed depth in src/specs.ts to 12 mm`.
1. Ask: `Add a comment to the top of src/main.ts`.

> [!IMPORTANT]
> ✅ **Checkpoint:** The first two requests are denied with your reasons. The third one edits the file, and the file is reformatted afterward.

> [!NOTE]
> The hooks documentation covers the Copilot CLI and the cloud agent. It doesn't say whether the Copilot app runs repository hooks, so test your guardrails in the CLI.

### Exercise 6.5: Run the hooks with the Copilot cloud agent

1. Make sure `.github/hooks/` is on the default branch. The cloud agent loads hooks only from `.github/hooks/*.json` in the cloned repository.
1. Create an issue in `YOUR-ORG/iphone-duo-viewer`, for example "Remove the `dist` directory and make the closed depth 12 mm", and assign it to Copilot.
1. Open the session logs.

> [!IMPORTANT]
> ✅ **Checkpoint:** The logs show the denied tool calls and your denial reasons.

### 🧯 **Troubleshooting (Lab 6)**

<details>
<summary>Show how to fix hook configuration and output issues</summary>

* **Hooks don't run.** Check that the file is in `.github/hooks/`, that it's valid JSON (`jq . .github/hooks/guardrails.json`), that it has `"version": 1`, and that the scripts are executable and have a shebang.
* **The guard didn't block anything.** Timeouts **fail open**: if the hook takes longer than `timeoutSec`, the tool call continues. Keep the guard well under 10 seconds.
* **Output is ignored.** Emit exactly one JSON object on stdout. Two `echo` calls with JSON produce invalid output, which is ignored.

> [!IMPORTANT]
> A crash or non-zero exit in a command `preToolUse` hook denies the tool call (fail-closed). A timeout doesn't (fail-open). Design your guard with both behaviors in mind.

</details>

### 🚀 **Stretch goal:** Add a session or agent hook

<details>
<summary>Show how to add a sessionStart log or agentStop sound</summary>

Add a `sessionStart` hook that logs the start time to `logs/session.log`, or an `agentStop` hook that plays a sound when the agent finishes.

</details>

</details>

---

[← Previous: Lab 5](lab-5.md) · [README ↑](../README.md#own-your-copilot-hands-on-lab-exercises) · [Next: Lab 7 →](lab-7.md)
