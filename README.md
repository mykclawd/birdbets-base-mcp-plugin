# BirdBets Base MCP Plugin

Base MCP custom plugin for BirdBets prediction markets.

This repository contains only the plugin spec. The plugin is contract-only: it reads BirdBets state from Base contracts, builds calldata directly from the market and MYKCLAWD ABIs, and submits through Base MCP `send_calls`.

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

Claude should complete the Base MCP onboarding gate first by calling `get_wallets`, then use only Base MCP on-chain reads and the plugin's contract constants/ABI instructions to build `approve` and `betYes`/`betNo` calldata. It should submit the ordered calls through Base MCP `send_calls` and ask you to approve in Base Account.

The plugin intentionally does not depend on BirdBets HTTP APIs or custom host allowlisting.
