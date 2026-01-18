### Hyperliquid Design Architecture Overview

Hyperliquid is a custom Layer 1 (L1) blockchain designed primarily as a high-performance platform for decentralized finance (DeFi), with a focus on perpetual futures (perps) trading, spot markets, and broader financial applications. It combines specialized native components for trading efficiency with a general-purpose execution environment for programmability. The architecture is built to address the limitations of traditional blockchains like Ethereum, such as low throughput, high latency, and fragmented liquidity, while aiming for a "fully on-chain open financial system." Below, I'll break down the key components based on official documentation, technical analyses, and ecosystem discussions.

#### Core Components
Hyperliquid's architecture is divided into two main interconnected layers: **HyperCore** and **HyperEVM**, forming a unified global state secured by a custom consensus mechanism. This design allows for seamless interactions between high-performance trading primitives and programmable smart contracts, without the need for bridges, proofs, or trusted intermediaries.

- **HyperCore (Native Execution Layer)**:
  - Handles performant, non-EVM components optimized for trading.
  - Key features:
    - **Fully On-Chain Order Book and Matching Engine**: Processes orders, trades, funding payments, and liquidations directly on-chain. Supports perps with up to 50x leverage, spot trading, and auto-deleveraging (ADL) to prevent bad debt.
    - **Margin and Risk Management**: Includes portfolio margining, borrowing/lending, and liquidation mechanisms. Liquidations first attempt the order book for better prices, with backstop by the Hyperliquid Liquidity Pool (HLP), a community-owned vault.
    - **Staking and Governance**: Uses the native HYPE token for staking, securing the network, and community proposals (e.g., HIP-3 for permissionless market deployments).
    - **Oracles**: Validator-operated for core perps (weighted median from major CEXs); HIP-3 markets allow custom oracles (e.g., via MPC or Pyth Network).
    - **Multi-Sig and Bridges**: Manages assets like USDC via an Arbitrum bridge (being deprecated for native USDC minting) and multi-sig controls for security.
  - State Management: HyperCore maintains margin, matching engine state, and liquidity pools. All trades are verifiable on-chain in real-time.
  - Performance: Designed for sub-second finality (0.07-second block times) and zero-gas trading to eliminate wallet pop-ups and delays.

- **HyperEVM (General-Purpose Layer)**:
  - An Ethereum-compatible virtual machine (EVM) for deploying smart contracts using standard tools like Solidity.
  - Integration with HyperCore: Uses "precompiles" for read/write interactions.
    - **Read Precompiles**: Smart contracts can query HyperCore data (e.g., order book prices) directly.
    - **Write Precompiles**: Contracts can execute actions on HyperCore (e.g., sending orders or swaps) as part of block execution.
  - Enables composability: Builders can deploy ERC-20 tokens, link them to HyperCore tickers for order book trading, or build lending protocols that use HyperCore liquidity for liquidations.
  - Example Workflow: Deploy an ERC-20 on HyperEVM, auction a HyperCore ticker, link them for seamless transfers and trading. This avoids bridging risks and allows permissionless listings.

- **Consensus Mechanism: HyperBFT**:
  - Custom algorithm based on HotStuff and subsequent BFT (Byzantine Fault Tolerance) improvements.
  - Properties:
    - Asynchronous: Consensus doesn't block on execution; transactions are sequenced without waiting for block hashes.
    - Optimistic Responsiveness: Block times bounded only by network delay, not fixed synchronous timers.
    - High Throughput: Supports up to 200,000 TPS (transactions per second), with current bottlenecks at ~20,000 TPS under Tendermint (prior consensus, now migrated).
    - Security: Permissionless validator set; BFT proof-of-stake ensures state verification across nodes.
  - Migration: Rolled out incrementally in 2024 for stability, then scaled for throughput.

