# BirdBets Market Plugin

> [!IMPORTANT]
>
> ## STOP — COMPLETE BASE MCP ONBOARDING BEFORE USING THIS PLUGIN
>
> Before calling any BirdBets prepare endpoint, you MUST complete the Base MCP onboarding flow:
>
> 1. Call `get_wallets` to detect the user's Base Account wallet.
> 2. Present wallet status and the normal Base MCP transaction disclaimer.
> 3. Use the detected wallet address as the `from` query parameter for every prepare call.

BirdBets is a prediction market on Base where users bet MYKCLAWD on whether tomorrow's BirdBuddy visit count will be greater than the market threshold. This plugin fetches BirdBets market state, prepares unsigned approval and bet calldata, then executes the ordered batch through Base MCP `send_calls`.

**Supported chain:** Base mainnet (`8453`, Base MCP chain name `base`).

**Writes:** Every prepared transaction must be executed through Base MCP `send_calls` and approved by the user in Base Account.

---

## Read Endpoints

Fetch BirdBets integration context:

```text
GET https://birdbets.mykclawd.xyz/api/bankr/context
```

Fetch today's or tomorrow's market snapshot:

```text
GET https://birdbets.mykclawd.xyz/api/markets/snapshot?market=Today
GET https://birdbets.mykclawd.xyz/api/markets/snapshot?market=Tomorrow
```

Fetch a payout preview for a proposed bet:

```text
GET https://birdbets.mykclawd.xyz/api/markets/snapshot?market=Tomorrow&side=YES&stake=10
GET https://birdbets.mykclawd.xyz/api/markets/snapshot?market=Tomorrow&side=NO&stake=10
```

Fetch recent BirdBuddy visit stats:

```text
GET https://birdbets.mykclawd.xyz/api/birdbuddy/visits?days=7
```

Use read endpoints to explain threshold, market status, current odds, pools, payout preview, and close time before preparing a transaction.

---

## Prepare Endpoint

Prepare a YES or NO bet for tomorrow's market:

```text
GET https://birdbets.mykclawd.xyz/api/base-mcp/prepare/bet?from=<wallet>&side=<YES|NO>&stake=<decimalMYKCLAWD>
```

Query parameters:

- `from`: detected Base Account EVM wallet address.
- `side`: `YES` or `NO`.
- `stake`: human-readable MYKCLAWD amount, for example `10` or `2.5`.

The prepare endpoint validates:

- Tomorrow's market exists.
- The market is unresolved and open.
- The wallet has enough MYKCLAWD.
- Current allowance for the prediction market.

If allowance is insufficient, the response includes an `approve` transaction before the bet transaction.

Response:

```json
{
  "ok": true,
  "chain": "base",
  "chainId": 8453,
  "from": "0x...",
  "market": {
    "label": "Tomorrow",
    "marketId": "20260528",
    "marketDate": "2026-05-28",
    "threshold": "12",
    "bettingClosesAt": "1780000000",
    "bettingClosesAtIso": "2026-05-28T04:00:00.000Z"
  },
  "bet": {
    "side": "YES",
    "stake": "10000000000000000000",
    "stakeFormatted": "10",
    "token": "MYKCLAWD"
  },
  "wallet": {
    "balance": "50000000000000000000",
    "balanceFormatted": "50",
    "allowance": "0",
    "allowanceFormatted": "0",
    "requiresApproval": true
  },
  "contracts": {
    "predictionMarket": "0x79435444379154c49c7cbEd68a371A815EADfa5b",
    "bettingToken": "0xE3C5FCfBfea42D5CE2492FD82c239B5503f17ba3"
  },
  "transactions": [
    {
      "step": "approve",
      "to": "0xE3C5FCfBfea42D5CE2492FD82c239B5503f17ba3",
      "value": "0x0",
      "data": "0x...",
      "chainId": 8453,
      "description": "Approve 10 MYKCLAWD for BirdBets"
    },
    {
      "step": "bet",
      "to": "0x79435444379154c49c7cbEd68a371A815EADfa5b",
      "value": "0x0",
      "data": "0x...",
      "chainId": 8453,
      "description": "Bet YES with 10 MYKCLAWD on 2026-05-28"
    }
  ]
}
```

If the wallet lacks MYKCLAWD, the endpoint returns `ok: false` with an insufficient-balance message. Use Base MCP `swap` to acquire MYKCLAWD, then call the prepare endpoint again.

---

## send_calls Mapping

Map every `transactions[*]` entry into one Base MCP `send_calls` request:

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

Preserve transaction order. `approve` must come before `bet`. If the prepare endpoint returns only one transaction, pass only that call.

---

## Orchestration Pattern

```text
1. Base MCP get_wallets -> choose the Base Account EVM address.
2. Fetch /api/markets/snapshot?market=Tomorrow&side=<side>&stake=<stake>.
3. Explain market threshold, odds, payout preview, and risk.
4. Fetch /api/base-mcp/prepare/bet?from=<wallet>&side=<side>&stake=<stake>.
5. If insufficient MYKCLAWD, use Base MCP swap to acquire MYKCLAWD and repeat step 4.
6. Convert transactions[] into send_calls(chain="base", calls=[...]).
7. Present Base Account approval URL to the user.
8. Poll get_request_status(requestId).
9. On confirmation, report the transaction hash, side, stake, market date, and threshold.
```

Do not use BirdBets oracle or admin endpoints. Do not place a bet unless the user explicitly selected side and stake.
