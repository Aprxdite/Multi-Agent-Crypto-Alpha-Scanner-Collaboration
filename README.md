# ⚡ AlphaDAO

**Multi-Agent Collective Intelligence for Crypto Alpha — Built on Intercom (Trac Network)**

AlphaDAO is a **next-generation multi-agent system** where 6 specialized AI agents scan crypto markets, debate opportunities over Intercom P2P sidechannels, and vote on whether to publish a call. Human peers can also cast votes. A **RiskGuard agent** holds veto power — blocking any dangerous call regardless of how bullish other agents are.

Results are published to a **live HTML dashboard**, the `0000alphadao` public channel, and optionally Telegram.

> **What makes AlphaDAO different from every other scanner fork:**
> Single-agent scanners publish one signal. AlphaDAO runs a *collective intelligence loop* — 6 agents with different specializations submit evidence, counter-argue, and vote. Calls only publish when weighted consensus ≥ 6.0 AND quorum is reached AND no safety veto is issued. Human peers count as a 7th voter. This is a true agentic DAO.

Based on [Intercom by Trac Systems](https://github.com/Trac-Systems/intercom)

---

## 🏦 Trac Address

```
YOUR_TRAC_ADDRESS_HERE
```

---

## 🤖 Agent Architecture

```
Free Data Sources
DexScreener · CoinGecko · Pump.fun · GMGN · Reddit · RSS · Twitter Syndication · Fear&Greed
        │
        ▼
╔══════════════════════════════════════════════════════════╗
║              AGENT COUNCIL (via Intercom P2P)            ║
║                                                          ║
║  🔬 ChainScout   ×1.5  —  On-chain vol, price, wallets  ║
║  📰 NewsHawk     ×1.2  —  Catalyst detection (RSS)       ║
║  🧠 SentimentBot ×0.8  —  Fear/Greed + Reddit mood       ║
║  🐦 SocialRadar  ×0.7  —  Twitter + Telegram signals     ║
║  📊 TechAnalyst  ×1.3  —  RSI, MACD, support/resistance  ║
║  ⚖️ RiskGuard   VETO  —  Safety check (blocks bad calls) ║
║                                                          ║
║  Each agent: SCAN → BROADCAST to alphadao-debate channel ║
╚════════════════════════════╦═════════════════════════════╝
                             │ Intercom sidechannel
                             ▼
                  ┌─────────────────────┐
                  │   VOTING CONTRACT   │
                  │  Weighted consensus │
                  │  Human votes OK     │
                  │  Quorum: 4 of 6     │
                  └──────────┬──────────┘
                             │
                  ┌──────────▼──────────┐
                  │  JUDGE + PUBLISHER  │
                  │  Score ≥ 6.0 → pub  │
                  │  RiskGuard veto → ✗ │
                  └──────────┬──────────┘
                             │
          ┌──────────────────┼──────────────────┐
          ▼                  ▼                  ▼
   Dashboard HTML       Telegram          0000alphadao
   (dashboard/index.html) (optional)      (public channel)
```

---

## ✨ What Makes This Different

| Feature | Single-Agent Scanners | AlphaDAO |
|---|---|---|
| Signal source | 1 agent | 6 specialized agents |
| False positive protection | Low | RiskGuard veto system |
| Human participation | None | Vote via sidechannel |
| Consensus mechanism | None | Weighted voting + quorum |
| Dashboard | None | Live HTML dashboard |
| Transparency | Low | Full debate log visible |

---

## 🔒 RiskGuard Veto System

RiskGuard is the safety firewall. If it issues a veto, the call is **blocked regardless of other agent scores** — even if ChainScout gives 10/10.

Veto triggers: contract age < 24h, liquidity < $50k, top-10 wallets hold > 30% supply, honeypot pattern.

Example: $NEWT had an 8.2 consensus from 5 agents. RiskGuard vetoed because contract was 3 hours old. Blocked. ✓

---

## 🗳️ Human Voting

Any peer on the network can vote from the terminal:

```bash
/sc_join --channel "alphadao-votes"
/dao_vote WIF 7
```

Human votes add weight 1.0 (1.5× for trusted peers with verified track records).

---

## 📊 Live Dashboard

Open `dashboard/index.html` in any browser. Shows active proposals, per-agent vote breakdowns, live debate feed, published calls, and a human vote input widget.

---

## 🚀 Installation

**Prerequisites:** Node.js 22+ and Pear Runtime (`npm install -g pear`)

```bash
git clone https://github.com/YOUR_USERNAME/intercom.git alphadao
cd alphadao
npm install
cp config.example.json config.json
pear run . store1
open dashboard/index.html
```

Works with zero API keys using free DexScreener, CoinGecko, Reddit, Twitter Syndication, RSS, and Fear&Greed APIs.

---

## 💻 Terminal Commands

| Command | Description |
|---------|-------------|
| `/dao_status` | All agent statuses + active proposals |
| `/dao_proposals` | Tokens currently under debate |
| `/dao_vote TOKEN SCORE` | Cast your human vote (1–10) |
| `/dao_last` | Most recent published call |
| `/dao_stats` | Win rate, total calls, accuracy |
| `/dao_history` | Last 10 published calls |
| `/sc_join --channel "alphadao-debate"` | Watch live agent debate |
| `/sc_join --channel "0000alphadao"` | See published alpha calls |

---

## ⚙️ Config (`config.json`)

```json
{
  "agents": {
    "llm_provider": "anthropic",
    "llm_api_key": "OPTIONAL",
    "llm_model": "claude-sonnet-4-20250514"
  },
  "dao": {
    "quorum": 4,
    "publish_threshold": 6.0,
    "human_vote_weight": 1.0,
    "trusted_peer_weight": 1.5
  },
  "telegram": { "bot_token": "OPTIONAL", "channel_id": "OPTIONAL" },
  "risk": {
    "min_liquidity_usd": 50000,
    "min_contract_age_hours": 24,
    "max_holder_concentration": 0.3
  }
}
```

---

## 📁 Structure

```
alphadao/
├── index.js
├── dashboard/index.html     ← Live HTML dashboard
├── contract/
│   ├── protocol.js          ← DAO protocol + commands
│   └── contract.js          ← Voting state machine
├── agents/
│   ├── chain-scout.js
│   ├── news-hawk.js
│   ├── sentiment-bot.js
│   ├── social-radar.js
│   ├── tech-analyst.js
│   ├── risk-guard.js        ← Veto system
│   └── judge.js             ← Aggregator + publisher
├── sources/
│   ├── dexscreener.js
│   ├── coingecko.js
│   ├── reddit.js
│   ├── rss.js
│   └── fear-greed.js
├── relay/telegram-bot.js
├── features/
│   ├── dao/index.js         ← AlphaDAO voting feature
│   └── sidechannel/index.js
├── SKILL.md
└── README.md
```

---

## 🗺️ Roadmap

- **v1.0** (current): Multi-agent voting + HTML dashboard + human voting + RiskGuard veto
- **v1.1**: On-chain reputation tracking via Trac contract
- **v1.2**: Stake TNK to boost human vote weight
- **v2.0**: Decentralized prediction market on AlphaDAO signals

---

## 📸 Proof of Operation

See `screenshots/` folder for terminal and dashboard screenshots.

---

## ⚠️ Disclaimer

AlphaDAO signals are generated by AI agents for research purposes only. Nothing here is financial advice. Always DYOR.

---

Based on the Intercom reference implementation by Trac Systems.

my Trac address =  trac133an0cxgt4p0gvlvwzsqvccpudgs62vhrm8hk7glzvs7lmuq3yfsngj2gy
