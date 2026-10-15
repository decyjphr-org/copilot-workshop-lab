## 🚀 Lab 8: Keystone: iPhone Duo knolling viewer

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

### Exercise 8.2: Build the viewer (35 min)

1. In the Copilot app, start a new session for `iphone-duo-viewer` in **Plan** mode, and give it this prompt:

   ```text
   Turn this scaffold into an app to view the iPhone Duo. Do not stop until you have a
   good 3D model of the iPhone Duo.

   Use three.js and render to an HTML canvas. Add a Knolling toggle that lays out every
   component in a non-overlapping, aligned grid with a straight-on view. When knolling is
   turned off, return smoothly to the exploded view. Add a setting to control the transition.

   The iPhone Duo was announced at Apple Park on September 9, 2026, alongside the iPhone 18 Pro and iPhone 18 Pro Max, and is scheduled for release on October 23, 2026. Here's where to find each thing you listed. Wikipedia

   Dimensions and specs: Apple's official Tech Specs page (apple.com/iphone-duo/specs) is the authoritative source. It includes labeled dimension diagrams for both the open and closed states. The key numbers:

   - Open: 6.48 inches (164.6 mm) wide, 4.64 inches (117.8 mm) tall, and 0.21 inch (5.2 mm) deep. Apple
   - Closed: 3.31 inches (84.1 mm) wide, the same 117.8 mm tall, and 0.44 inch (11.3 mm) deep, weighing 8.96 ounces (254 grams). Apple
   - Displays: a 7.6-inch inner folding OLED at 1878×2670 (430 ppi) and a 5.4-inch outer OLED at 1398×2034 (460 ppi). Apple

   The same page also covers the chip, cameras, battery, the modem, and a diagram of the buttons and connectors.

   The announcement: Apple Newsroom's "Apple unveils iPhone Duo" press release is the primary source. For summaries, Tom's Guide and iDrop News both have detailed write-ups, and Wikipedia's iPhone Duo article gives background and history.

   References:

   https://www.apple.com/iphone-duo/specs/

   https://en.wikipedia.org/wiki/IPhone_Duo

   https://www.phonerepairguru.com/news/g331e94rfbdqe55zzrh4qzj9w36xyl

   https://wccftech.com/behold-the-apple-iphone-duo-apples-first-2000-juggernaut-replete-with-a-7-6-inch-inner-screen-a-hinge-made-up-of-100-individual-components-and-a-dual-battery-architecture/

   https://www.appleheadlines.com/iphone-duo/

   Pictures of sample app

   see attached
   ```
### Sample app

Attach these screenshots of a finished sample app:

<img width="419" height="387" alt="image-20260921090605126" src="https://github.com/user-attachments/assets/cad553fe-5f60-452f-8b4c-7af082121731" />

<img width="678" height="626" alt="image-20260921090535726" src="https://github.com/user-attachments/assets/d299fcd1-c142-4d8f-82cc-051bbd3982ce" />

