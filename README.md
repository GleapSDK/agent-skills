# Gleap Agent Skills

Agent skills that help AI coding agents integrate and work with [Gleap](https://gleap.io). Built on the open [Agent Skills](https://agentskills.io) format.

## Getting started

Run the install command in your project directory:

```bash
npx skills add GleapSDK/agent-skills
```

This adds the skills to a `.skills/` folder in your project.

### Claude Code

Skills are picked up automatically. Just start Claude Code in your project and prompt away:

```
claude
> Add Gleap to my project
```

You can also use `/` commands to select and run a skill directly:

```
> /gleap-sdk-setup
> /migrate-from-intercom
```

### Cursor

Use `/` in the chat to select and run a skill directly, or just describe what you need — Cursor's agent will pick the right skill automatically.

### Codex

- **Explicit invocation:** Include the skill directly in your prompt. In CLI/IDE, run `/skills` or type `$` to mention a skill.
- **Implicit invocation:** Codex automatically picks the right skill when your task matches the skill description.

No configuration needed. The agent reads the skill when your prompt matches.

## Skills

### gleap-sdk-setup

Set up the Gleap SDK in any project. The agent detects your platform, installs the correct package, initializes the SDK, configures permissions, and sets up common API calls like user identification and event tracking.

**Supported platforms:** JavaScript (Angular, React, Vue, Next.js, Nuxt, and more), iOS, Android, React Native, Flutter, Ionic/Capacitor, Cordova, FlutterFlow

**Example prompts:**
- "Add Gleap to my project"
- "Set up the Gleap feedback SDK"
- "Integrate Gleap into my React Native app"

### migrate-from-intercom

Switch from Intercom to Gleap. The agent finds all Intercom references in your codebase, removes the Intercom SDK, installs Gleap, and replaces every API call with its Gleap equivalent. Also handles server-side REST API migration.

**Supported platforms:** JavaScript, iOS, Android, React Native, Flutter, Server-side REST API (Node.js, PHP, Ruby, Go, Java, .NET)

**Example prompts:**
- "Migrate from Intercom to Gleap"
- "Replace Intercom with Gleap"
- "Switch our feedback tool from Intercom to Gleap"

## How it works

Each skill is a directory with a `SKILL.md` file containing instructions and metadata. Agents load only skill names and descriptions at startup, then read the full instructions when a matching task comes up — keeping context efficient.

Learn more about the [Agent Skills format](https://agentskills.io/specification).
