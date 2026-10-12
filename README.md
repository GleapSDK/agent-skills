# Gleap Agent Skills

![Gleap Agent Skills](https://pub-01880b8170424478bf33a7d7174fa371.r2.dev/agent-skills.png)

Agent skills that help AI coding agents set up [Gleap](https://www.gleap.ai) SDKs and migrate from Intercom. Use them to integrate customer support, live chat and in-app feedback. Built on the open [Agent Skills](https://agentskills.io) format.

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

Install the **Gleap** plugin from the [Cursor Marketplace](https://cursor.com/marketplace). It bundles all skills below plus the hosted Gleap MCP server (`https://mcp.gleap.io/mcp`), so Cursor can also work with your Gleap tickets, contacts and help center. On first connect, Cursor opens the Gleap sign-in in your browser; pick the project you want to connect. No API key needed.

Use `/` in the chat to select and run a skill directly, or just describe what you need — Cursor's agent will pick the right skill automatically.

### Codex

- **Explicit invocation:** Include the skill directly in your prompt. In CLI/IDE, run `/skills` or type `$` to mention a skill.
- **Implicit invocation:** Codex automatically picks the right skill when your task matches the skill description.

No configuration needed. The agent reads the skill when your prompt matches.

### Other agents

These skills are compatible with most AI coding agents that support the Agent Skills format. See [agentskills.io](https://agentskills.io/) for a full list of supported agents and more details.

## Skills

### gleap

Work with your Gleap workspace through the [Gleap MCP server](https://www.gleap.ai/mcp): search and triage tickets, read and answer customer conversations, check feature request demand, manage contacts and help center articles, and pull support statistics. Requires the Gleap MCP server (included in the Cursor plugin; see the [setup guide](https://www.gleap.ai/mcp) for other agents).

**Example prompts:**
- "Summarize the open Gleap tickets tagged billing from this week"
- "Which feature requests have the most upvotes right now?"
- "Draft a reply to ticket #4821 and stage it in the composer"
- "What was our median first response time last month?"

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

## MCP server and network

The plugin's `mcp.json` connects to `https://mcp.gleap.io/mcp` (Streamable HTTP). US-hosted projects can point it at `https://mcp.us.gleap.ai/mcp` instead. Authentication is OAuth 2.1 (authorization code with PKCE and dynamic client registration) against `https://dashapi.gleap.io` (US: `https://dashapi.us.gleap.ai`); access is limited to the one Gleap project you approve. The plugin stores no API keys and requests no filesystem or shell access.

The `gleap-sdk-setup` skill's `scripts/get-latest-versions.sh` reads the latest SDK versions from npm, Maven Central, pub.dev and GitHub. It sends no data.

## License

MIT, see [LICENSE](LICENSE).