- **Overall System Flow**:
  - Users interact via wallets or APIs; transactions are sequenced by HyperBFT validators.
  - Execution: HyperCore handles native ops (e.g., matching); HyperEVM runs contracts that can call into HyperCore.
  - Settlement: All on-chain, with real-time transparency (e.g., trades, positions, liquidations indexed by nodes).
  - Scalability: Horizontal via HIP-3 (permissionless perp deployments) and vertical via native USDC integration to replace bridges.

| Component | Purpose | Key Tech | Performance Metrics |
|-----------|---------|----------|---------------------|
| HyperCore | Native trading primitives | On-chain order book, oracles, staking | 20k-200k TPS; 0.07s blocks |
| HyperEVM | Programmable contracts | EVM-compatible; precompiles for Core integration | Unified state; no bridging |
| HyperBFT | Consensus | Custom BFT (HotStuff-based) | Asynchronous; optimistic responsiveness |
| HLP Vault | Liquidity backstop | Community-owned pool | Permissionless deposits; supports liquidations |

This architecture positions Hyperliquid as a "financial layer" rather than a general blockchain, prioritizing speed and efficiency for DeFi over broad utility.

### Evidence of Centralization

Despite its DeFi branding and claims of progressive decentralization, Hyperliquid exhibits significant centralization in its current implementation. This is not hidden— the team acknowledges it's a "race to decentralize"—but critics argue it operates more like a centralized exchange (CEX) with blockchain window dressing. Here's a substantiated breakdown:

- **Validator Centralization**:
  - Limited Set: As of early 2026, Hyperliquid runs with around 24 validators, many controlled by the team or foundation. This is far below thousands needed for robust decentralization (e.g., Ethereum has ~1M validators).
  - Control Risks: Validators can submit oracle prices (weighted median from CEXs), but a single key could theoretically override (though disputed as HIP-3-specific). Community concerns include "excessive centralization" and potential for downtime or manipulation during upgrades.
  - Examples: In 2025 incidents (e.g., POPCAT volatility, API failures), the chain froze bridges or relied on team intervention, highlighting single points of failure.

- **Governance and Control Mechanisms**:
  - Foundation Influence: The Hyper Foundation holds sway over decisions, including upgrades and fee distributions (97% of revenue buys back/burns HYPE). Critics call this a "double-edged sword," enabling speed but risking long-term centralization.
  - "Godmode" Features: Decompiled code reveals functions like CoreWriter (allows EVM contracts to send Core actions) and ModifyNonCirculatingSupply (alters token supply display). While defended as testnet-only or informational, their presence "undermines the trust model."
  - Transaction Submission: Controlled by ~8 undisclosed addresses, minimizing MEV but enabling potential censorship. Future upgrades aim for MEV-resistant, multi-proposer blocks.

- **Comparison to Competitors**:
  - Hyperliquid claims more transparency than Lighter (single centralized sequencer) or Aster (centralized matching). However, it's less decentralized than fully open chains like Solana.
  - Incidents: 2025's "Black Wednesday" exposed HLP vulnerabilities and coordinated CEX pressure, amplifying centralization fears.

- **Progressive Decentralization Plan**:
  - The team plans full open-sourcing of HyperCore (HyperEVM already is) and validator expansion. However, as of 2026, it's "not considered a CEX" but lacks open node software, raising "trust model" issues.
  - Community Response: X discussions highlight "hidden operators" and "keys that can override prices," with calls for GitHub openness.

In summary, Hyperliquid's centralization stems from team-controlled validators, closed-source elements, and intervention capabilities, making it functionally similar to a CEX despite on-chain transparency claims. This enables its speed but invites scrutiny.

### Source Code and Research Developments Visibility

Hyperliquid is not fully open-source, which limits public scrutiny of its internals and fuels centralization debates. Key points:

- **Closed-Source Core**:
  - Node and validator code remains private, preventing independent verification of execution or consensus. Only HyperEVM execution is open-source.
  - GitHub Presence: Repos like hyperliquid-dex include SDKs (e.g., Python/JS APIs) and some tools, but not the full chain code. Community forks exist for bots, but core logic is absent.

