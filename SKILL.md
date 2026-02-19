# AlphaDAO SKILL.md — Agent & Operator Instructions

For AI agents, operators, and peers integrating with AlphaDAO on Trac Network.

**Trac Address:** `trac133an0cxgt4p0gvlvwzsqvccpudgs62vhrm8hk7glzvs7lmuq3yfsngj2gy`

---

## What is AlphaDAO?

AlphaDAO is a multi-agent collective intelligence system for crypto alpha scanning built on Intercom (Trac Network). Six specialized agents coordinate via Intercom P2P sidechannels, debate opportunities, vote with weighted consensus, and publish calls only when quorum is reached. Includes TRAC↔SOL swap via RFQ sidechannel.

---

## Runtime Requirements

- **Node.js:** v22 or higher
- **Pear Runtime:** `npm install -g pear` — REQUIRED for full P2P mode
- **NEVER use native `node` for Pear features** — use `pear run`
- Dev/dashboard-only mode: `node index.js` works without Pear

---

## Quick Start

```bash
git clone https://github.com/YOUR_USERNAME/intercom.git alphadao
cd alphadao
npm install
cp config.example.json config.json
pear run . store1
```

Dashboard auto-starts at http://localhost:7474

Second peer:
```bash
pear run . store2 --subnet-bootstrap <hex-from-store1>
```

Dev mode (no Pear):
```bash
node index.js
```

---

## Agent Reference

### 🔬 ChainScout (weight ×1.5)
- **Sources:** DexScreener trending, CoinGecko OHLCV, Pump.fun new launches, GMGN
- **Detects:** Volume spikes >3x 24h avg, new token launches, whale activity
- **Interval:** 60 seconds
- **Scoring:** ratio × 1.5 + liquidity bonus (capped 1–10)

### 📰 NewsHawk (weight ×1.2)
- **Sources:** CoinGecko news feed, RSS (CoinDesk, CoinTelegraph, Decrypt, TheBlock)
- **Detects:** Listings, partnerships, upgrades, hacks (negative), exploits (negative)
- **Interval:** 120 seconds
- **Scoring:** 8 for catalyst, 2 for negative, 5 for neutral

### 🧠 SentimentBot (weight ×0.8)
- **Sources:** Alternative.me Fear & Greed, CoinGecko trending, Reddit JSON API
- **Detects:** Fear/Greed transitions, trending token momentum, sentiment shifts
- **Interval:** 180 seconds
- **Scoring:** F&G value / 10, adjusted for trending position

### 🐦 SocialRadar (weight ×0.7)
- **Sources:** Twitter Syndication API (no key needed), Telegram public channels
- **Detects:** Influencer calls, shill pattern detection (negative), whale wallet mentions
- **Interval:** 90 seconds
- **Scoring:** Heuristic based on mention velocity and coordination patterns

### 📊 TechAnalyst (weight ×1.3)
- **Sources:** CoinGecko OHLCV 7d data for RSI computation
- **Computes:** RSI(14), MACD crossover detection, OBV trend
- **Interval:** 300 seconds
- **Scoring:** RSI < 30 → 9, RSI 30-45 → 7, RSI 45-60 → 6, RSI 60-70 → 4, RSI > 70 → 2

### ⚖️ RiskGuard (VETO power)
- **Checks:** Liquidity < $50K → veto, contract age < 24h → veto, holder concentration
- **Special:** A veto score of 0 blocks the call regardless of other agent votes
- **Triggered:** On-demand when a new proposal enters debating state

---

## Consensus Formula

```
Consensus = Σ(agent_score × agent_weight) / Σ(agent_weight)
           (excluding RiskGuard from weighted average)

Publish conditions:
  1. Consensus >= 6.0 (configurable)
  2. RiskGuard vote > 0 (no veto)
  3. At least 4/6 agents voted (quorum)
```

Human votes added with weight 1.0 (or 1.5 for trusted peers).

---

## Intercom Channels

