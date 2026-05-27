# BirdBets Market Plugin

> [!IMPORTANT]
>
> ## STOP — COMPLETE BASE MCP ONBOARDING BEFORE USING THIS PLUGIN
>
> Before building or submitting any BirdBets transaction, you MUST complete the Base MCP onboarding flow:
>
> 1. Call `get_wallets` to detect the user's Base Account wallet.
> 2. Present wallet status and the normal Base MCP transaction disclaimer.
> 3. Use the detected wallet address for all balance, allowance, and position reads.

> [!CAUTION]
> ## SOLIDITY-ONLY PLUGIN
>
> This plugin uses only Base MCP on-chain reads and Base MCP `send_calls`. Build calldata from the Solidity ABI and contract constants below.

BirdBets is a prediction market on Base where users bet MYKCLAWD on whether tomorrow's BirdBuddy visit count will be greater than the market threshold. This plugin builds BirdBets contract calldata directly and executes the ordered batch through Base MCP `send_calls`.

**Supported chain:** Base mainnet (`8453`, Base MCP chain name `base`).

**Writes:** Every prepared transaction must be executed through Base MCP `send_calls` and approved by the user in Base Account.

---

## Contract Constants

Use these contract addresses unless the user explicitly provides updated BirdBets deployment details:

```text
Prediction market: 0x79435444379154c49c7cbEd68a371A815EADfa5b
MYKCLAWD token:    0xE3C5FCfBfea42D5CE2492FD82c239B5503f17ba3
Chain:             base
Chain ID:          8453
Token decimals:    18
```

Market IDs use `YYYYMMDD` in the BirdBets timezone. For example, May 28, 2026 is `20260528`. Compute it from the date, or ask the user to confirm the market ID.

---

## Verified Function Selectors

Use these exact selectors. Do not recompute them, substitute alternatives, or guess if a read reverts.

```text
balanceOf(address)         = 0x70a08231
allowance(address,address) = 0xdd62ed3e
markets(uint256)           = 0xb1283e77
oddsBps(uint256)           = 0xf2bc1574
approve(address,uint256)   = 0x095ea7b3
betYes(uint256,uint256)    = 0x408a2f6a
betNo(uint256,uint256)     = 0x22a18d42
```

For `markets(uint256)`, the first 4 bytes must be `0xb1283e77`. Selectors such as `0x4db7b4a9` and `0x9a6b3e47` are wrong for this contract.

---

## Direct Contract Reads

Before placing a bet, use Base MCP read-contract capability. If the exact tool name differs, use the available Base MCP tool for direct on-chain reads.

Read MYKCLAWD:

```json
[
  {
    "type": "function",
    "name": "balanceOf",
    "stateMutability": "view",
    "inputs": [{ "name": "account", "type": "address" }],
    "outputs": [{ "name": "", "type": "uint256" }]
  },
  {
    "type": "function",
    "name": "allowance",
    "stateMutability": "view",
    "inputs": [
      { "name": "owner", "type": "address" },
      { "name": "spender", "type": "address" }
    ],
    "outputs": [{ "name": "", "type": "uint256" }]
  }
]
```

Read BirdBets market:

```json
[
  {
    "type": "function",
    "name": "markets",
    "stateMutability": "view",
    "inputs": [{ "name": "marketId", "type": "uint256" }],
    "outputs": [
      { "name": "exists", "type": "bool" },
      { "name": "resolved", "type": "bool" },
      { "name": "date", "type": "string" },
      { "name": "threshold", "type": "uint256" },
      { "name": "yesPool", "type": "uint256" },
      { "name": "noPool", "type": "uint256" },
      { "name": "createdAt", "type": "uint256" },
      { "name": "bettingClosesAt", "type": "uint256" },
      { "name": "resolvedAt", "type": "uint256" },
      { "name": "actualVisits", "type": "uint256" },
      { "name": "winningSide", "type": "uint8" }
    ]
  },
  {
    "type": "function",
    "name": "oddsBps",
    "stateMutability": "view",
    "inputs": [{ "name": "marketId", "type": "uint256" }],
    "outputs": [
      { "name": "yesBps", "type": "uint256" },
      { "name": "noBps", "type": "uint256" }
    ]
  }
]
```

Required checks:

