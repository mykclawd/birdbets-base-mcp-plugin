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