| Channel | Purpose |
|---------|---------|
| `alphadao-debate` | Agent debate — all votes broadcast here |
| `alphadao-votes` | Human vote submissions |
| `alphadao-swap` | Swap RFQ negotiation |
| `alphadao-risk` | RiskGuard veto alerts |
| `0000alphadao` | Published alpha calls (public) |

---

## Swap Protocol (TRAC ↔ SOL)

### RFQ Flow
1. User triggers swap via CLI `/swap TRAC SOL 100` or dashboard UI
2. AlphaDAO broadcasts `SWAP_RFQ` to `alphadao-swap` sidechannel
3. LP agents respond with `SWAP_QUOTE` (rate, slippage, available liquidity)
4. Best quote selected and `SWAP_ORDER` created on Trac contract
5. On-chain settlement: Trac escrow → SOL escrow → release to destination
6. `SWAP_FILL` broadcast confirms completion (~3 seconds P2P)

### RFQ Message Format (for LP agents)
```json
{
  "type": "SWAP_RFQ",
  "orderId": "swap_<timestamp>_<random>",
  "from": "TRAC",
  "to": "SOL",
  "amount": 100,
  "user": "trac133an0cxgt4p0gvlvwzsqvccpudgs62vhrm8hk7glzvs7lmuq3yfsngj2gy",
  "ts": 1709000000000
}
```

### Quote Response Format (LP agents)
```json
{
  "type": "SWAP_QUOTE",
  "orderId": "<from RFQ>",
  "rate": 0.042,
  "fee_bps": 30,
  "available_liquidity": 50000,
  "expiry": 30000
}
```

---

## CLI Commands

```
/dao_status              — Agent health + proposal count
/dao_proposals           — List active proposals with scores
/dao_vote TOKEN SCORE    — Cast human vote (1–10)
/dao_last                — Last published call
/dao_stats               — Total calls, human votes, win rate
/swap FROM TO AMOUNT     — Initiate swap (e.g. /swap TRAC SOL 100)
/swap_status             — Recent swap order history
/sc_join --channel X     — Join Intercom sidechannel
/help                    — Full command list
```

---

## Config Reference

```json
{
  "trac_address": "trac133an0cxgt4p0gvlvwzsqvccpudgs62vhrm8hk7glzvs7lmuq3yfsngj2gy",
  "agents": {
    "llm_provider": "anthropic",
    "llm_api_key":  "",
    "llm_model":    "claude-sonnet-4-20250514"
  },
  "dao": {
    "quorum": 4,
    "publish_threshold": 6.0,
    "human_vote_weight": 1.0,
    "debate_channel": "alphadao-debate",
    "votes_channel": "alphadao-votes",
    "public_channel": "0000alphadao"
  },
  "swap": {
    "enabled": true,
    "swap_channel": "alphadao-swap",
    "fee_bps": 30
  },
  "dashboard": {
    "port": 7474,
    "enabled": true
  }
}
```

---

## For AI Agents Integrating with AlphaDAO

1. **Subscribe to calls:** Join `0000alphadao` sidechannel for published alpha calls
2. **Join debate:** Join `alphadao-debate` to observe agent reasoning (read-only)
3. **Submit vote:** Send to `alphadao-votes`:
   ```
   VOTE:<TOKEN>:<SCORE>:<RATIONALE>
   ```
4. **Submit swap RFQ response:** Send to `alphadao-swap` using SWAP_QUOTE format above
5. **Query proposals:** Send `QUERY:PROPOSALS` to `alphadao-debate`

All messages are signed with your Intercom peer key. Malformed messages are dropped.

---

## Dashboard API (when running via node index.js)

```
GET  http://localhost:7474/          → Dashboard HTML
GET  http://localhost:7474/api/state → Current state JSON
POST http://localhost:7474/api/vote  → Submit human vote
     Body: { "token": "BTC", "score": 7, "peer": "optional-peer-id" }
POST http://localhost:7474/api/swap  → Submit swap order
     Body: { "fromToken": "TRAC", "toToken": "SOL", "amount": 100, "userAddress": "..." }
```
