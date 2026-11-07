# SpectralEdge by Cotes — MCP server

[![MCP Badge](https://lobehub.com/badge/mcp/cotes-spectraledge)](https://lobehub.com/mcp/cotes-spectraledge)

Read SpectralEdge's model signals for Kalshi's Bitcoin 15-minute markets (series `KXBTC15M`) from Claude, ChatGPT, Cursor or any MCP client: the round trading now with every reading raised so far, and settled rounds with how each reading resolved.

- **Endpoint:** `https://mcp.cotes.ai/mcp` (Streamable HTTP, hosted — nothing to install)
- **Auth:** sign in with a SpectralEdge account (OAuth), or send an API key
- **Access:** an active SpectralEdge trial or subscription
- **Read-only:** no tool places orders or touches a Kalshi account

## Features

- **Live round:** the contest trading now, its strike and window, and every model reading so far.
- **Settled history:** past rounds newest first, each reading graded `win`, `loss` or `void`, paged by time.
- **One round by ticker:** any contest, live or settled, with its readings.
- **Prompts:** a live-round summary, and a hit-rate count at a confidence threshold you choose.

## Connect

### Claude (claude.ai and Claude Desktop)

Settings → Connectors → **Add custom connector**, URL `https://mcp.cotes.ai/mcp`, no key. Claude opens a SpectralEdge sign-in: enter your email, then the six-digit code, and approve.

### ChatGPT

Add `https://mcp.cotes.ai/mcp` as a custom app with OAuth, then sign in the same way.

### Claude Code

```bash
claude mcp add --transport http spectraledge https://mcp.cotes.ai/mcp
```

Then run `/mcp` in Claude Code to sign in. To use an API key instead:

```bash
claude mcp add --transport http spectraledge https://mcp.cotes.ai/mcp --header "Authorization: Bearer cotes_sk_..."
```

### Cursor

`~/.cursor/mcp.json`:

```json
{
  "mcpServers": {
    "spectraledge": {
      "url": "https://mcp.cotes.ai/mcp",
      "headers": { "Authorization": "Bearer cotes_sk_..." }
    }
  }
}
```

### VS Code

`.vscode/mcp.json`:

```json
{
  "servers": {
    "spectraledge": {
      "type": "http",
      "url": "https://mcp.cotes.ai/mcp",
      "headers": { "Authorization": "Bearer cotes_sk_..." }
    }
  }
}
```

### Any other client

Use `https://mcp.cotes.ai/mcp`. Clients that support MCP OAuth find the sign-in from the server's `401` response. Otherwise create an API key under **Account → API keys** at [app.cotes.ai](https://app.cotes.ai) and send it in either header:

```
Authorization: Bearer cotes_sk_...
X-API-Token: cotes_sk_...
```

Signed-in apps are listed under **Account → API keys → Connected apps**, where you can disconnect them. Reconnecting the same app replaces its previous connection.

## Tools

### `get_active_contest`

The round trading now, with every reading raised so far. Readings' `outcome` is `pending` until the round settles.

| Parameter | Default | |
|---|---|---|
| `series` | `KXBTC15M` | Series prefix, not a full ticker |

Returns `404` in the moment between one round settling and the next opening; retry shortly.

> What is SpectralEdge reading on the current BTC 15-minute contest?

### `list_contests`

Settled rounds, newest first, every reading graded.

| Parameter | Default | |
|---|---|---|
| `series` | `KXBTC15M` | Series prefix |
| `limit` | `20` | 1–100 rounds |
| `since` | — | ISO-8601; rounds whose `window.expires_at` is at or after it |
| `before` | — | ISO-8601, inclusive on `window.expires_at`. To page back, pass the oldest `expires_at` you received and drop the repeated row |

> Of the last 50 settled rounds, how often did the first reading at 80% confidence or higher win?

### `get_contest`

One round by full ticker, live or settled, with its readings. Take tickers from another tool's `ticker` field; invented or truncated tickers return `404`.

| Parameter | Default | |
|---|---|---|
| `ticker` | required | e.g. `KXBTC15M-26NOV061530-30` |

> Show me the readings for KXBTC15M-26NOV061530-30 and where the side changed.

### Sample output

Each tool returns the API's status and JSON body. A settled round, with 3 of its 168 readings (the model reads every few seconds):

```json
{
  "status_code": 200,
  "body": {
    "contests": [
      {
        "ticker": "KXBTC15M-26NOV061530-30",
        "state": "settled",
        "window": { "opens_at": "2026-11-06T20:15:00Z", "expires_at": "2026-11-06T20:30:00Z" },
        "strike_price_dollars": 75745.57,
        "settled_side": "down",
        "settled_price_dollars": 75694.95,
        "signals": [
          { "raised_at": "2026-11-06T20:16:00.001768Z", "side": "down", "confidence": 61.82, "entry_price_cents": 58.0, "edge_cents": 3.82, "outcome": "win", "model_version": "34fa82b3" },
          { "raised_at": "2026-11-06T20:23:00.000008Z", "side": "down", "confidence": 87.41, "entry_price_cents": 80.0, "edge_cents": 7.41, "outcome": "win", "model_version": "34fa82b3" },
          { "raised_at": "2026-11-06T20:29:55.001313Z", "side": "down", "confidence": 99.5, "entry_price_cents": 100.0, "edge_cents": -0.5, "outcome": "win", "model_version": "34fa82b3" }
        ]
      }
    ]
  }
}
```

## Prompts

### `monitor_live_signals`

Summarises the live round: ticker, window, and each reading as YES/NO with confidence, entry price and edge. Argument: `series` (default `KXBTC15M`).

> Use monitor_live_signals.

### `measure_accuracy`

Counts how the last N settled rounds resolved, taking the first reading per round at or above your threshold, and reports the count, wins, losses and hit rate. Arguments: `confidence_threshold` (required), `series` (default `KXBTC15M`), `limit` (default `50`).

> Use measure_accuracy with confidence_threshold 75 and limit 100.

## Reading a signal

- Every reading the model raised is returned, including ones below the threshold SpectralEdge acts on. The API does not publish that threshold or mark which readings fired, so apply your own confidence threshold before treating a reading as a call.
- `side` is `up` or `down`. The contract settles YES on up and NO on down.
- `confidence` is the model's probability for `side`, 0–100.
- `entry_price_cents` is the venue's quote on the same 0–100 scale.
- `edge_cents` is `confidence` minus `entry_price_cents`, in probability points, not money.
- `outcome` is `pending` until the round settles, then `win`, `loss` or `void`.

This is research data, not trade instructions or financial advice.

## Errors and limits

| Status | Where | Meaning |
|---|---|---|
| `401` | HTTP, on `/mcp` | No key and no sign-in. OAuth clients start the sign-in from this. |
| `401` | Tool result | A key or sign-in that is unknown or revoked. |
| `402` | Tool result | The account has no active trial or subscription, or its trial needs a card. The body links to [app.cotes.ai](https://app.cotes.ai). |
| `404` | Tool result | No live round right now (`get_active_contest`), or an unknown ticker. |
| `429` | HTTP, on `/mcp` | Over the rate limit; wait and retry. |

Tool-result errors come back as the tool's output (`status_code` and `body`), not as MCP protocol errors.

Requests are limited per IP to 1 per second on average, with bursts up to 20.

## Links

- API docs: [spectralbot.cotes.ai/docs](https://spectralbot.cotes.ai/docs/)
- App and account: [app.cotes.ai](https://app.cotes.ai)
- Examples and Claude Code skills: [gitlab.com/cotes-ai-public/spectraledge-examples](https://gitlab.com/cotes-ai-public/spectraledge-examples)

This repository holds the server's marketplace listing; the server itself is hosted by Cotes.

## License

MIT — see [LICENSE](LICENSE).