1. Provide the sample pictures from the sample app design
1. Review the plan. Push back on anything that ignores the repository instructions. Then approve it.
1. Use the **Run** script to watch progress in the integrated browser. Iterate until the app meets the acceptance criteria. If no Run script is configured, use [Exercise 1.6](lab-1.md#-troubleshooting-exercise-16-if-the-app-doesnt-open-in-a-browser-configure-it-to-run-the-viewer) or start `npm run dev` in a terminal and open its local URL.

> [!TIP]
> If you completed optional Lab 6, your guard protects `src/specs.ts` from edits. If the agent needs to add new spec values, temporarily remove `src/specs\.ts` from the guard pattern, review the diff yourself, and then put it back.

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


### References

* [Apple iPhone Duo Tech Specs](https://www.apple.com/iphone-duo/specs/)
* [iPhone Duo on Wikipedia](https://en.wikipedia.org/wiki/IPhone_Duo)
* [Phone Repair Guru: iPhone Duo](https://www.phonerepairguru.com/news/g331e94rfbdqe55zzrh4qzj9w36xyl)
* [Wccftech: Behold the Apple iPhone Duo](https://wccftech.com/behold-the-apple-iphone-duo-apples-first-2000-juggernaut-replete-with-a-7-6-inch-inner-screen-a-hinge-made-up-of-100-individual-components-and-a-dual-battery-architecture/)
* [Apple Headlines: iPhone Duo](https://www.appleheadlines.com/iphone-duo/)

### 🚀 **Stretch goal:** Exercise 8.1: Check your setup (10 min)

<details>
<summary>Show how to verify the workshop setup from the CLI</summary>

Confirm the core setup and any optional customizations you chose to complete. Skip rows for optional exercises you did not run; plugins, personal agents, and hooks are not required to build the viewer. Confirm it in a CLI session in the viewer repo:

| From | What | Check |
|------|------|-------|
| Lab 1 | Repository, three.js, and Playwright instructions | `copilot instruction` |
| Lab 1 | App **Run** script | Run it in the Copilot app |
| Labs 2–3, 7 | `code-review`, `release-validator`, `issue-triage` (shared or plugin skills) | `/skills list` |
| Lab 0 | `visualize` and `html-in-canvas` (preloaded) | `/skills list` |
| Lab 3 | Web Design Guidelines skill | `/skills list` |
| Labs 4, 7 | `code-explainer` and optional `readme-specialist` (shared or plugin agents), coordinator, `my-review-style` | `/agent` |
| Lab 5 | Context7 and built-in Playwright | `/mcp` |
| Lab 6 | Guardrail and format hooks (optional) | `/env` |

> [!IMPORTANT]
> ✅ **Checkpoint:** Every applicable row checks out; skipped optional rows are recorded.

</details>

### 🚀 **Stretch goal:** Exercise 8.3: Review and ship (15 min)

<details>
<summary>Show how to review the changes and prepare a pull request</summary>

1. If installed through the optional labs, run `/code-review` (shared or plugin skill) and your `my-review-style` agent on the session's changes. Otherwise, ask Copilot to review the changes against the repository instructions without editing files.
1. Ask `code-explainer` (shared or plugin agent) to explain the finished architecture, and use the `visualize` skill to produce a data-flow diagram of the exploded-to-knolling transition.
1. If installed, ask `readme-specialist` (shared or plugin agent) to update the README with setup steps, the diagram, and screenshots. Otherwise, ask Copilot directly to update the README. Use Playwright to capture the screenshots.
1. If you skipped optional Lab 6, first publish the viewer using [Exercise 6.1](lab-6.md#exercise-61-publish-the-viewer-repo-to-github); hooks are not required for publishing. In the app, run `/pr-open` to open a pull request, then request a Copilot code review on it.

> [!IMPORTANT]
> ✅ **Checkpoint:** You demo the exploded view, the knolling toggle, and the transition setting to your table.

</details>

### 🚀 **Stretch goals**

<details>
<summary>Show ways to extend the knolling viewer</summary>

* Add an `agentStop` hook that returns `{"decision": "block", "reason": "..."}` until an automated check passes (for example, a Playwright script that confirms no two knolled parts overlap). The CLI stops forcing more turns after 8 consecutive blocks.
* Add an open and closed fold animation using the hinge.
* Use the preloaded `html-in-canvas` skill to render live HTML labels (part name and dimensions) next to each knolled part inside the three.js scene. HTML-in-Canvas is an experimental Chromium API behind a flag, so keep a fallback path.
* Write a `knolling-layout` skill in `common-agents/.github/skills/`, graduate it into `workshop-kit`, bump the version, push, and run `copilot plugin update --all`.

</details>

---

[← Previous: Lab 7](lab-7.md) · [README ↑](../README.md#own-your-copilot-hands-on-lab-exercises) · [Next: README →](../README.md)
