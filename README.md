# SpectralEdge by Cotes — MCP server

[![MCP Badge](https://lobehub.com/badge/mcp/cotes-spectraledge)](https://lobehub.com/mcp/cotes-spectraledge)

Read SpectralEdge's model signals for Kalshi's BTC, ETH, SOL and XRP 15-minute markets from Claude, ChatGPT, Cursor or any MCP client: the round trading now with every reading raised so far, and settled rounds with how each reading resolved.

- **Endpoint:** `https://mcp.cotes.ai/mcp` (Streamable HTTP, hosted — nothing to install)
- **Auth:** sign in with a SpectralEdge account (OAuth), or send an API key
- **Access:** an active SpectralEdge trial or subscription
- **Read-only:** no tool places orders or touches a Kalshi account

## Choose the asset

For BTC use `KXBTC15M`, ETH `KXETH15M`, SOL `KXSOL15M`, and XRP `KXXRP15M`.
`get_active_contest` and `list_contests` require `series`; there is no default, and a
call without one returns a validation error naming the field. The fixed
`spectraledge://contests/active` resource and the two prompts still default to BTC.
Keep different assets and model versions separate in accuracy, calibration, and P&L
reports. Use each asset's own dollar price scale; XRP prices can have fractions of a cent.

The MCP `list_series` tool returns the crypto catalog and latest
stored readings. A catalog entry does not promise a running producer. Empty signals
are not a neutral forecast; report that no readings were stored. `last_signal_at`
can be stale, and a null model tag can belong to an older unversioned reading.

## Features

- **Live round:** the contest trading now, its strike and window, and every model reading so far.
- **Settled history:** past rounds newest first, each reading graded `win`, `loss` or `void`, paged by time, optionally only the readings that fired.
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

### `list_series`

Returns the crypto catalog, active tickers, and the model tag and timestamp of the latest stored reading for each series. Takes no arguments.

### `get_active_contest`

The round trading now, with every reading raised so far. Readings' `outcome` is `pending` until the round settles.

| Parameter | Default | |
|---|---|---|
| `series` | required | Series prefix, not a full ticker: `KXBTC15M`, `KXETH15M`, `KXSOL15M` or `KXXRP15M` |

Returns `404` in the moment between one round settling and the next opening; retry shortly. A series that isn't in the catalog returns the same `404` and never resolves: the `detail` says which case it is, so check the prefix against `list_series` instead of retrying.

> What is SpectralEdge reading on the current BTC 15-minute contest?

### `list_contests`

Settled rounds, newest first, every reading graded.

| Parameter | Default | |
|---|---|---|
| `series` | required | Series prefix |
| `limit` | `20` | 1–100 rounds with `fired_only`; at most 10 without it |
| `since` | — | ISO-8601; rounds whose `window.expires_at` is at or after it |
| `before` | — | ISO-8601; rounds whose `window.expires_at` is at or before it |
| `fired_only` | `false` | When `true`, each round lists only its readings with `fired: true` (SpectralEdge's own calls) |
| `cursor` | — | The `next_cursor` from the previous result, for the next older page. Pass the same `series`, `since` and `fired_only`, and not `before` |

A settled round lists every reading raised during it, one every few seconds, so without `fired_only` a page holds at most 10 rounds. A full page carries `next_cursor`; pass it back as `cursor` for the next older page, and stop when a result has none.

> Of the last 50 settled rounds, how often did the first reading at 80% confidence or higher win?

### `get_contest`

One round by full ticker, live or settled, with its readings. Take tickers from another tool's `ticker` field; invented or truncated tickers return `404`.

| Parameter | Default | |
|---|---|---|
| `ticker` | required | e.g. `KXBTC15M-26NOV061530-30` |

> Show me the readings for KXBTC15M-26NOV061530-30 and where the side changed.

### Sample output

Each tool returns the API's status and JSON body. A settled Bitcoin round from `list_contests` with `fired_only: true`, abridged to 3 of its 5 fired readings (every reading carries its contest's `ticker` too, omitted here). Without `fired_only` the same round lists every reading, one every few seconds:

```json
{
  "status_code": 200,
  "body": {
    "contests": [
      {
        "ticker": "KXBTC15M-26NOV190930-30",
        "series": "KXBTC15M",
        "asset": "BTC",
        "state": "settled",
        "window": { "opens_at": "2026-11-19T14:15:00Z", "expires_at": "2026-11-19T14:30:00Z" },
        "strike_price_dollars": 63893.17,
        "settled_side": "down",
        "settled_price_dollars": 63889.4,
        "signals": [
          { "raised_at": "2026-11-19T14:25:35.000007Z", "side": "down", "confidence": 92.55, "entry_price_cents": 85.0, "edge_cents": 7.55, "outcome": "win", "model_version": "60f4797d", "fired": true },
          { "raised_at": "2026-11-19T14:25:45.000938Z", "side": "down", "confidence": 93.96, "entry_price_cents": 86.0, "edge_cents": 7.96, "outcome": "win", "model_version": "60f4797d", "fired": true },
          { "raised_at": "2026-11-19T14:25:55.000007Z", "side": "down", "confidence": 93.63, "entry_price_cents": 87.0, "edge_cents": 6.63, "outcome": "win", "model_version": "60f4797d", "fired": true }
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

- Every reading the model raised is returned, including ones that did not fire. `fired` is `true` on readings that cleared SpectralEdge's confidence, edge and distance-from-strike thresholds in force when they were raised; those are its calls. To judge readings by your own confidence threshold instead, apply it and say which threshold you used.
- `side` is `up` or `down`. The contract settles YES on up and NO on down.
- `confidence` is the model's probability for `side`, 0–100.
- `entry_price_cents` is the venue's quote on the same 0–100 scale.
- `edge_cents` is `confidence` minus `entry_price_cents`, in probability points, not money.
- `outcome` is `pending` until the round settles, then `win`, `loss` or `void`.
- `fired` is whether SpectralEdge showed the reading as a signal.

This is research data, not trade instructions or financial advice.

## Errors and limits

| Status | Where | Meaning |
|---|---|---|
| `401` | HTTP, on `/mcp` | No key and no sign-in. OAuth clients start the sign-in from this. |
| `401` | Tool result | A key or sign-in that is unknown or revoked. |
| `402` | Tool result | The account has no active trial or subscription, or its trial needs a card. The body links to [app.cotes.ai](https://app.cotes.ai). |
| `404` | Tool result | No live round right now (`get_active_contest`), a series that isn't in the catalog (it never resolves; check `list_series`), or an unknown ticker. |
| — | Tool error | `get_active_contest` or `list_contests` called without `series`: a validation error naming the field. |
| `429` | HTTP, on `/mcp` | Over the rate limit; wait and retry. |

API errors come back as the tool's output (`status_code` and `body`), not as MCP protocol errors. A missing argument is reported as a tool error with a validation message instead.

Requests are limited per IP to 1 per second on average, with bursts up to 20.

## Links

- API docs: [spectralbot.cotes.ai/docs](https://spectralbot.cotes.ai/docs/)
- App and account: [app.cotes.ai](https://app.cotes.ai)
- Examples and Claude Code skills: [gitlab.com/cotes-ai-public/spectraledge-examples](https://gitlab.com/cotes-ai-public/spectraledge-examples)

This repository holds the server's marketplace listing; the server itself is hosted by Cotes.

## License

MIT — see [LICENSE](LICENSE).

## Resources

The fixed `spectraledge://contests/active` resource stays on BTC. Use `spectraledge://contests/active/{series}` for another asset. Recorded series outside the crypto catalog, such as KXGOLD15M, remain queryable through contest tools and resources.