- **Research and Developments**:
  - Opaque: Internal R&D (e.g., HyperBFT optimizations, ADL algorithms) isn't publicly shared beyond high-level announcements. Disputes like the 2025 $653M ADL analysis couldn't be fully verified without code access.
  - Plan: Team states code will open when "secure," prioritizing innovation over immediate transparency. Critics argue this delays accountability.

- **Public Access**:
  - Docs and APIs: Detailed in GitBook, but no full codebase. Third-party tools (e.g., flowsurface for orderflow) rely on public APIs, not source.
  - Community Efforts: Open-source bots and SDKs exist, but they interface with, not reveal, the core system.

This opacity means independent researchers can't audit developments, aligning with user claims of "no one can see the source code." However, on-chain data (e.g., trades) is verifiable via nodes, offering partial transparency.


### Historical Major "Rekts" on Hyperliquid DEX

Hyperliquid, a Layer 1 blockchain-focused perp DEX, has experienced several notable incidents involving significant user or protocol losses (often called "rekts" in crypto slang). These include exploits, manipulations, large-scale liquidations, and user errors. Based on comprehensive research from news sources, on-chain analyses, and X discussions, I've compiled a chronological list of all major historical events up to January 18, 2026. Note that Hyperliquid emphasizes these are not always "hacks" but often result from mechanics like leverage, liquidity, or user actions. Smaller daily liquidations (e.g., individual trades under $10K) are common but not included here as "historical rekts"—focus is on events with $1M+ impact or widespread attention.

I've categorized them by type for clarity, with estimated losses, key details, and context. Total documented losses across these events exceed $100M, though much stems from user leverage rather than protocol flaws.

#### 1. **May 2024: GCR Account Compromise and ETHFI Manipulation**
   - **Description**: Hacker compromised the GCR (@GCRClassic) X account to pump $ETHFI. An associated wallet (0x5e3) opened a $1M long on Hyperliquid just before the fake announcement, but closed at a small loss ($3.5K). This highlighted social engineering risks tied to trading.
   - **Losses**: Minimal direct ($3.5K for attacker), but indirect market volatility led to scattered liquidations.
   - **Impact**: Not a protocol exploit; user/social layer issue. Hyperliquid unaffected, but it underscored integration risks with off-chain signals.

#### 2. **March 2025: ETH Whale Liquidation Incident (0xf3f4 User)**
   - **Description**: A trader opened a massive $200M+ ETH long, built unrealized PNL, withdrew funds to lower margin, and timed a self-liquidation. This caught the HLP (Hyperliquid Liquidity Pool) off-guard, as it backstopped the trade at unfavorable prices. Hyperliquid denied it was a hack, calling it mechanics abuse. Max leverage reduced post-event (BTC to 40x, ETH to 25x).
   - **Losses**: Trader profited ~$1.8M; HLP lost ~$4M (1% of vault; all-time HLP PNL still +$60M). Over $80M in HLP withdrawals followed due to panic.
   - **Impact**: Exposed risks in large-position handling and withdrawal mechanics. Protocol patched leverage rules; community debated if it was an "exploit" or clever trading.

#### 3. **March 2025: JELLY/JELLYJELLY Price Manipulation Exploit**
   - **Description**: Attacker used three accounts for "self-liquidation" on JELLY perps: Deposited $7M quickly, opened $2.15M+ positions, manipulated prices via buy walls, then crashed it. This drained liquidity and forced HLP to cover. JELLY futures were paused.
   - **Losses**: Protocol/HLP ~$4M to $13.5M (reports vary; Arkham cited $4M direct, but cascading effects hit $13.5M). Attacker profited via manipulation.
   - **Impact**: Highlighted thin liquidity risks in altcoin perps. Hyperliquid paused trading; seen as a mechanics abuse rather than code bug. Community called for better oracle/liquidity safeguards.

