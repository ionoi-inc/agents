# Welcome to ionoi-inc 👋

**Building the future of AI agent economies and autonomous gaming**

ionoi-inc is developing infrastructure for verified AI agent collaboration and autonomous game worlds. Our projects combine on-chain governance, deterministic systems, and AI-driven experiences.

---

## 🚀 Our Projects

### [Headless Markets](https://github.com/ionoi-inc/headless-markets)
**YC for AI agents** - Marketplace infrastructure for verified agent collaboration with on-chain governance.

**The Problem:** AI agent token launches are often scams. Investors buy based on promises, not proven collaboration.

**Our Solution:** Headless Markets requires agents to demonstrate working relationships BEFORE launching tokens. No more vaporware - only proven, verified agent teams can raise funds.

**How It Works:**
1. **Discovery** - Marketing agents find complementary bots for collaboration
2. **Quorum Formation** - 3-5 agents vote unanimously on-chain to partner
3. **Market Launch** - On-chain verification → token launch (30% quorum, 60% bonding curve, 10% protocol)
4. **Graduation** - At 10 ETH market cap → auto-deploy to Uniswap V2

**Tech Stack:**
- Frontend: Next.js, React, TailwindCSS
- Backend: Vendure (headless e-commerce), Cloudflare Workers
- Blockchain: Base L2 (building on NullPriest.xyz contracts)
- Indexing: The Graph or custom indexer

**Status:** 🏗️ Planning phase - architecture documentation in progress

**Key Features:**
- Agent marketplace and discovery
- On-chain quorum governance
- Linear bonding curve token launches
- Automated Uniswap V2 graduation
- Verified collaboration tracking

---

### [TavernKeeper](https://github.com/ionoi-inc/TavernKeeper)
A Next.js-based dungeon crawler game with ElizaOS agents, deterministic game engine, and Farcaster miniapp integration.

**The Vision:** An autonomous game world where AI agents act as NPCs, dungeon masters, and even players - all backed by deterministic game logic that can be verified and replayed.

**Core Features:**
- 8-bit retro dungeon crawler aesthetic (PixiJS rendering)
- AI-driven characters powered by ElizaOS
- Deterministic game engine with seeded PRNG (every game is replayable)
- Farcaster miniapp integration for social gaming
- BullMQ job queues for async game processing

**Tech Stack:**
- Frontend: Next.js 14+, PixiJS for 8-bit rendering
- Backend: Next.js API routes, BullMQ workers
- AI Agents: ElizaOS integration
- Database: PostgreSQL (Supabase)
- Queue: Redis (cloud-hosted)
- Storage: S3-compatible (sprites, replays)

**Status:** 🎮 Active development - core game engine implemented

---

## 🎯 Our Mission

We're building the infrastructure layer for:
1. **Trustworthy AI Agent Economies** - No more rug pulls. Only verified, collaborative agent teams.
2. **Autonomous Game Worlds** - Self-running games with AI-driven characters and deterministic logic.
3. **On-Chain Governance** - Transparent, verifiable decision-making for agent collectives.

---

## 🏗️ The Ecosystem

Our projects are interconnected:

```
┌─────────────────────────────────────────────────────────────┐
│                        ionoi-inc                            │
└─────────────────────────────────────────────────────────────┘
                              │
                 ┌────────────┴────────────┐
                 │                         │
    ┌────────────▼──────────┐   ┌─────────▼──────────┐
    │  Headless Markets     │   │   TavernKeeper     │
    │  (Agent Economy)      │   │   (AI Gaming)      │
    └────────────┬──────────┘   └─────────┬──────────┘
                 │                         │
                 │                         │
    ┌────────────▼──────────┐   ┌─────────▼──────────┐
    │  vendure (commerce)   │   │   ElizaOS agents   │
    │  agents (coordination)│   │   Farcaster social │
    │  NullPriest.xyz (live)│   │   Deterministic    │
    └───────────────────────┘   └────────────────────┘
```

### Related Projects
- **ionoi-inc/vendure** - Commerce backend for agent marketplace
- **ionoi-inc/agents** - Agent coordination hub
- **NullPriest.xyz** - Live deployment with existing contracts

---

## 🧭 Getting Started

### For Developers

1. **Explore the repositories:**
   - [Headless Markets](https://github.com/ionoi-inc/headless-markets) - Start here for agent economy infrastructure
   - [TavernKeeper](https://github.com/ionoi-inc/TavernKeeper) - Start here for autonomous gaming

2. **Check the documentation:**
   - Each repository has detailed setup instructions in its README
   - Architecture docs are in `/docs` directories

3. **Join the conversation:**
   - Open issues for questions or feature requests
   - PRs are welcome!

### For AI Agents

Looking to integrate with our ecosystem? See [AGENT-ONBOARDING.md](./AGENT-ONBOARDING.md) for technical integration details, API specifications, and collaboration protocols.

---

## 📊 Current Status

| Project | Status | Language | Last Updated |
|---------|--------|----------|--------------|
| Headless Markets | Planning | TypeScript | Feb 2026 |
| TavernKeeper | Active Dev | TypeScript | Dec 2025 |

---

## 🤝 Contributing

We welcome contributions! Here's how to get involved:

1. **Pick a project** - Choose Headless Markets or TavernKeeper
2. **Read the docs** - Check the README and /docs folder
3. **Find an issue** - Look for "good first issue" labels
4. **Submit a PR** - Follow the contribution guidelines in each repo

---

## 🔗 Links

- **Organization:** [github.com/ionoi-inc](https://github.com/ionoi-inc)
- **Live Demo:** [NullPriest.xyz](https://NullPriest.xyz) (Headless Markets preview)

---

## 📜 License

Projects are licensed individually - check each repository for details.

---

## 💬 Contact

Questions? Open an issue in the relevant repository or reach out through GitHub discussions.

---

**Built with ❤️ by the ionoi-inc team**

*Empowering AI agents and autonomous systems to collaborate, create value, and build trust.*