- `markets(marketId).exists === true`
- `markets(marketId).resolved === false`
- Current time is before `bettingClosesAt`
- MYKCLAWD `balanceOf(wallet) >= amountWei`
- If `allowance(wallet, predictionMarket) < amountWei`, include an approval call before the bet call

If Base MCP cannot do reads, ask the user to confirm market details and token balance, then build calldata from the constants below.

---

## Build Calldata Directly

Convert the human stake to wei using 18 decimals:

```text
amountWei = stake * 10^18
```

Function selectors:

```text
approve(address,uint256) = 0x095ea7b3
betYes(uint256,uint256) = 0x408a2f6a
betNo(uint256,uint256) = 0x22a18d42
```

Use the verified selectors above for read calls too.

Encode calldata using standard Ethereum ABI encoding:

```text
approve calldata = selector approve + abi.encode(predictionMarket, amountWei)
YES calldata     = selector betYes + abi.encode(marketId, amountWei)
NO calldata      = selector betNo + abi.encode(marketId, amountWei)
```

If the assistant has an ABI encoding tool, use it. If not, construct the calldata by left-padding each `uint256` or address argument to 32 bytes.

Write ABI:

```json
[
  {
    "type": "function",
    "name": "approve",
    "stateMutability": "nonpayable",
    "inputs": [
      { "name": "spender", "type": "address" },
      { "name": "value", "type": "uint256" }
    ],
    "outputs": [{ "name": "", "type": "bool" }]
  },
  {
    "type": "function",
    "name": "betYes",
    "stateMutability": "nonpayable",
    "inputs": [
      { "name": "marketId", "type": "uint256" },
      { "name": "amount", "type": "uint256" }
    ],
    "outputs": []
  },
  {
    "type": "function",
    "name": "betNo",
    "stateMutability": "nonpayable",
    "inputs": [
      { "name": "marketId", "type": "uint256" },
      { "name": "amount", "type": "uint256" }
    ],
    "outputs": []
  }
]
```

## send_calls Mapping

Submit calls through Base MCP `send_calls`:

```json
{
  "chain": "base",
  "calls": [
    {
      "to": "<transactions[0].to>",
      "value": "<transactions[0].value>",
      "data": "<transactions[0].data>"
    },
    {
      "to": "<transactions[1].to>",
      "value": "<transactions[1].value>",
      "data": "<transactions[1].data>"
    }
  ]
}
```

Preserve order. `approve` must come before the bet call. If allowance is already sufficient, omit the approval call.

For a 500 MYKCLAWD NO bet on market `20260528`, the calls are:

```json
{
  "chain": "base",
  "calls": [
    {
      "to": "0xE3C5FCfBfea42D5CE2492FD82c239B5503f17ba3",
      "value": "0x0",
      "data": "0x095ea7b300000000000000000000000079435444379154c49c7cbed68a371a815eadfa5b00000000000000000000000000000000000000000000001b1ae4d6e2ef500000"
    },
    {
      "to": "0x79435444379154c49c7cbEd68a371A815EADfa5b",
      "value": "0x0",
      "data": "0x22a18d4200000000000000000000000000000000000000000000000000000000013526b000000000000000000000000000000000000000000000001b1ae4d6e2ef500000"
    }
  ]
}
```

---

## Orchestration Pattern

```text
1. Base MCP get_wallets -> choose the Base Account EVM address.
2. Determine tomorrow's market ID as YYYYMMDD in America/New_York, or ask the user to confirm it.
3. Read market state, MYKCLAWD balance, and allowance through Base MCP on-chain reads.
4. Explain threshold, odds from oddsBps, pools, close time, wallet balance, and whether approval is needed.
5. If MYKCLAWD balance is insufficient, use Base MCP swap to acquire MYKCLAWD.
6. Build approve calldata if allowance is insufficient.
7. Build betYes or betNo calldata from marketId and amountWei.
8. Submit ordered calls through send_calls(chain="base", calls=[...]).
9. Present Base Account approval URL to the user.
10. Poll get_request_status(requestId).
11. On confirmation, report the transaction hash, side, stake, market date, and threshold.
```

Do not attempt oracle or owner-only contract actions. Do not place a bet unless the user explicitly selected side and stake.

## If You Cannot Read Chain State

If Base MCP cannot perform the read calls, ask the user to confirm:

- Market ID
- Threshold
- Whether the market is open
- MYKCLAWD balance
- Current allowance

Then build calldata from the constants and ABI in this plugin and submit through `send_calls`.
