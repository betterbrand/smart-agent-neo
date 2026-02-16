# Smart Agent Neo

**Smart Agent Neo** is a personal AI assistant platform with Morpheus decentralized inference as a first-class provider. It runs on your own devices and connects to the channels you already use: WhatsApp, Telegram, Slack, Discord, Google Chat, Signal, iMessage, Microsoft Teams, Matrix, and WebChat.

The core differentiator: decentralized AI inference via the [Morpheus network](https://mor.org), alongside centralized providers like Anthropic, OpenAI, Venice, and Ollama. Run open-source models (Kimi K2.5, GLM 4.7, Qwen3 235B, Llama 3.3 70B) through a P2P inference network with no single point of failure.

## Morpheus Integration

Morpheus is supported in two modes:

| Mode | How it works | Setup |
|------|-------------|-------|
| **Gateway** (recommended) | API gateway at `api.mor.org`, OpenAI-compatible | Set `MORPHEUS_API_KEY` |
| **Local proxy-router** (advanced) | Fully decentralized, on-chain session management via MOR staking | Set `MORPHEUS_ROUTER_URL` + wallet |

Available models on the Morpheus network:

| Model | Context | Reasoning |
|-------|---------|-----------|
| Kimi K2.5 | 131K | No |
| Kimi K2.5 Web | 131K | No |
| Kimi K2 Thinking | 131K | Yes |
| GLM 4.7 Flash | 202K | No |
| GLM 4.7 | 202K | Yes |
| Qwen3 235B | 131K | Yes |
| Llama 3.3 70B | 131K | No |
| GPT OSS 120B | 131K | No |

Models are dynamically discovered from the blockchain when using a local proxy-router. The static catalog serves as a fallback.

Full setup guide: [docs/providers/morpheus.md](docs/providers/morpheus.md)

## Install

Runtime: **Node >= 22**

```bash
npm install -g smart-agent-neo@latest

smart-agent-neo onboard --install-daemon
```

Works with npm, pnpm, or bun. The onboarding wizard walks you through gateway setup, workspace configuration, channel pairing, and provider auth.

## From Source

```bash
git clone https://github.com/betterbrand/smart-agent-neo.git
cd smart-agent-neo

pnpm install
pnpm ui:build
pnpm build

pnpm smart-agent-neo onboard --install-daemon

# Dev loop (auto-reload on TS changes)
pnpm gateway:watch
```

## Quick Start

```bash
# Start the gateway
smart-agent-neo gateway --port 18789 --verbose

# Talk to the assistant
smart-agent-neo agent --message "Ship checklist" --thinking high

# Send a message to a channel
smart-agent-neo message send --to +1234567890 --message "Hello from Smart Agent Neo"
```

## Providers

Smart Agent Neo supports multiple inference providers. Configure via environment variables or the `smart-agent-neo onboard` wizard.

| Provider | Auth | Notes |
|----------|------|-------|
| **Morpheus** | `MORPHEUS_API_KEY` | Decentralized inference via MOR network |
| Anthropic | `ANTHROPIC_API_KEY` or OAuth | Claude models |
| OpenAI | `OPENAI_API_KEY` or OAuth | GPT models |
| Venice | `VENICE_API_KEY` | Privacy-focused inference |
| Ollama | Local | Self-hosted models |
| Google | `GEMINI_API_KEY` | Gemini models |
| Amazon Bedrock | AWS SDK chain | AWS-hosted models |
| xAI | `XAI_API_KEY` | Grok models |
| Groq | `GROQ_API_KEY` | Fast inference |
| Together | `TOGETHER_API_KEY` | Open-source models |
| OpenRouter | `OPENROUTER_API_KEY` | Model aggregator |

Providers are auto-detected from environment variables and auth profiles. Multiple providers can be active simultaneously with automatic failover.

## Architecture

```
WhatsApp / Telegram / Slack / Discord / Google Chat / Signal / iMessage / Teams / Matrix / WebChat
               |
               v
+-------------------------------+
|           Gateway             |
|       (control plane)         |
|     ws://127.0.0.1:18789      |
+---------------+---------------+
                |
                +-- Pi agent (RPC)
                +-- CLI (smart-agent-neo ...)
                +-- WebChat UI
                +-- macOS app
                +-- iOS / Android nodes
                |
                v
+-------------------------------+
|       Inference Providers     |
|  Morpheus | Anthropic | OpenAI|
|  Venice | Ollama | Bedrock    |
+-------------------------------+
```

The Gateway is a local-first WebSocket control plane that manages sessions, channels, tools, and events. Inference requests are routed to whichever provider is configured, with automatic failover when multiple providers are available.

## Key Features

- **Multi-channel inbox** -- WhatsApp, Telegram, Slack, Discord, Google Chat, Signal, iMessage (via BlueBubbles), Microsoft Teams, Matrix, WebChat
- **Multi-agent routing** -- Route channels/accounts to isolated agents with separate workspaces and sessions
- **Voice Wake + Talk Mode** -- Always-on speech for macOS/iOS/Android with ElevenLabs
- **Live Canvas** -- Agent-driven visual workspace with A2UI
- **Browser control** -- Managed Chrome/Chromium with CDP control
- **Skills platform** -- Bundled, managed, and workspace skills with install gating
- **Companion apps** -- macOS menu bar app, iOS node, Android node
- **Sandbox mode** -- Docker-based sandboxing for non-main sessions

## Configuration

Minimal `~/.smart-agent-neo/smart-agent-neo.json`:

```json5
{
  agent: {
    model: "morpheus/kimi-k2.5",
  },
}
```

Or use a centralized provider:

```json5
{
  agent: {
    model: "anthropic/claude-opus-4-6",
  },
}
```

## Security

- **Default:** Tools run on the host for the main session (full access when it's just you).
- **Group/channel safety:** Set `agents.defaults.sandbox.mode: "non-main"` to run non-main sessions inside Docker sandboxes.
- **DM pairing:** Unknown senders receive a pairing code; approve with `smart-agent-neo pairing approve <channel> <code>`.

Run `smart-agent-neo doctor` to surface risky or misconfigured policies.

## Chat Commands

Send these in any connected channel:

| Command | Description |
|---------|-------------|
| `/status` | Session status (model + tokens + cost) |
| `/new` or `/reset` | Reset the session |
| `/compact` | Compact session context |
| `/think <level>` | Set thinking level (off/minimal/low/medium/high/xhigh) |
| `/verbose on\|off` | Toggle verbose output |
| `/usage off\|tokens\|full` | Per-response usage footer |
| `/restart` | Restart the gateway |
| `/activation mention\|always` | Group activation toggle |

## Remote Gateway

The gateway runs well on a small Linux instance. Clients (macOS app, CLI, WebChat) connect over Tailscale Serve/Funnel or SSH tunnels. Device nodes (macOS/iOS/Android) handle local actions like camera, screen recording, and notifications via `node.invoke`.

## Development Channels

| Channel | Description | npm tag |
|---------|-------------|---------|
| **stable** | Tagged releases (`vYYYY.M.D`) | `latest` |
| **beta** | Prereleases (`vYYYY.M.D-beta.N`) | `beta` |
| **dev** | Head of `main` | `dev` |

Switch with: `smart-agent-neo update --channel stable|beta|dev`

## License

[MIT](LICENSE)
