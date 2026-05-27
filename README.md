# BirdBets Base MCP Plugin

Base MCP custom plugin for BirdBets prediction markets.

This repository contains only the plugin spec. The BirdBets app hosts the read and prepare endpoints used by the plugin:

```text
GET https://birdbets.mykclawd.xyz/api/markets/snapshot?market=Tomorrow
GET https://birdbets.mykclawd.xyz/api/base-mcp/prepare/bet?from=<wallet>&side=<YES|NO>&stake=<decimalMYKCLAWD>
```

The prepare endpoint returns unsigned calldata as an ordered `transactions[]` batch. Base MCP maps that batch into `send_calls`, and the user approves the transaction in Base Account.

## Plugin

- [`plugins/birdbets-market.md`](plugins/birdbets-market.md)

## Requirements

- Base MCP connected to the assistant.
- A Base Account wallet.
- MYKCLAWD on Base for betting, or enough liquid Base assets to swap into MYKCLAWD.

## Use In Claude Web

1. Add Base MCP to Claude web as a custom integration.
2. Open [`plugins/birdbets-market.md`](plugins/birdbets-market.md).
3. Copy the full plugin spec.
4. Paste it into Claude and say:

```text
Use this Base MCP custom plugin for BirdBets. Follow its onboarding gate and use Base MCP send_calls for transactions.
```

Then ask something like:

```text
Using the BirdBets plugin, show me tomorrow's market odds and prepare a 10 MYKCLAWD YES bet.
```

Claude should complete the Base MCP onboarding gate first by calling `get_wallets`, then use BirdBets read endpoints, call the prepare endpoint, map `transactions[]` into Base MCP `send_calls`, and ask you to approve in Base Account.

## Allowlist Caveat

Base's custom plugin docs note that custom plugin HTTP hosts may not be allowlisted for Base MCP `web_request`. If Claude cannot fetch the BirdBets prepare endpoint directly, open the prepare URL yourself, paste the JSON response into Claude, and it can still map `transactions[]` into `send_calls`.
