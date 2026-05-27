# BirdBets Base MCP Plugin

Base MCP custom plugin for BirdBets prediction markets.

This repository contains only the plugin spec. The plugin builds BirdBets calldata directly from the market and MYKCLAWD contracts so it can work even when Base MCP cannot fetch custom BirdBets HTTP endpoints.

The BirdBets app also hosts optional read and prepare endpoints:

```text
GET https://birdbets.mykclawd.xyz/api/markets/snapshot?market=Tomorrow
GET https://birdbets.mykclawd.xyz/api/base-mcp/prepare/bet?from=<wallet>&side=<YES|NO>&stake=<decimalMYKCLAWD>
```

The plugin should prefer direct contract calldata construction. The prepare endpoint can be used when available, but Base MCP may not be allowlisted to fetch it.

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
4. Paste it into Claude and ask it to add the markdown file to your Base MCP plugin configuration so BirdBets is always available:

```text
Add this BirdBets markdown file to my Base MCP plugin so it is always available in future chats. Use it as a Base MCP custom plugin, follow its onboarding gate, and use Base MCP send_calls for transactions.
```

5. After Claude confirms the plugin is available, ask something like:

```text
Using the BirdBets plugin, show me tomorrow's market odds and prepare a 10 MYKCLAWD YES bet.
```

Claude should complete the Base MCP onboarding gate first by calling `get_wallets`, then use the plugin's contract constants and ABI instructions to build `approve` and `betYes`/`betNo` calldata, submit the ordered calls through Base MCP `send_calls`, and ask you to approve in Base Account.

## Allowlist Caveat

Base's custom plugin docs note that custom plugin HTTP hosts may not be allowlisted for Base MCP `web_request`. This plugin is designed to keep working in that case by building calldata directly from the smart contract ABI. The prepare endpoint is a convenience path, not a requirement.
