<p align="center">
  <img src="https://img.shields.io/npm/v/node-red-contrib-mcp?style=flat-square&color=7E57C2" alt="npm version" />
  <img src="https://img.shields.io/npm/dm/node-red-contrib-mcp?style=flat-square&color=7E57C2" alt="downloads" />
  <img src="https://img.shields.io/badge/Node--RED-3.0+-red?style=flat-square" alt="Node-RED" />
  <img src="https://img.shields.io/badge/license-Apache--2.0-blue?style=flat-square" alt="License" />
  <img src="https://img.shields.io/github/stars/BavarianAnalyst/node-red-contrib-mcp?style=flat-square" alt="Stars" />
</p>

# node-red-contrib-mcp

### Connect AI agents to any MCP server — directly from Node-RED.

[MCP](https://modelcontextprotocol.io) (Model Context Protocol) is the open standard by Anthropic for connecting AI models to external tools and data. **This package brings MCP to Node-RED** — the world's most popular low-code platform for industrial automation, IoT, and event-driven workflows.

> Build agentic AI visually. Connect any LLM to any tool. No code required.

---

## Why?

Node-RED has **4M+ installations** across factories, energy, building automation, and IoT. MCP has **10,000+ servers** for databases, APIs, file systems, and more. Until now, there was no bridge.

**node-red-contrib-mcp** connects both worlds:

```
┌─────────────┐      ┌─────────────────┐      ┌─────────────┐
│             │      │                 │      │             │
│  Node-RED   │─────▶│  MCP Protocol   │─────▶│  MCP Server │
│  (your flow)│      │  (this package) │      │  (any tool) │
│             │◀─────│                 │◀─────│             │
└─────────────┘      └─────────────────┘      └─────────────┘
       │                                             │
       │              ┌─────────────┐                │
       └─────────────▶│   LLM API   │◀──────────────┘
                      │ (any model) │
                      └─────────────┘
```

---

## Nodes

| Node | Color | Description |
|------|-------|-------------|
| **mcp server** | config | Connect to any MCP server (Streamable HTTP or SSE transport) |
| **llm config** | config | Configure any OpenAI-compatible LLM (OpenAI, Anthropic, Ollama, vLLM, LiteLLM, ...) |
| **mcp tool** | purple | Call a specific MCP tool with arguments |
| **mcp tools** | purple | Discover all available tools on an MCP server |
| **mcp resource** | purple | Read a resource from an MCP server |
| **llm call** | orange | Send a prompt to an LLM — supports system prompt, JSON mode, multi-turn |
| **ai agent** | blue | **Autonomous agent loop** — LLM reasons over MCP tools until it has an answer |

---

## Install

```bash
cd ~/.node-red
npm install node-red-contrib-mcp
```

Or search for **`node-red-contrib-mcp`** in the Node-RED Palette Manager (Menu → Manage palette → Install).

---

## Quick Start

### 1. Call an MCP tool

```
[inject] → [mcp tool] → [debug]
```

Configure the **mcp tool** node with your MCP server URL and tool name. Pass arguments as `msg.payload` (object).

### 2. AI Agent — the magic node

```
[inject "What is the OEE of machine CNC-001?"] → [ai agent] → [debug]
```

The **ai agent** node is a full agentic loop:

1. Discovers available tools from your MCP server
2. Sends the user question + tool definitions to the LLM
3. LLM decides which tools to call → executes them via MCP
4. Feeds results back → LLM reasons → calls more tools or responds
5. Outputs the final answer + a log of all tool calls

**This is the same pattern as Claude, ChatGPT, or Cursor — but visual, in Node-RED.**

### 3. Multi-step pipeline

```
[inject] → [mcp tool "get_production_data"] → [llm call "Analyze this"] → [mcp tool "create_report"] → [debug]
```

Chain MCP tools and LLM calls for deterministic, auditable workflows.

---

## Configuration

### MCP Server (config node)

| Field | Description | Example |
|-------|-------------|---------|
| **URL** | MCP server endpoint | `http://localhost:3001/mcp` |
| **Transport** | Protocol variant | `Streamable HTTP` (default) or `SSE` |
| **API Key** | Optional Bearer token | `sk-...` |

### LLM Provider (config node)

| Field | Description | Example |
|-------|-------------|---------|
| **Base URL** | OpenAI-compatible endpoint | `https://api.openai.com/v1` |
| **Model** | Model identifier | `gpt-4o` |
| **API Key** | Your API key | `sk-...` |

**Works with:** OpenAI, Anthropic (via LiteLLM/proxy), Ollama (`http://localhost:11434/v1`), vLLM, Azure OpenAI, Google Gemini (via proxy), any OpenAI-compatible API.

---

## Use Cases

### Manufacturing & IIoT
- Connect AI agents to **OPC-UA**, **MQTT**, and factory MCP servers
- OEE monitoring, capacity planning, predictive maintenance
- Quality management with AI-driven root cause analysis

### Building Automation
- AI agents that query BACnet/Modbus data via MCP
- Smart energy management with LLM reasoning

### IT Automation
- Database agents (query PostgreSQL, MongoDB via MCP)
- DevOps: AI monitors logs, triggers actions
- Document processing pipelines

### Prototyping & Education
- Fastest way to prototype agentic AI workflows
- Visual debugging — see every tool call in Node-RED debug panel

---

## Example: OpenShopFloor (91 free MCP tools)

[OpenShopFloor](https://github.com/BavarianAnalyst/openshopfloor) is an open-source factory simulation with **91 MCP tools** for manufacturing AI. Connect these nodes to a running factory — for free.

| MCP Server | Port | Tools |
|-----------|------|-------|
| ERP | `:8021` | Orders, BOMs, customers, materials |
| Production / OEE | `:8024` | Machine status, OEE, capacity planning |
| Quality (QMS) | `:8023` | Inspections, defects, SPC |
| Warehouse (WMS) | `:8022` | Stock levels, reservations, movements |

```
MCP Server URL: http://your-server:8021/mcp
```

---

## How the AI Agent works

```
User: "Why did OEE drop on machine 9014 last week?"
                    │
                    ▼
            ┌──────────────┐
            │   AI Agent   │
            │              │
            │  1. List tools from MCP server
            │  2. LLM picks: get_oee(machine=9014, period=last_week)
            │  3. Execute via MCP → get result
            │  4. LLM picks: get_downtime_events(machine=9014)
            │  5. Execute via MCP → get result
            │  6. LLM synthesizes final answer
            │              │
            └──────┬───────┘
                   │
                   ▼
  "OEE dropped from 85% to 62% due to 3 unplanned
   stops: bearing failure (47min), tool change delay
   (23min), and material shortage (18min)."
```

---

## msg Properties

### mcp tool call

| Input | Type | Description |
|-------|------|-------------|
| `msg.payload` | object | Tool arguments |
| `msg.topic` | string | Tool name (if not in config) |

| Output | Type | Description |
|--------|------|-------------|
| `msg.payload` | any | Tool result (parsed) |
| `msg.mcpResult` | object | Raw MCP response |

### ai agent

| Input | Type | Description |
|-------|------|-------------|
| `msg.payload` | string | User question / task |

| Output | Type | Description |
|--------|------|-------------|
| `msg.payload` | string | Agent's final answer |
| `msg.agentLog` | array | Tool call log `[{tool, args, result}]` |
| `msg.iterations` | number | LLM calls made |

---

## Requirements

- Node-RED >= 3.0.0
- Node.js >= 18.0.0
- An MCP server to connect to
- An LLM API key (for `llm-call` and `ai-agent` nodes)

## Contributing

Issues and PRs welcome! [github.com/BavarianAnalyst/node-red-contrib-mcp](https://github.com/BavarianAnalyst/node-red-contrib-mcp)

## License

[Apache-2.0](LICENSE) — use it anywhere, commercially or not.

---

<p align="center">
  Built by <a href="https://github.com/BavarianAnalyst/openshopfloor">OpenShopFloor</a> — the open-source AI platform for factory operations
  <br><br>
  <a href="https://openshopfloor.zeroguess.ai">Live Demo</a> · <a href="https://github.com/BavarianAnalyst/openshopfloor">GitHub</a> · <a href="https://flows.nodered.org/node/node-red-contrib-mcp">Node-RED Library</a>
</p>
