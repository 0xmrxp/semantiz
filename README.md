<div align="center">

# semantiz

**Web4 Decentralized Semantic Search Engine**

[![License](https://img.shields.io/badge/License-Apache%202.0-blue.svg)](LICENSE)
[![Status](https://img.shields.io/badge/Status-Pre--Alpha-orange.svg)]()
[![Python](https://img.shields.io/badge/Python-3.11%2B-3776AB?logo=python&logoColor=white)](https://python.org)
[![Solidity](https://img.shields.io/badge/Solidity-0.8%2B-363636?logo=solidity&logoColor=white)](https://soliditylang.org)
[![React](https://img.shields.io/badge/React-18%2B-61DAFB?logo=react&logoColor=black)](https://react.dev)
[![IPFS](https://img.shields.io/badge/IPFS-Enabled-65C2CB?logo=ipfs&logoColor=white)](https://ipfs.tech)
[![FastAPI](https://img.shields.io/badge/FastAPI-0.100%2B-009688?logo=fastapi&logoColor=white)](https://fastapi.tiangolo.com)
[![Contributions Welcome](https://img.shields.io/badge/Contributions-Welcome-brightgreen.svg)](CONTRIBUTING.md)

*Own your search. Earn your contribution.*

[Architecture](#architecture) · [Tech Stack](#tech-stack) · [Roadmap](#roadmap) · [Blueprint](BLUEPRINT.md) · [Research](RESEARCH.md)

</div>

---

## Overview

The web search market is structurally broken. Google and Bing control over 90% of queries through closed, opaque indices. Users generate the signal that powers ranking improvements yet receive no economic return. Search history is sold to advertisers without explicit consent. Content is optimized for algorithms, not for informational quality.

**semantiz** flips this model. It is a fully decentralized, peer-to-peer semantic search engine where:

- **You own your query history** — no query ever touches a centralized server
- **Nodes earn token rewards** for crawling, indexing, and serving results via Proof of Contribution
- **Content is ranked by meaning**, not by keyword density or paid placement — dense vector embeddings make semantic manipulation computationally expensive
- **No single point of failure** — the network continues operating as long as any subset of nodes remains online
- **Storage is permanent** — indexed content lives on IPFS and Filecoin, not on a company's servers

semantiz draws from proven academic research: the [Semantica](https://arxiv.org/abs/2502.10151) LLM-guided semantic tree overlay (10× more semantic user discovery, 2× better document recall vs. state-of-the-art), the [IPNI](https://docs.ipfs.tech/concepts/ipni/) distributed content routing protocol, [DS⁴](https://link.springer.com/chapter/10.1007/978-3-642-36973-5_96) social-semantic routing, and the [Open Web Index](https://openwebsearch.eu/) open crawl infrastructure.

---

## Architecture

semantiz is organized into four interlocking layers:

```
┌──────────────────────────────────────────────────────────────────┐
│                        APPLICATION LAYER                         │
│   Web UI (React + Tailwind)   │   REST/GraphQL API   │   Browser │
│                               │   (FastAPI)          │   Extension│
└───────────────────────────────┬──────────────────────┬───────────┘
                                │                      │
┌───────────────────────────────▼──────────────────────▼───────────┐
│                         SEMANTIC LAYER                           │
│  EmbeddingService (SBERT/BGE)  │  SemanticIndex (FAISS/Qdrant)  │
│  QueryProcessor (vector ANN)   │  SemanticOverlayNetwork (SON)  │
└───────────────────────────────┬──────────────────────────────────┘
                                │
┌───────────────────────────────▼──────────────────────────────────┐
│                        CONSENSUS LAYER                           │
│  ProofOfContribution (Solidity)  │  ReputationSystem            │
│  RewardDistributor               │  NodeRegistry                │
└───────────────────────────────┬──────────────────────────────────┘
                                │
┌───────────────────────────────▼──────────────────────────────────┐
│                         STORAGE LAYER                            │
│  IPFS (content-addressed)   │  Filecoin (persistence deals)     │
│  IPNI (CID → node routing)  │  Local SQLite (index cache)       │
└──────────────────────────────────────────────────────────────────┘
```

### How a Search Works

```
1. User types:   "how to make coffee without a machine"
2. EmbeddingService converts query → float[768] dense vector
3. QueryProcessor:
   ├── Scans local FAISS index (cache hit path, ~5ms)
   └── Routes to top-K semantically proximate SON peers (P2P path)
4. Results ranked by: cosine_similarity × node_reputation × freshness_score
5. User sees: ranked list of content + IPFS CIDs, never a centralized URL
```

The Semantic Overlay Network (SON) — inspired by Semantica's binary k-means trie — clusters nodes by the semantic centroid of their stored content. A query for "quantum error correction" is automatically routed toward nodes that index physics content, skipping nodes specialized in cooking or finance. This makes retrieval fast without a central index.

---

## Tech Stack

| Layer | Primary | Alternatives |
|---|---|---|
| Smart Contract | Solidity 0.8 + Foundry | Rust (Solana), Move (Aptos) |
| Embedding Models | Sentence-BERT, BGE | OpenAI Ada-002, Llama.cpp (local) |
| Vector Index | FAISS | Qdrant, Milvus |
| P2P Network | libp2p | WebRTC (browser tier), Hypercore |
| Storage | IPFS + Filecoin | Arweave, Storacha |
| Frontend | React 18 + Tailwind CSS | Vue, Svelte |
| Backend API | FastAPI (Python 3.11+) | Go, Node.js |
| Local Index DB | SQLite + sqlite-vec | Postgres + pgvector |
| Build / Contracts | Foundry / Hardhat | |
| Dev Runtime | uv (Python), Bun (JS/TS) | |

---

## Repository Structure

```
semantiz/
├── contracts/          # Solidity smart contracts (Foundry workspace)
│   ├── src/
│   │   ├── ProofOfContribution.sol
│   │   ├── RewardDistributor.sol
│   │   └── NodeRegistry.sol
│   └── test/
├── core/               # Python core engine (uv workspace)
│   ├── crawler/        # CrawlerNode — fetch, extract, normalize
│   ├── embedding/      # EmbeddingService — SBERT, BGE, Ada-002
│   ├── index/          # SemanticIndex — FAISS wrapper + SQLite
│   ├── network/        # libp2p node, DHT router, SON protocol
│   ├── consensus/      # PoC proof generation and verification
│   └── api/            # FastAPI search and management endpoints
├── browser-extension/  # MV3 browser extension (Bun/TypeScript)
│   ├── service-worker/ # Persistent background indexing agent
│   └── popup/          # Extension UI
├── frontend/           # React + Tailwind web UI (Bun/TypeScript)
├── proto/              # Protobuf message definitions
├── scripts/            # Dev setup, deployment, migration scripts
├── docs/               # Additional documentation
├── BLUEPRINT.md        # Full technical architecture specification
├── RESEARCH.md         # Deep analysis of academic references
├── LICENSE             # Apache 2.0
└── README.md
```

---

## Roadmap

### Phase 1 — Core Engine *(Months 1–2)*
- [ ] CrawlerNode: seed URL fetching, HTML extraction, content normalization
- [ ] EmbeddingService: SBERT integration, batch embedding pipeline
- [ ] SemanticIndex: FAISS flat index, SQLite metadata store
- [ ] IPFSStorageAdapter: content add, CID retrieval, pin management
- [ ] SearchAPI: `POST /api/v1/search` with vector similarity ranking
- [ ] Basic test suite and CI pipeline

### Phase 2 — Network + Consensus *(Months 3–5)*
- [ ] libp2p node: Kademlia DHT, Gossipsub, peer discovery
- [ ] Semantic Overlay Network: k-means trie topology, chain-hop routing
- [ ] IPNI-compatible advertisement chain for index distribution
- [ ] PoC smart contract (Solidity) deployed on testnet
- [ ] Node reputation system: coverage, freshness, accuracy, availability scoring
- [ ] Slashing mechanism for malicious/lazy nodes

### Phase 3 — User-Facing *(Months 6–7)*
- [ ] Web UI: React search dashboard with result cards
- [ ] REST API: full public developer API with OpenAPI docs
- [ ] Browser extension: background indexing via Service Worker
- [ ] Network statistics dashboard
- [ ] Mainnet deployment preparation

### Phase 4 — Scale *(Months 8+)*
- [ ] Index sharding across heterogeneous node tiers
- [ ] Browser-tier WebRTC peer integration (lightweight nodes)
- [ ] DAO governance layer
- [ ] Cross-chain reward bridging

---

## Key Concepts

**Proof of Contribution (PoC)** — A smart contract mechanism that rewards nodes proportionally to the quality of their index contributions. Quality is scored across four dimensions: Coverage (unique documents indexed), Freshness (recency of updates), Accuracy (embedding quality validated by peer sampling), and Availability (node uptime). Nodes that submit fraudulent proofs are slashed.

**Semantic Overlay Network (SON)** — Nodes are connected not randomly, but by semantic proximity. Each node computes the mean embedding of its stored content. Nodes with high cosine similarity to each other become direct peers. This makes query routing inherently efficient: a query about climate science propagates through a cluster of nodes that index climate science content.

**Content Addressing** — Every document is identified by its content hash (CID), not its URL. This means identical content from different sources resolves to the same address, duplicates are automatically deduplicated, and content integrity is verified cryptographically.

---

## Research Foundation

semantiz is grounded in five peer-reviewed references. See [RESEARCH.md](RESEARCH.md) for the full technical analysis.

| Reference | Contribution to semantiz |
|---|---|
| [Semantica — arXiv:2502.10151](https://arxiv.org/abs/2502.10151) | Binary k-means trie topology, chain-hop routing, soft clustering (Δ parameter), 10× peer discovery improvement |
| [DS⁴ — Springer ECIR 2013](https://link.springer.com/chapter/10.1007/978-3-642-36973-5_96) | Dual-facet routing: social graph + semantic proximity; trust-enhanced result ranking |
| [Open Web Index](https://openwebsearch.eu/) | Bootstrapping corpus (9.14B URLs, 35 TiB), WARC+Parquet two-layer architecture, content quality classifiers |
| [IPNI Spec](https://docs.ipfs.tech/concepts/ipni/) | Advertisement chain protocol, ContextID-based cluster namespacing, Delegated Routing V1 API |
| [In-Browser Agentic Web — Zenodo:17229737](https://zenodo.org/records/17229737) | Browser-tier peer design: Service Worker agent, ONNX-in-browser embeddings, WebRTC P2P, OPFS vector storage |

---

## Getting Started

> **Note:** semantiz is in pre-alpha. The following setup targets local development and experimentation. Production deployment documentation will be published alongside Phase 1 completion.

**Prerequisites**

- Python 3.11+ (managed via [uv](https://github.com/astral-sh/uv))
- [Bun](https://bun.sh) for frontend and browser extension
- [Foundry](https://book.getfoundry.sh) for smart contract development
- Docker (optional, for IPFS node)
- A running IPFS daemon (`ipfs daemon`)

**Clone and bootstrap**

```bash
git clone https://github.com/0xmrxp/semantiz.git
cd semantiz

# Install Python dependencies
uv sync

# Install frontend dependencies
cd frontend && bun install && cd ..

# Install browser extension dependencies
cd browser-extension && bun install && cd ..

# Build and test contracts
cd contracts && forge build && forge test && cd ..
```

**Run the core API locally**

```bash
# Start IPFS daemon (in a separate terminal)
ipfs daemon

# Start the semantiz API server
uv run python -m core.api.main --host 0.0.0.0 --port 8000

# Test a search
curl -X POST http://localhost:8000/api/v1/search \
  -H "Content-Type: application/json" \
  -d '{"query": "decentralized semantic search", "limit": 10}'
```

---

## Contributing

Contributions are welcome across all layers — Python backend, Solidity contracts, React frontend, and research.

1. Fork the repository
2. Create a feature branch: `git checkout -b feat/your-feature`
3. Commit with clear messages: `git commit -m "feat: add HNSW index support"`
4. Push and open a Pull Request

Please read [BLUEPRINT.md](BLUEPRINT.md) before contributing to understand the intended architecture. For research-oriented contributions, see [RESEARCH.md](RESEARCH.md).

---

## License

Copyright 2026 semantiz contributors

Licensed under the Apache License, Version 2.0. See [LICENSE](LICENSE) for the full license text.

---

<div align="center">

Built on the shoulders of [Semantica](https://arxiv.org/abs/2502.10151), [IPNI](https://docs.ipfs.tech/concepts/ipni/), [OpenWebSearch.EU](https://openwebsearch.eu/), [DS⁴](https://link.springer.com/chapter/10.1007/978-3-642-36973-5_96), and [In-Browser Agentic Web](https://zenodo.org/records/17229737).

</div>
