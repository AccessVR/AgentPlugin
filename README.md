# OrchestrateXR for AI assistants

Connect your AI assistant to [OrchestrateXR](https://orchestratexr.com) and teach it how the work is done.

This plugin brings two things:

- **The OrchestrateXR MCP server** at `https://app.orchestratexr.com/mcp`. It gives your assistant the tools to read and build your content. Every call runs as you, with your permissions.
- **Skills** that give the assistant the sequence for each job: which tools to run, in what order, and with which arguments. Without them, it works the steps out from tool names, one search at a time.

| Skill | Use it to |
|---|---|
| `orchestratexr` | Get oriented: what the records are, how to find tools, which organization to work in, and when to confirm |
| `design-training` | Work out what to build: what people need to do differently, why they don't yet, the objectives, and which content fits |
| `plan-a-project` | Plan in a Project: write its brief, add the people it answers to, save its sources, and track what it produces |
| `build-experience` | Create or extend a 360° Experience with scenes, text, questions, hotspots and media, then validate and publish it |
| `build-conversation` | Set up an AI Character, the Persona and Replica behind it, how it's scored, and the Thread around it |
| `add-assets` | Upload files, import them from a link, or generate images, narration and Replica videos |
| `knowledge-from-sources` | Turn documents and websites into knowledge a Persona or Character draws on |
| `assemble-course` | Put Experiences, Threads and Guides into a Course and publish it |
| `review-results` | Find how learners did: Sessions, Conversations, scores and Course progress |

Creating and changing content needs **Authoring Tools** on your account. OrchestrateXR support turns it on per user. Without it, your assistant can still look things up.

## Install

### Claude Code

```bash
claude plugin marketplace add AccessVR/AgentPlugin
claude plugin install orchestratexr@orchestratexr
```

Run `/mcp` and sign in to OrchestrateXR when your browser opens.

### Claude on the web and Claude Desktop

1. Go to **Customize › Plugins › Add › Add marketplace** and enter `https://github.com/AccessVR/AgentPlugin`.
2. Install **OrchestrateXR**.
3. Open the plugin's **Connectors** tab, connect OrchestrateXR, and sign in.

### Gemini CLI

```bash
gemini extensions install https://github.com/AccessVR/AgentPlugin
```

### Codex, Cursor, GitHub Copilot and others

The plugin carries manifests for Codex (`.codex-plugin/`) and Cursor (`.cursor-plugin/`). Add this repository as a plugin marketplace in either app.

For any other assistant that reads Agent Skills, copy the `skills/` folder into the place it looks, such as `.agents/skills/` or `.github/skills/`. Then add `https://app.orchestratexr.com/mcp` as an MCP server.

## Data

The plugin stores nothing. Your assistant sends its requests to your OrchestrateXR account through `app.orchestratexr.com`, signed in as you, and sees only what you can see in OrchestrateXR. You can disconnect an assistant at any time from **AI Assistant Connections** on your OrchestrateXR profile.

Read [Connecting Claude Code and Other AI Assistants](https://app.orchestratexr.com/guides/orchestratexr/connecting-ai-assistants) for the full guide.

## Contributing

This repository is published from OrchestrateXR's application repository, where the plugin is developed next to the server it describes, and it is overwritten on every release. Please don't open pull requests here. Write to support@orchestratexr.com instead, and we'll make the change at the source.

## License

MIT
