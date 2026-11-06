# SpectralEdge by Cotes — MCP server

Read SpectralEdge's model signals for Kalshi's Bitcoin 15-minute markets (series `KXBTC15M`) from Claude, ChatGPT or any MCP client: the round trading now with every reading raised so far, and settled rounds with how each signal resolved.

**Endpoint:** `https://mcp.cotes.ai/mcp` (Streamable HTTP)

## Connect

**Sign in (Claude, ChatGPT and other OAuth-capable clients).** Add `https://mcp.cotes.ai/mcp` as a custom connector with no key. The client opens a SpectralEdge sign-in: enter your email, then the six-digit code, and approve. Disconnect any time under Account → API keys → Connected apps at [app.cotes.ai](https://app.cotes.ai).

**API key (scripts, CLIs, clients without OAuth).** Create a key under Account → API keys at [app.cotes.ai](https://app.cotes.ai) and send it as either header:

```
Authorization: Bearer cotes_sk_...
X-API-Token: cotes_sk_...
```

Claude Code:

```bash
claude mcp add --transport http spectraledge https://mcp.cotes.ai/mcp --header "Authorization: Bearer cotes_sk_..."
```

Access needs an active SpectralEdge trial or subscription. Without one, tool calls return `402` with a link to subscribe.

## Tools

All read-only. None place orders or touch a Kalshi account.

| Tool | What it returns |
|---|---|
| `get_active_contest` | The BTC 15-minute round trading now, with every signal raised so far. |
| `list_contests` | Settled rounds, newest first, each signal graded win, loss or void. Paged with `since` / `before`, up to 100 per call. |
| `get_contest` | One round by full ticker, live or settled, with its signals. |

Prompts: `monitor_live_signals` (summarise the live round) and `measure_accuracy` (hit rate over recent settled rounds at a confidence threshold you choose).

## Reading a signal

Signals are every reading the model raised, including ones below the threshold SpectralEdge acts on, so apply your own confidence threshold before treating one as a call. `side` is `up` / `down` (the contract settles YES on up, NO on down). `confidence` and `entry_price_cents` are on the same 0–100 scale; `edge_cents` is confidence minus entry price. This is research data, not financial advice.

## Links

- API docs: [spectralbot.cotes.ai/docs](https://spectralbot.cotes.ai/docs/)
- App: [app.cotes.ai](https://app.cotes.ai)
- Examples and agent skills: [gitlab.com/cotes-ai-public/spectraledge-examples](https://gitlab.com/cotes-ai-public/spectraledge-examples)

The server's source is not in this repository; this repository holds its marketplace listing.