#### 4. **July 2025: Cross-Chain Reference (GMX Reentrancy Exploit Mentioned in Context)**
   - **Description**: While not directly on Hyperliquid, discussions around perp DEX risks often referenced GMX's $42M reentrancy attack on its GLP pool. Hyperliquid users cited this as a cautionary tale for similar vaults like HLP.
   - **Losses**: N/A for Hyperliquid (GMX-specific: $42M).
   - **Impact**: Increased scrutiny on Hyperliquid's risk engine; no direct incident, but it fueled calls for audits.

#### 5. **September 2025: Major Trader Losses During Volatility**
   - **Description**: High-leverage trades imploded amid market swings, with one trader racking up $43M in losses. Not an exploit, but platform-wide volatility amplified rekts.
   - **Losses**: Individual: $43M; broader DeFi impact in the millions from cascading liquidations.
   - **Impact**: Showed leverage risks; Hyperliquid's speed enabled quick but devastating trades.

#### 6. **October 2025: Market Crash Mass Liquidations**
   - **Description**: Crypto wipeout led to historic liquidations. Four traders lost $10M+ each (e.g., 0x1a67: $18.73M wiped; 0x1d52: $16.43M, $140 left). Total event: Hundreds of millions across perps.
   - **Losses**: ~$100M+ in one day (Hyperliquid share significant); one user lost $21M from private key leak (not protocol fault).
   - **Impact**: Largest single-day rekts; highlighted CEX underreporting vs. on-chain transparency. Hyperliquid CEO noted CEXs hide similar events.

#### 7. **November 2025: POPCAT Price Manipulation Attack**
   - **Description**: Attacker deposited $3M USDC from OKX, split into 19 wallets, opened $20-30M longs, placed $30M buy wall at $0.21 to pump, then removed it causing 43% crash. Triggered mass shorts liquidation.
   - **Losses**: $63M total liquidations; HLP ~$4.9M. Attacker's wallets liquidated but profited indirectly.
   - **Impact**: Second major attack in 2025; exposed altcoin perp vulnerabilities. Trading paused briefly.

#### 8. **November 2025: General Price Manipulation (Second Incident)**
   - **Description**: Follow-up attack using similar tactics; details sparse but involved oracle/price feed abuse.
   - **Losses**: Over $1M (protocol and users).
   - **Impact**: Reinforced need for decentralized oracles; Hyperliquid iterated on HIP-3 for custom feeds.

#### 9. **January 2026: Ongoing Large Liquidations (e.g., BTC, HYPE, FARTCOIN)**
   - **Description**: Recent volatility: $5.1M BTC-USD single liquidation; $7.18M HYPE long capitulation; $1.45M FARTCOIN hit. Total 24h: $97M-$146M across 76K+ traders. Not exploits, but leverage-driven rekts.
   - **Losses**: Daily totals ~$100M+; individuals up to $7M+.
   - **Impact**: Standard for perps (86% trader loss rate per reports); Hyperliquid's 50% market share amplifies visibility.

| Event Date | Type | Key Asset | Estimated Losses | Cause |
|------------|------|-----------|------------------|-------|
| May 2024 | Manipulation | ETHFI | Minimal ($3.5K) | Social hack |
| March 2025 | Self-Liquidation | ETH | $4M (HLP) | Mechanics abuse |
| March 2025 | Exploit | JELLY | $4M-$13.5M | Price manipulation |
| Sept 2025 | Trader Loss | Various | $43M (individual) | Leverage volatility |
| Oct 2025 | Mass Liquidations | BTC/ETH | $100M+ | Market crash |
| Oct 2025 | User Hack | Various | $21M (individual) | Key compromise |
| Nov 2025 | Manipulation | POPCAT | $63M ($4.9M HLP) | Pump/dump |
| Nov 2025 | Manipulation | Unknown | $1M+ | Price attack |
| Jan 2026 | Liquidations | BTC/HYPE/FARTCOIN | $100M+ (daily) | Leverage |

Hyperliquid's team often attributes these to design (e.g., HLP as a risky vault) and has patched issues progressively. No major incidents reported in late 2025 beyond November. For real-time monitoring, check bots like @HyperRektBot or on-chain explorers.



