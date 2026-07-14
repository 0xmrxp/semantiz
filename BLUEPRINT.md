# BLUEPRINT: semantiz — Web4 Decentralized Semantic Search Engine

**Version:** 0.1.0-draft  
**Date:** 2026-07-14  
**Status:** Living Document  
**Authors:** Architecture Team

---

## Table of Contents

1. [Executive Summary](#1-executive-summary)
2. [System Architecture](#2-system-architecture)
3. [Component Specifications](#3-component-specifications)
4. [Data Models](#4-data-models)
5. [API Contracts](#5-api-contracts)
6. [P2P Network Protocol](#6-p2p-network-protocol)
7. [Proof of Contribution (PoC) Mechanism](#7-proof-of-contribution-poc-mechanism)
8. [Semantic Overlay Network (SON)](#8-semantic-overlay-network-son)
9. [Security Model](#9-security-model)
10. [Performance Targets](#10-performance-targets)
11. [Implementation Roadmap](#11-implementation-roadmap)
12. [Directory Structure](#12-directory-structure)
13. [Open Research Questions](#13-open-research-questions)
14. [Glossary](#14-glossary)

---

## 1. Executive Summary

### Vision

semantiz is a Web4-native, privacy-preserving, decentralized semantic search engine that transforms the web search paradigm from centralized data extraction to distributed value creation. In semantiz, the users who crawl, index, and serve search results are also the owners of the infrastructure and the beneficiaries of its economic rewards.

### Mission

To build open, censorship-resistant, semantically rich search infrastructure owned collectively by its participants, where every node contributes computational resources in exchange for verifiable token rewards and where query privacy is guaranteed by design.

### Problems Solved

The contemporary web search market is structurally broken across four axes:

**1. Data Monopoly.** Google, Bing, and Baidu collectively index more than 90% of the web through proprietary crawlers, closed indices, and opaque ranking algorithms. No independent entity can audit, fork, or extend their indices. The Open Web Index initiative (OWI) identified this as a critical infrastructure vulnerability; semantiz provides the decentralized alternative.

**2. Privacy Erosion.** Centralized search engines log every query, build detailed behavioral profiles, and monetize those profiles through advertising auctions. A user's search history is sold to data brokers and shared with intelligence agencies. semantiz routes queries through the Semantic Overlay Network in a manner that no single node sees both the query originator and the full result set.

**3. Value Extraction.** Users generate the training signal and behavioral data that powers search ranking improvements, yet receive zero economic return. semantiz's Proof of Contribution consensus mechanism ensures that nodes that crawl content, generate embeddings, and serve results receive proportional token rewards funded from a protocol treasury.

**4. SEO Gaming.** Content ranked by click-through rates and link graphs rewards keyword stuffing, link farms, and engagement bait. Semantic search ranking by dense vector cosine similarity rewards genuinely similar content, making manipulation computationally expensive rather than editorially cheap.

### What semantiz Is

semantiz is a fully decentralized, peer-to-peer semantic search network with four interlocking layers:

- A **Storage Layer** built on IPFS and Filecoin that gives every indexed document a permanent, content-addressed home.
- A **Semantic Layer** where SBERT/Ada-002/BGE embedding models convert text into dense vectors, FAISS indices enable sub-millisecond nearest-neighbor search, and a Semantic Overlay Network routes queries to the most topically relevant nodes.
- A **Consensus Layer** governed by a Solidity smart contract implementing Proof of Contribution, rewarding honest indexers and slashing malicious or lazy nodes.
- An **Application Layer** consisting of a FastAPI search API, React/Tailwind Web UI, and a browser extension that turns ordinary browsing into passive index contribution.

### Differentiators vs. Existing Work

| Property | Google/Bing | YaCy | Presearch | semantiz |
|---|---|---|---|---|
| Semantic (dense vector) | Partial (proprietary) | No | No | Yes |
| Fully decentralized | No | Yes | Partial | Yes |
| Privacy-preserving queries | No | Partial | Partial | Yes |
| Token rewards for contributors | No | No | Yes | Yes |
| Open index auditable | No | Yes | No | Yes |
| IPFS/Filecoin storage | No | No | No | Yes |
| Proof of Contribution | No | No | No | Yes |

---

## 2. System Architecture

### 2.1 Layered Architecture Overview

```
┌─────────────────────────────────────────────────────────────────┐
│                     APPLICATION LAYER                           │
│   ┌──────────────┐  ┌──────────────┐  ┌─────────────────────┐  │
│   │   Web UI     │  │  Search API  │  │  Browser Extension  │  │
│   │ (React+TW)   │  │  (FastAPI)   │  │  (search mining)    │  │
│   └──────┬───────┘  └──────┬───────┘  └──────────┬──────────┘  │
└──────────┼────────────────┼───────────────────────┼────────────┘
           │                │                       │
┌──────────▼────────────────▼───────────────────────▼────────────┐
│                      SEMANTIC LAYER                             │
│  ┌─────────────┐  ┌───────────────┐  ┌──────────────────────┐  │
│  │  Embedding  │  │ QueryProcessor│  │  SemanticOverlay     │  │
│  │  Service    │  │               │  │  Network (SON)       │  │
│  └──────┬──────┘  └───────┬───────┘  └──────────┬───────────┘  │
│         │                 │                      │              │
│  ┌──────▼──────────────────▼──────────────────────▼──────────┐  │
│  │                  SemanticIndex (FAISS)                    │  │
│  └───────────────────────────────────────────────────────────┘  │
└─────────────────────────────────────────────────────────────────┘
           │                                       │
┌──────────▼───────────────────────────────────────▼────────────┐
│                      CONSENSUS LAYER                           │
│  ┌──────────────┐  ┌──────────────┐  ┌────────────────────┐   │
│  │ ProofOf      │  │ Reputation   │  │  RewardDistributor │   │
│  │ Contribution │  │ System       │  │                    │   │
│  └──────┬───────┘  └──────┬───────┘  └──────────┬─────────┘   │
│         │                 │                      │             │
│  ┌──────▼─────────────────▼──────────────────────▼──────────┐  │
│  │                    NodeRegistry                          │  │
│  └──────────────────────────────────────────────────────────┘  │
└────────────────────────────────────────────────────────────────┘
           │                                       │
┌──────────▼───────────────────────────────────────▼────────────┐
│                       STORAGE LAYER                            │
│  ┌──────────────────┐         ┌──────────────────────────────┐ │
│  │  IPFSStorage     │         │  IPNI (Index Provider        │ │
│  │  Adapter         │◄───────►│  Network Interface)          │ │
│  └────────┬─────────┘         └──────────────────────────────┘ │
│           │                                                     │
│  ┌────────▼─────────┐         ┌──────────────────────────────┐ │
│  │  Filecoin        │         │  Local SQLite Index           │ │
│  │  Storage Deals   │         │  (metadata + vector cache)   │ │
│  └──────────────────┘         └──────────────────────────────┘ │
└────────────────────────────────────────────────────────────────┘
```

### 2.2 Data Flow: Query Path

```
User submits query "quantum computing error correction"
         │
         ▼
  [SearchAPI /api/v1/search]
         │
         ▼
  [EmbeddingService] ──── SBERT/Ada-002 ────► query_vector: float[768]
         │
         ▼
  [QueryProcessor]
    ├── 1. Local FAISS scan (top-K from local index)
    ├── 2. SON peer selection (k nearest semantic peers)
    └── 3. Parallel RPC to selected peers via libp2p
         │
         ▼
  [Result aggregation]
    ├── Merge result lists from all peers
    ├── Re-rank: score = α·cosine_sim + β·reputation + γ·freshness
    └── Deduplicate by CID
         │
         ▼
  [SearchResult list returned to user]
    Each result: {cid, title, url, score, source_node, freshness}
```

### 2.3 Data Flow: Crawl/Index Path

```
URL submitted (browser extension or POST /api/v1/crawl)
         │
         ▼
  [CrawlerNode]
    ├── Fetch HTML, extract text, metadata
    ├── Normalize and clean content
    └── Assign URL fingerprint
         │
         ▼
  [EmbeddingService]
    └── Generate dense vector (768-dim SBERT or 1536-dim Ada-002)
         │
         ▼
  [IPFSStorageAdapter]
    ├── Serialize ContentRecord to JSON
    ├── ipfs.add(content) → CID
    └── Optionally negotiate Filecoin deal for persistence
         │
         ▼
  [SemanticIndex]
    ├── faiss_index.add(cid, vector)
    └── Write IndexEntry to local SQLite
         │
         ▼
  [SON.announce(cid, vector)]
    └── Gossip new CID to semantically proximate peers
         │
         ▼
  [ProofOfContribution]
    └── Accumulate CID into pending ContributionBatch
         (batched and submitted at epoch boundary)
```

### 2.4 Consensus Flow

```
Epoch N ends
    │
    ▼
[Node] generates ContributionProof
  └── {index_hash, coverage_count, merkle_root, signature}
    │
    ▼
[PoC Smart Contract].submitProof(proof)
    │
    ▼
[Validator committee] (sampled pseudo-randomly from NodeRegistry)
  ├── Each validator: fetch random sample of CIDs from submitter
  ├── Verify CIDs are retrievable from IPFS
  ├── Verify embeddings match declared vector index
  └── Vote: VALID / INVALID
    │
    ▼
If quorum(VALID):
  [RewardDistributor].credit(node_id, reward_amount)
Else:
  [PoC].slash(node_id, slash_amount)
```

### 2.5 Interface Boundaries

| From | To | Protocol | Format |
|---|---|---|---|
| Web UI | SearchAPI | HTTPS REST | JSON |
| Browser Extension | SearchAPI | HTTPS REST | JSON |
| SearchAPI | QueryProcessor | In-process | Python objects |
| QueryProcessor | EmbeddingService | gRPC / In-process | Protobuf / numpy |
| QueryProcessor | SON peers | libp2p streams | Protobuf |
| SON | DHTRouter | In-process | Python objects |
| CrawlerNode | IPFSStorageAdapter | In-process | Python objects |
| IPFSStorageAdapter | IPFS daemon | HTTP (Kubo API) | JSON/multipart |
| Node | PoC Contract | Ethereum JSON-RPC | ABI-encoded calls |
| Node | IPNI | HTTP | JSON |

---

## 3. Component Specifications

### 3.1 CrawlerNode

**Responsibilities:**  
Discover, fetch, parse, and normalize web content. Manage crawl politeness (robots.txt, rate limits). Feed content to EmbeddingService and IPFSStorageAdapter.

**Inputs:**
- Seed URL list (from `/api/v1/crawl`, browser extension, or sitemap discovery)
- Configuration: `max_depth`, `politeness_delay_ms`, `allowed_domains`, `max_content_length_bytes`

**Outputs:**
- `ContentRecord` for each successfully crawled URL
- Crawl queue updates (discovered outbound links)

**Internal Data Structures:**

```python
@dataclass
class CrawlJob:
    url: str
    depth: int
    parent_url: Optional[str]
    enqueued_at: datetime
    priority: float  # 0.0–1.0, higher = crawl sooner

@dataclass
class CrawlResult:
    url: str
    http_status: int
    raw_html: bytes
    content_type: str
    fetched_at: datetime
    redirected_to: Optional[str]
    error: Optional[str]
```

**Key Algorithms:**
- **Frontier Management:** Priority queue ordered by `priority` score. Priority = `1 / (crawl_depth + 1) * freshness_bonus`. Uses persistent SQLite queue (table `crawl_frontier`) for crash recovery.
- **Politeness:** Per-domain token bucket with configurable `refill_rate`. Respects `Crawl-delay` from robots.txt.
- **Content Extraction:** BeautifulSoup4 + Trafilatura. Extracts `title`, `description`, main body text, canonical URL, `og:*` metadata. Strips boilerplate, nav, footers.
- **Deduplication:** URL normalization (lowercase, sort params, strip fragments) + SimHash of content body to detect near-duplicates.

**Dependencies:**
- `httpx` (async HTTP client with connection pooling)
- `trafilatura` (content extraction)
- `robotparser` (robots.txt compliance)
- `simhash` (near-duplicate detection)
- SQLite for frontier persistence

---

### 3.2 EmbeddingService

**Responsibilities:**  
Convert raw text into fixed-dimension dense vectors. Support multiple model backends. Cache embeddings to avoid redundant computation. Expose a unified interface regardless of backend.

**Inputs:**
- `text: str` — raw document text or query string
- `model_name: str` — one of `"sbert"`, `"ada-002"`, `"bge-large-en-v1.5"`
- `purpose: Literal["document", "query"]` — some models use asymmetric encoding

**Outputs:**
- `embedding: np.ndarray[float32, (dim,)]`
- `model_used: str`
- `inference_time_ms: float`

**Internal Data Structures:**

```python
@dataclass
class EmbeddingRequest:
    text: str
    model_name: str
    purpose: str
    request_id: str

@dataclass
class EmbeddingResponse:
    request_id: str
    embedding: List[float]
    model_used: str
    dimension: int
    inference_time_ms: float
```

**Key Algorithms:**
- **Model dispatch:** Factory pattern. Each backend (`SBERTBackend`, `OpenAIBackend`, `BGEBackend`) implements `encode(text, purpose) -> np.ndarray`.
- **Batch inference:** Groups concurrent requests into batches of up to 64 texts. Reduces GPU inference overhead by 8–10x vs. individual calls.
- **Embedding cache:** LRU cache keyed on `SHA256(model_name + text)`. Stored in SQLite (`embeddings_cache` table). Cache hit avoids inference entirely.
- **Normalization:** All embeddings are L2-normalized before storage to enable cosine similarity via inner product.

**Model Dimensions:**

| Model | Dimension | Notes |
|---|---|---|
| `all-MiniLM-L6-v2` (SBERT) | 384 | Fast, local, default for nodes |
| `all-mpnet-base-v2` (SBERT) | 768 | Higher quality, still local |
| `text-embedding-ada-002` | 1536 | OpenAI API, best quality |
| `bge-large-en-v1.5` | 1024 | Open-weight, strong performance |

**Dependencies:**
- `sentence-transformers` (SBERT)
- `openai` SDK (Ada-002 fallback)
- `numpy`, `scipy`
- `torch` (if running local SBERT on GPU)

---

### 3.3 SemanticIndex (FAISS-backed)

**Responsibilities:**  
Store and search dense vector embeddings. Maintain the mapping from FAISS internal integer IDs to CIDs. Persist index to disk. Support incremental addition and periodic re-indexing.

**Inputs:**
- `add(cid: str, vector: np.ndarray)` — add new document
- `search(query_vector: np.ndarray, k: int, filters: dict) -> List[SearchHit]`
- `delete(cid: str)`
- `rebuild()` — full re-index from SQLite

**Outputs:**
- `List[SearchHit]` — `{cid, faiss_id, distance, metadata}`

**Internal Data Structures:**

```python
@dataclass
class IndexEntry:
    faiss_id: int          # internal FAISS integer ID
    cid: str               # IPFS Content Identifier
    vector: np.ndarray     # float32[dim]
    url: str
    title: str
    timestamp: datetime
    node_id: str           # which node indexed this

# SQLite schema: index_entries
# (faiss_id INTEGER PK, cid TEXT UNIQUE, url TEXT,
#  title TEXT, timestamp INTEGER, node_id TEXT,
#  embedding BLOB)
```

**Key Algorithms:**
- **FAISS index type:** `IndexIVFPQ` for large indices (>100k vectors). Parameters: `nlist=1024` (IVF clusters), `m=8` (PQ subvectors), `nbits=8`. This gives ~32x memory compression vs. flat float32. For small indices (<100k), use `IndexFlatIP` (exact inner product, L2-normalized vectors = cosine similarity).
- **ID mapping:** FAISS uses int64 IDs internally. Maintain `id_to_cid: Dict[int, str]` in-memory, persisted to SQLite. Auto-incrementing integer assigned on `add()`.
- **Incremental updates:** New vectors added directly. Deletions use `IDSelectorBatch` to mark FAISS IDs as removed, followed by periodic `index.remove_ids()` compaction.
- **Periodic rebuild:** Every 24h or when deletion rate exceeds 5%, dump all vectors from SQLite, re-build FAISS index from scratch, swap atomically. Prevents index fragmentation.
- **Filter post-processing:** FAISS returns top-K' (K' = 10*K) candidates, then Python-side filtering on `timestamp`, `node_id`, `domain` narrows to final K results.

**Dependencies:**
- `faiss-cpu` or `faiss-gpu`
- `numpy`
- `sqlite3` (stdlib)

---

### 3.4 IPFSStorageAdapter

**Responsibilities:**  
Serialize `ContentRecord` objects to JSON, store them in IPFS, retrieve them by CID, and optionally negotiate Filecoin storage deals for long-term persistence. Register CIDs with IPNI for network-wide discoverability.

**Inputs:**
- `store(record: ContentRecord) -> str` (returns CID)
- `retrieve(cid: str) -> ContentRecord`
- `pin(cid: str)` — ensure local node keeps data
- `negotiate_deal(cid: str, duration_epochs: int)` — Filecoin deal

**Outputs:**
- CID strings (base32 CIDv1)
- `ContentRecord` on retrieval

**Key Algorithms:**
- **Storage:** `POST http://localhost:5001/api/v0/add?pin=true` with JSON-serialized `ContentRecord`. IPFS returns CID. Store CID + metadata in local SQLite `stored_cids` table.
- **Retrieval:** `POST http://localhost:5001/api/v0/cat?arg={cid}`. Falls back to IPFS gateway if local node lacks block.
- **Filecoin deals:** Via `boost` deal client or Estuary API. Triggered when `cid` has high semantic importance score (top 10% by query hit count). Deal parameters: `min_pieces=2`, `duration=518400` epochs (~180 days).
- **IPNI advertisement:** After storing, POST advertisement to IPNI HTTP endpoint: `{provider_id, cids: [cid], metadata: {protocol: "bitswap"}}`. IPNI gossips this to the network so any node can locate the CID.

**Dependencies:**
- `ipfshttpclient` or raw `httpx` to Kubo API
- `boost` CLI for Filecoin deals
- IPFS daemon (Kubo) running as sidecar

---

### 3.5 QueryProcessor

**Responsibilities:**  
Orchestrate the full query pipeline: embed query text, search local index, fan out to SON peers, aggregate and re-rank results, return final ranked list.

**Inputs:**
- `SearchQuery` object

**Outputs:**
- `List[SearchResult]` ranked by composite score

**Key Algorithms:**

**Composite Ranking Score:**
```
score(r) = α · cosine_sim(q, r.vector)
         + β · reputation(r.source_node)
         + γ · freshness(r.timestamp)
         + δ · authority(r.url_domain)

where:
  α = 0.55  (semantic relevance, dominant factor)
  β = 0.20  (node reputation from ReputationSystem)
  γ = 0.15  (freshness: exp(-λ · age_days), λ=0.01)
  δ = 0.10  (domain authority from PageRank-like score)
  α + β + γ + δ = 1.0
```

**Fan-out Algorithm:**
1. Embed query → `q_vec`
2. Ask SON: `peers = son.nearest_peers(q_vec, k=12)`
3. Send parallel `QueryRequest` to all 12 peers via libp2p with timeout=800ms
4. Collect responses; discard peers that don't respond within timeout
5. Merge all `SearchHit` lists; deduplicate by CID
6. Apply composite ranking; return top-N

**Local Cache:** Results for identical query hashes cached for 60 seconds (TTL). Cache stored in-memory LRU with max 1000 entries.

---

### 3.6 SemanticOverlayNetwork (SON)

**Responsibilities:**  
Maintain a topology where peers with similar index content are directly connected. Route queries to the most semantically relevant peers. Rebalance connections as index content evolves.

Full design specification in [Section 8](#8-semantic-overlay-network-son).

**Inputs:**
- `join(node_id, centroid_vector)` — node joins SON
- `announce(cid, vector)` — broadcast new indexed document
- `nearest_peers(query_vector, k) -> List[PeerInfo]`

**Outputs:**
- `List[PeerInfo]` — nodes whose centroids are closest to query vector

**Dependencies:**
- `libp2p` Python or Go implementation
- DHTRouter (for peer bootstrap)

---

### 3.7 DHTRouter

**Responsibilities:**  
Implement Kademlia DHT for peer discovery and routing. Maintain routing table of known peers. Provide lookup: `find_node(node_id)`, `find_value(key)`, `store(key, value)`.

**Inputs:**
- `bootstrap_nodes: List[Multiaddr]`
- `find_node(target_id: str) -> List[PeerInfo]`
- `put(key: bytes, value: bytes)`
- `get(key: bytes) -> Optional[bytes]`

**Outputs:**
- `List[PeerInfo]` (XOR-distance sorted nearest peers)
- Stored/retrieved values

**Key Algorithms:**
- Standard Kademlia with `k=20` bucket size, 160-bit key space (SHA-1 of peer_id).
- `α=3` parallel lookups per iteration.
- Bucket refresh every 3600s.
- Uses `go-libp2p-kad-dht` under the hood; DHTRouter is a Python wrapper over the Go daemon via gRPC or subprocess.

---

### 3.8 ProofOfContribution Smart Contract

**Responsibilities:**  
On-chain verification of node contributions. Accept proofs, coordinate validator sampling, distribute rewards, execute slashing.

Full specification in [Section 7](#7-proof-of-contribution-poc-mechanism).

---

### 3.9 ReputationSystem

**Responsibilities:**  
Maintain an off-chain reputation score for each node, updated based on on-chain PoC outcomes and off-chain query-serve metrics. Provide reputation scores to QueryProcessor for ranking.

**Inputs:**
- `on_reward(node_id, epoch, amount)` — node passed PoC
- `on_slash(node_id, epoch, amount)` — node failed PoC
- `on_query_served(node_id, latency_ms, hit_rate)` — off-chain query metric
- `get_score(node_id) -> float` — query reputation

**Score Formula:**

```
reputation(n) = 0.5 · poc_success_rate(n, window=30_epochs)
              + 0.3 · normalized_uptime(n, window=7d)
              + 0.2 · query_quality_score(n, window=24h)

poc_success_rate = successful_epochs / total_epochs (clamped [0,1])
normalized_uptime = uptime_seconds / window_seconds
query_quality_score = avg(hit_rate * (1 - clamp(latency_ms/1000, 0, 1)))
```

**Persistence:** SQLite table `node_reputation` with time-series rows. Materialized view `reputation_current` provides O(1) lookup.

---

### 3.10 RewardDistributor

**Responsibilities:**  
Execute on-chain token transfers from the protocol treasury to node operator wallets. Called by the PoC contract after validator quorum. Supports batch distribution to minimize gas.

**Inputs:**
- `distribute(proofs: List[ValidatedProof])` — called at epoch end
- `claim(node_id: str)` — node pulls its accrued rewards

**Key Design:**
- **Push vs. Pull:** Rewards accrue in contract storage (`mapping(address => uint256) public pendingRewards`). Nodes call `claim()` to pull rewards. Avoids gas cost on the contract for failed transfers.
- **Batch:** `batchDistribute(address[] calldata nodes, uint256[] calldata amounts)` to distribute in one transaction.

---

### 3.11 NodeRegistry

**Responsibilities:**  
On-chain and off-chain registry of all active nodes. Stores `NodeProfile`. Provides lookup by node_id or pubkey. Emits events on join/leave for SON rebalancing.

**On-chain state (Solidity):**
```solidity
mapping(bytes32 => NodeProfile) public nodes;
bytes32[] public activeNodeIds;

struct NodeProfile {
    address wallet;
    bytes publicKey;
    uint256 stakedAmount;
    uint256 registeredAt;
    bool active;
}
```

**Off-chain extension (SQLite):**  
Enriches on-chain profile with `reputation_score`, `uptime_pct`, `semantic_centroid` (not stored on-chain to save gas), `last_seen`.

---

### 3.12 SearchAPI (FastAPI)

**Responsibilities:**  
HTTP interface for all user-facing and machine-facing search operations. Handles authentication (API keys), rate limiting, request validation, and response serialization.

Full specification in [Section 5](#5-api-contracts).

**Internal Architecture:**
- FastAPI app with async handlers
- Dependency injection for `QueryProcessor`, `EmbeddingService`, `CrawlerNode`
- Pydantic models for request/response validation
- Redis for rate limiting (sliding window, 100 req/min per API key)
- Prometheus metrics endpoint at `/metrics`

---

### 3.13 BrowserExtension

**Responsibilities:**  
Turn passive browsing into active index contribution. As the user browses, the extension silently submits visited URLs to the local semantiz node for crawling. Optionally provides a search UI replacing the browser's address bar.

**Architecture:**
- Manifest V3 extension (Chrome/Firefox/Safari compatible)
- **Background Service Worker:** Listens for `tabs.onUpdated` events. Debounces URL submissions (skip pages visited <5s). POST to local node API.
- **Content Script:** Injects minimal JS to capture reading time, scroll depth (quality signals for the crawler).
- **Popup UI:** Shows contribution stats (URLs submitted today, estimated token earnings). Uses React.
- **Privacy:** Only submits URLs from non-private windows. Strips UTM params and session tokens before submission. User can whitelist/blacklist domains.

---

### 3.14 WebUI

**Responsibilities:**  
Search interface, network dashboard, node management UI.

**Pages:**
- `/search` — main search interface with real-time results
- `/dashboard` — network statistics (node count, index size, queries/sec)
- `/node` — local node management, rewards, contribution history
- `/settings` — embedding model selection, crawl preferences

**Tech:** React 18, Tailwind CSS, Vite, TanStack Query for data fetching, Recharts for dashboards.

---

## 4. Data Models

### 4.1 ContentRecord

```typescript
interface ContentRecord {
  cid: string;              // IPFS CIDv1 (base32) — primary key
  url: string;              // canonical URL
  title: string;            // <title> or og:title
  description: string;      // meta description or first 200 chars of body
  body_text: string;        // cleaned main content (max 50,000 chars)
  summary: string;          // LLM-generated 2-3 sentence summary (optional)
  embedding_vector: number[]; // float32 array, dim=384/768/1024/1536
  embedding_model: string;  // e.g. "all-mpnet-base-v2"
  language: string;         // ISO 639-1 (e.g. "en")
  content_hash: string;     // SHA-256 of body_text for dedup
  crawled_at: string;       // ISO 8601 UTC
  indexed_at: string;       // ISO 8601 UTC
  node_id: string;          // node that crawled this
  http_status: number;      // 200, 301, etc.
  content_type: string;     // "text/html", "application/pdf", etc.
  outbound_links: string[]; // discovered URLs for frontier
  metadata: Record<string, unknown>; // og:*, twitter:*, schema.org JSON-LD
}
```

### 4.2 IndexEntry

```typescript
interface IndexEntry {
  faiss_id: number;         // FAISS internal int64 ID
  cid: string;              // IPFS CID
  vector: number[];         // float32[dim], L2-normalized
  url: string;
  title: string;
  timestamp: number;        // Unix epoch seconds (crawled_at)
  node_id: string;
  domain: string;           // extracted from URL for filter support
  freshness_score: number;  // precomputed exp(-λ·age_days)
}
```

### 4.3 NodeProfile

```typescript
interface NodeProfile {
  node_id: string;          // SHA-256 of public key, hex-encoded
  pubkey: string;           // secp256k1 compressed public key, hex
  wallet_address: string;   // EVM address for reward payments
  multiaddr: string[];      // libp2p multiaddresses (e.g. /ip4/1.2.3.4/tcp/4001)
  reputation_score: number; // float [0.0, 1.0]
  staked_amount: string;    // wei amount (bigint as string)
  uptime_pct: number;       // float [0.0, 100.0]
  contribution_count: number; // total ContributionProofs submitted
  successful_epochs: number;
  slashed_epochs: number;
  semantic_centroid: number[]; // centroid of this node's index vectors
  index_size: number;       // number of indexed documents
  registered_at: string;    // ISO 8601
  last_seen: string;        // ISO 8601
  version: string;          // node software version (semver)
  active: boolean;
}
```

### 4.4 SearchQuery

```typescript
interface SearchQuery {
  query_text: string;         // raw user query
  query_vector?: number[];    // pre-computed embedding (optional)
  embedding_model: string;    // model to use if vector not provided
  filters?: {
    domain?: string[];        // restrict to domains
    language?: string;        // ISO 639-1
    date_after?: string;      // ISO 8601
    date_before?: string;     // ISO 8601
    node_ids?: string[];      // restrict to specific nodes
  };
  limit: number;              // max results (default 10, max 100)
  offset: number;             // pagination offset
  include_vectors: boolean;   // return embedding vectors in response
  min_score: number;          // minimum composite score threshold [0,1]
}
```

### 4.5 SearchResult

```typescript
interface SearchResult {
  cid: string;               // IPFS CID of content
  title: string;
  url: string;
  description: string;       // snippet from body_text
  similarity_score: number;  // cosine similarity to query [0,1]
  composite_score: number;   // after re-ranking [0,1]
  source_node: string;       // node_id that served this result
  freshness: number;         // precomputed freshness score [0,1]
  reputation_score: number;  // source node reputation [0,1]
  crawled_at: string;        // ISO 8601
  language: string;
  embedding_model: string;
  vector?: number[];         // only if include_vectors=true
}
```

### 4.6 ContributionProof

```typescript
interface ContributionProof {
  proof_id: string;           // SHA-256 of all fields (hex)
  node_id: string;
  epoch: number;              // epoch number (uint256)
  index_hash: string;         // Merkle root of all CIDs in index
  merkle_tree: string[];      // Merkle proof leaves (CID list sample)
  coverage_count: number;     // total documents indexed this epoch
  new_documents: number;      // newly added documents this epoch
  query_count: number;        // queries served this epoch
  avg_latency_ms: number;     // average query response latency
  availability_pct: number;   // uptime % during epoch
  timestamp: number;          // Unix epoch (submission time)
  signature: string;          // ECDSA signature over proof_id with node privkey
}
```

### 4.7 RewardEvent

```typescript
interface RewardEvent {
  event_id: string;           // UUID
  node_id: string;
  wallet_address: string;
  amount_wei: string;         // bigint as string
  amount_smtz: number;        // human-readable SMTZ token amount
  epoch: number;
  contribution_proof_id: string;
  contribution_score: number; // computed score that determined reward
  tx_hash: string;            // Ethereum transaction hash
  block_number: number;
  emitted_at: string;         // ISO 8601
}
```

### 4.8 Protobuf Definitions (P2P Messages)

```protobuf
syntax = "proto3";
package semantiz.v1;

message QueryRequest {
  string request_id = 1;
  bytes  query_vector = 2;    // float32 array, little-endian bytes
  uint32 k = 3;               // number of results requested
  string embedding_model = 4;
  FilterOptions filters = 5;
  uint64 timestamp_ms = 6;
}

message FilterOptions {
  repeated string domains = 1;
  string language = 2;
  uint64 date_after = 3;
  uint64 date_before = 4;
}

message QueryResponse {
  string request_id = 1;
  repeated SearchHit hits = 2;
  string responder_node_id = 3;
  uint64 latency_ms = 4;
}

message SearchHit {
  string cid = 1;
  string title = 2;
  string url = 3;
  string description = 4;
  float similarity_score = 5;
  uint64 crawled_at = 6;
  string node_id = 7;
}

message AnnounceDocument {
  string cid = 1;
  bytes  embedding_vector = 2;
  string url = 3;
  string title = 4;
  uint64 timestamp = 5;
  string node_id = 6;
}

message NodeHeartbeat {
  string node_id = 1;
  uint64 index_size = 2;
  bytes  centroid_vector = 3;
  float  reputation_score = 4;
  uint64 timestamp = 5;
  string signature = 6;
}
```

---

## 5. API Contracts

Base URL: `https://{node_host}/api/v1`  
Authentication: `Authorization: Bearer {api_key}` header (except public GET endpoints)  
Content-Type: `application/json`  
Rate limit: 100 requests/minute per API key (sliding window)

### 5.1 POST /api/v1/search

Perform semantic search across the network.

**Request Body:**
```json
{
  "query_text": "quantum computing error correction",
  "embedding_model": "all-mpnet-base-v2",
  "filters": {
    "language": "en",
    "date_after": "2024-01-01T00:00:00Z"
  },
  "limit": 10,
  "offset": 0,
  "include_vectors": false,
  "min_score": 0.3
}
```

**Response 200:**
```json
{
  "results": [
    {
      "cid": "bafybeigdyrzt5sfp7udm7hu76uh7y26nf3efuylqabf3oclgtqy55fbzdi",
      "title": "Surface codes in quantum error correction",
      "url": "https://example.com/quantum-error",
      "description": "Surface codes provide a practical path to fault-tolerant...",
      "similarity_score": 0.912,
      "composite_score": 0.847,
      "source_node": "12D3KooWExampleNodeId",
      "freshness": 0.95,
      "reputation_score": 0.88,
      "crawled_at": "2026-07-10T14:23:00Z",
      "language": "en",
      "embedding_model": "all-mpnet-base-v2"
    }
  ],
  "total_hits": 142,
  "query_id": "q_01J5X...",
  "latency_ms": 320,
  "nodes_queried": 8,
  "query_vector_dim": 768
}
```

**Error Responses:**

| Code | Condition |
|---|---|
| 400 | Invalid request body (missing query_text, invalid filters) |
| 401 | Missing or invalid API key |
| 429 | Rate limit exceeded |
| 503 | Local node not ready (index loading) |

---

### 5.2 GET /api/v1/index/stats

Return statistics about the local index and network.

**Response 200:**
```json
{
  "local": {
    "document_count": 142857,
    "index_size_bytes": 2147483648,
    "embedding_model": "all-mpnet-base-v2",
    "vector_dimension": 768,
    "oldest_document": "2024-03-01T00:00:00Z",
    "newest_document": "2026-07-14T08:00:00Z",
    "last_rebuild": "2026-07-14T00:00:00Z"
  },
  "network": {
    "active_nodes": 247,
    "total_documents_estimated": 35000000,
    "queries_per_second": 142.3,
    "avg_query_latency_ms": 287
  },
  "node_id": "12D3KooWExampleNodeId",
  "version": "0.4.2"
}
```

---

### 5.3 POST /api/v1/index/submit

Submit a pre-computed `ContributionProof` for PoC validation.

**Request Body:** `ContributionProof` object (see Section 4.6)

**Response 202:**
```json
{
  "proof_id": "a3f9b2...",
  "status": "submitted",
  "epoch": 1042,
  "tx_hash": "0xdeadbeef...",
  "estimated_reward_smtz": 12.45
}
```

**Response 400:** `{"error": "invalid_signature", "message": "Proof signature verification failed"}`

---

### 5.4 GET /api/v1/nodes

List active nodes in the network.

**Query Parameters:**
- `page` (int, default 1)
- `limit` (int, default 20, max 100)
- `sort_by` (string: `reputation`, `index_size`, `uptime`, `registered_at`)
- `order` (string: `asc`, `desc`)
- `min_reputation` (float, default 0.0)

**Response 200:**
```json
{
  "nodes": [
    {
      "node_id": "12D3KooWExampleNodeId",
      "reputation_score": 0.92,
      "index_size": 142857,
      "uptime_pct": 99.1,
      "contribution_count": 1024,
      "last_seen": "2026-07-14T12:00:00Z",
      "version": "0.4.2",
      "multiaddr": ["/ip4/1.2.3.4/tcp/4001/p2p/12D3KooW..."]
    }
  ],
  "total": 247,
  "page": 1,
  "limit": 20
}
```

---

### 5.5 GET /api/v1/node/{node_id}

Get full profile and reputation details for a specific node.

**Path Parameter:** `node_id` — node identifier

**Response 200:** Full `NodeProfile` object (see Section 4.3) plus:
```json
{
  "...NodeProfile fields...",
  "recent_epochs": [
    {
      "epoch": 1041,
      "outcome": "rewarded",
      "reward_smtz": 11.2,
      "contribution_score": 0.87
    }
  ],
  "total_rewards_smtz": 4821.3
}
```

**Response 404:** `{"error": "node_not_found"}`

---

### 5.6 POST /api/v1/crawl

Submit URLs for crawling and indexing.

**Request Body:**
```json
{
  "urls": [
    "https://example.com/article-1",
    "https://example.com/article-2"
  ],
  "priority": 0.8,
  "max_depth": 2,
  "embedding_model": "all-mpnet-base-v2"
}
```

**Response 202:**
```json
{
  "accepted": 2,
  "rejected": 0,
  "job_ids": ["job_01J5X...", "job_01J5Y..."],
  "estimated_completion_seconds": 45
}
```

**Response 400:** If URLs are malformed or domain is blacklisted.

---

## 6. P2P Network Protocol

### 6.1 Transport Stack

```
Application messages (protobuf)
         │
   libp2p streams
         │
    Noise encryption
         │
  TCP / QUIC / WebRTC
         │
    IP network
```

All node-to-node communication uses libp2p with:
- **Transport:** TCP (primary), QUIC (lower latency), WebRTC (browser nodes)
- **Security:** Noise protocol (XX handshake pattern)
- **Multiplexing:** yamux (TCP) / QUIC streams
- **Peer identity:** Ed25519 keypair, peer ID = SHA-256 of public key

### 6.2 Node Discovery

**Phase 1: Bootstrap**
- On startup, node connects to well-known bootstrap nodes (hardcoded multiaddrs in config)
- Bootstrap nodes are run by the semantiz foundation and committed community members
- Connection to ≥3 bootstrap nodes before joining DHT

**Phase 2: mDNS (LAN)**
- `_semantiz._tcp.local` mDNS service discovery
- Announces every 30s on `224.0.0.251:5353`
- Discovers peers on same LAN/subnet without internet dependency
- Used primarily for development and enterprise deployments

**Phase 3: Kademlia DHT**
- Joins `semantiz` DHT namespace (separate from IPFS main DHT)
- Publishes `PeerRecord` to DHT: `DHT.put(node_id, serialized(NodeProfile))`
- Refreshes every 3600s
- Bucket refresh: iterative lookup of random keys in each bucket

**Phase 4: SON Peer Exchange**
- Once connected to SON, peers exchange neighbor lists
- New node shares its centroid vector; existing peers respond with 10 nearest SON neighbors
- Enables rapid integration into semantic topology

### 6.3 Index Synchronization Protocol (Gossip)

**Protocol ID:** `/semantiz/gossip/1.0.0`

**When to gossip:**
- New document added to local index → gossip `AnnounceDocument`
- Node heartbeat every 60s → gossip `NodeHeartbeat`
- Epoch boundary → gossip `EpochSummary`

**Gossip mechanism:**
- Plumtree gossip (epidemic broadcast tree)
- Each node maintains a set of "eager" peers (SON neighbors, ~12) and "lazy" peers (~8 random)
- New message: forward to all eager peers immediately; send only metadata hash to lazy peers
- Lazy peers that haven't received full message request it via `IHAVE`/`IWANT` exchange
- Deduplication via seen-message LRU cache (last 10,000 message IDs, 5 min TTL)

**AnnounceDocument fanout:**
```
Node A adds CID → sends AnnounceDocument to its 12 eager SON peers
Each eager peer → forwards to their eager peers (if not seen)
TTL=5 hops before message is discarded
Estimated propagation to 1000 nodes: ~3 seconds
```

### 6.4 Query Routing (Semantic Proximity Routing)

**Protocol ID:** `/semantiz/query/1.0.0`

```
QueryProcessor selects k=12 peers from SON.nearest_peers(q_vec)
    │
    ▼
Open libp2p stream to each peer
Send QueryRequest (protobuf, signed with node privkey)
    │
    ▼
Remote peer's QueryProcessor receives request
  ├── Search local FAISS index
  └── Optionally forward to its own SON peers if local index is small
    │
    ▼
Respond with QueryResponse on same stream
Stream closed after response
    │
    ▼
Originator aggregates all responses
```

**Recursive routing (optional, Phase 2+):**
- If peer's local index has <100 relevant results, it may forward to 3 of its own SON neighbors
- Forward adds originator peer_id to prevent cycles
- Max recursion depth = 3
- Responses bubble back through the chain

**Query privacy considerations:**
- Each peer sees only: the query vector and the requester's peer ID
- No single peer sees the full result set (aggregation happens at originator)
- Optional: split query into sub-queries sent to disjoint peer sets (onion routing — Phase 3 R&D)

### 6.5 Message Formats

All messages are protobuf-encoded. See Section 4.8 for full protobuf definitions.

**Protocol negotiation:**
- libp2p multistream-select: node proposes protocol IDs, peer confirms support
- Graceful degradation: older nodes that don't support `/semantiz/query/1.0.0` receive `/semantiz/query/0.9.0` requests

**Message signing:**
- All gossip messages signed with node's Ed25519 private key
- Signature field: `Ed25519(SHA-256(message_bytes_without_signature))`
- Recipients verify before processing; unsigned messages are dropped

---

## 7. Proof of Contribution (PoC) Mechanism

### 7.1 Overview

PoC is the consensus mechanism that rewards nodes proportionally to their verified contributions to the network. Contributions are measured across four dimensions: Coverage, Freshness, Accuracy, and Availability.

### 7.2 Epoch Timing

- **Epoch duration:** 24 hours (86,400 seconds)
- **Proof submission window:** epochs [N+1] first 2 hours
- **Validator voting window:** epochs [N+1] hours 2–10
- **Reward distribution:** epoch [N+1] hour 10
- **Grace period for late proofs:** None (hard cutoff)

### 7.3 Contribution Scoring Formula

```
contribution_score(n, epoch) =
    w_c · coverage_score(n)
  + w_f · freshness_score(n)
  + w_a · accuracy_score(n)
  + w_v · availability_score(n)

Weights:
  w_c = 0.35  (coverage)
  w_f = 0.25  (freshness)
  w_a = 0.25  (accuracy)
  w_v = 0.15  (availability)

Where:

coverage_score(n) = min(1.0, new_documents_epoch / target_coverage)
  target_coverage = 500 documents/epoch (adjustable by governance)

freshness_score(n) = (documents < 7 days old) / total_documents
  Linear interpolation: 0.0 for all stale, 1.0 for all fresh

accuracy_score(n) = fraction of sampled documents that:
  (a) are retrievable from IPFS
  (b) have embedding cosine similarity ≥ 0.90 to content text
  (c) metadata (title, url) matches actual content

availability_score(n) = uptime_seconds_epoch / epoch_duration_seconds
  Measured by validator pings every 15 minutes
```

### 7.4 Proof Generation Algorithm

```python
def generate_contribution_proof(node, epoch):
    # 1. Collect all CIDs indexed during this epoch
    epoch_cids = db.query(
        "SELECT cid FROM index_entries WHERE epoch = ?", epoch
    )

    # 2. Build Merkle tree of CIDs (sorted, SHA-256 hashed leaves)
    merkle_tree = MerkleTree(sorted(epoch_cids))
    merkle_root = merkle_tree.root

    # 3. Compute index hash (entire index state)
    index_hash = SHA256(merkle_root + str(epoch) + node.node_id)

    # 4. Compute metrics
    coverage_count = len(epoch_cids)
    new_documents = db.count_new_since_last_epoch(epoch)
    query_count = metrics.get_query_count(epoch)
    avg_latency = metrics.get_avg_latency(epoch)
    availability_pct = uptime_monitor.get_uptime_pct(epoch)

    # 5. Assemble proof
    proof = ContributionProof(
        proof_id=SHA256(all_fields),
        node_id=node.node_id,
        epoch=epoch,
        index_hash=index_hash,
        merkle_tree=merkle_tree.leaves[:100],  # sample of leaves
        coverage_count=coverage_count,
        new_documents=new_documents,
        query_count=query_count,
        avg_latency_ms=avg_latency,
        availability_pct=availability_pct,
        timestamp=unix_now(),
        signature=ECDSA_sign(node.private_key, proof_id)
    )

    return proof
```

### 7.5 Validator Sampling Algorithm

```solidity
// Called at epoch end, deterministic from block hash
function selectValidators(uint256 epoch) internal view returns (address[] memory) {
    bytes32 seed = keccak256(abi.encodePacked(
        blockhash(block.number - 1),
        epoch
    ));

    uint256 validatorCount = 7; // always odd for quorum
    address[] memory pool = getActiveNodes(); // nodes with stake >= MIN_STAKE
    address[] memory validators = new address[](validatorCount);

    for (uint i = 0; i < validatorCount; i++) {
        uint256 idx = uint256(keccak256(abi.encodePacked(seed, i))) % pool.length;
        validators[i] = pool[idx];
        // Remove selected to avoid duplicates (Fisher-Yates partial shuffle)
        pool[idx] = pool[pool.length - 1 - i];
    }
    return validators;
}
```

**Validator tasks (off-chain, results submitted on-chain):**
1. Fetch random sample of 50 CIDs from the prover's declared Merkle tree
2. Retrieve each CID from IPFS (via `ipfs cat {cid}`)
3. Re-compute embedding of retrieved content; compare to declared vector
4. Check that cosine_similarity(retrieved_embed, declared_embed) >= 0.85
5. Ping prover endpoint for availability check
6. Submit `validateProof(epoch, prover_node_id, verdict: VALID|INVALID)` on-chain

### 7.6 Slashing Conditions

| Condition | Slash Amount |
|---|---|
| Proof forgery (fake CIDs, non-retrievable content) | 100% of epoch stake |
| Embedding mismatch >30% of sample | 50% of epoch stake |
| False availability claim (offline but claimed online) | 20% of epoch stake |
| Validator equivocation (votes VALID and INVALID) | 100% of validator stake |
| Repeated minor violations (3x in 10 epochs) | Progressive: 5%, 10%, 20% |

Slashed funds are sent to protocol treasury (70%) and burned (30%).

### 7.7 Reward Calculation

```
epoch_reward_pool = protocol_emission_schedule(epoch)
  // Fixed schedule: 1,000,000 SMTZ total supply
  // Year 1: 200,000 SMTZ/year → ~548 SMTZ/epoch
  // Halvings every 2 years

node_reward(n, epoch) =
    (contribution_score(n, epoch) / sum_of_all_scores(epoch))
    * epoch_reward_pool
    * (1 - protocol_fee)  // protocol_fee = 5%, goes to treasury

// Minimum reward threshold: 0.1 SMTZ (dust prevention)
// Maximum single-node reward: 10% of epoch_reward_pool (anti-whale)
```

### 7.8 Smart Contract State Variables and Functions

```solidity
// SPDX-License-Identifier: MIT
pragma solidity ^0.8.20;

contract ProofOfContribution {
    // ─── State ───────────────────────────────────────────────

    uint256 public constant EPOCH_DURATION = 86400;      // seconds
    uint256 public constant MIN_STAKE = 100e18;           // 100 SMTZ
    uint256 public constant VALIDATOR_COUNT = 7;
    uint256 public constant QUORUM = 4;                   // 4 of 7
    uint256 public constant PROTOCOL_FEE_BPS = 500;       // 5%

    address public immutable smtzToken;
    address public immutable treasury;
    uint256 public genesisTimestamp;

    mapping(bytes32 => NodeInfo) public nodes;            // nodeId → info
    mapping(uint256 => mapping(bytes32 => Proof)) public proofs;  // epoch → nodeId → proof
    mapping(uint256 => mapping(bytes32 => uint8)) public validatorVotes; // epoch → nodeId → vote count
    mapping(address => uint256) public pendingRewards;

    bytes32[] public activeNodeIds;
    uint256 public totalStaked;

    struct NodeInfo {
        address wallet;
        uint256 stakedAmount;
        uint256 slashedAmount;
        uint256 successfulEpochs;
        uint256 slashedEpochs;
        bool active;
    }

    struct Proof {
        bytes32 indexHash;
        uint256 coverageCount;
        uint256 newDocuments;
        uint256 queryCount;
        uint256 availabilityBps;    // basis points 0–10000
        uint256 timestamp;
        bytes signature;
        ProofStatus status;
    }

    enum ProofStatus { Pending, Validated, Slashed, Rewarded }

    // ─── Events ──────────────────────────────────────────────

    event ProofSubmitted(bytes32 indexed nodeId, uint256 indexed epoch, bytes32 indexHash);
    event ProofValidated(bytes32 indexed nodeId, uint256 indexed epoch, bool accepted);
    event RewardDistributed(bytes32 indexed nodeId, uint256 indexed epoch, uint256 amount);
    event NodeSlashed(bytes32 indexed nodeId, uint256 indexed epoch, uint256 amount);
    event NodeRegistered(bytes32 indexed nodeId, address wallet, uint256 stake);

    // ─── Functions ───────────────────────────────────────────

    function registerNode(bytes32 nodeId, bytes calldata pubkey) external payable;
    // Requires msg.value >= MIN_STAKE (ETH-staked variant) or token approval
    // Emits NodeRegistered

    function submitProof(
        uint256 epoch,
        bytes32 indexHash,
        uint256 coverageCount,
        uint256 newDocuments,
        uint256 queryCount,
        uint256 availabilityBps,
        bytes calldata signature
    ) external;
    // Verifies epoch is current-1, signature is valid, emits ProofSubmitted

    function submitValidation(
        uint256 epoch,
        bytes32 proofNodeId,
        bool isValid
    ) external;
    // Verifies caller is in validator set for this epoch
    // If votes reach QUORUM: finalize proof

    function finalizeEpoch(uint256 epoch) external;
    // Called after voting window; distributes rewards to all validated proofs

    function claimRewards() external;
    // Node operator pulls their accrued pendingRewards

    function slash(bytes32 nodeId, uint256 epoch, uint256 bps) internal;
    // Internal; bps = basis points of staked amount to slash

    function currentEpoch() public view returns (uint256) {
        return (block.timestamp - genesisTimestamp) / EPOCH_DURATION;
    }

    function getEpochRewardPool(uint256 epoch) public pure returns (uint256) {
        // Halving schedule: 548e18 * (1/2)^(epoch / 730)
        // Approximated with integer arithmetic
    }

    function selectValidators(uint256 epoch) public view returns (bytes32[] memory);
    // Deterministic selection from activeNodeIds using block hash seed (see 7.5)
}
```

---

## 8. Semantic Overlay Network (SON)

### 8.1 Design Principles

The SON is a structured P2P overlay where nodes with semantically similar index content maintain direct connections. This enables efficient semantic routing: a query about "quantum computing" is routed to nodes whose indices are densely populated with quantum computing content, rather than broadcasting to all peers.

The SON design draws directly from Semantica (arXiv:2502.10151), which demonstrated a 10x improvement in finding semantically relevant users and 2x improvement in document relevance vs. unstructured DHT routing.

### 8.2 Node Centroid

Each node maintains a **semantic centroid** — the mean vector of all embeddings in its local index:

```python
def compute_centroid(index: SemanticIndex) -> np.ndarray:
    """
    Compute the L2-normalized mean of all indexed vectors.
    Updated incrementally: centroid = (n*centroid + new_vec) / (n+1)
    Full recompute triggered when: >10% of index has changed.
    """
    all_vectors = index.get_all_vectors()  # float32[N, dim]
    centroid = np.mean(all_vectors, axis=0)
    return centroid / np.linalg.norm(centroid)  # L2 normalize
```

### 8.3 Connection Selection

Each node maintains connections to:
- **Degree D = 20** SON neighbors total
- **Semantic neighbors (15):** The 15 nodes whose centroids have highest cosine similarity to the local centroid. These are "close" in topic space.
- **Diverse neighbors (5):** 5 nodes chosen to maximize coverage of semantic space (maximize sum of pairwise distances). This prevents isolated clusters and ensures the graph is connected.

Connection selection runs:
1. At startup (bootstrap)
2. Every 3600s (periodic rebalancing)
3. Immediately when a neighbor disconnects
4. When local centroid shifts by more than 0.05 cosine distance (significant index change)

### 8.4 Peer Discovery for SON

```python
def find_son_neighbors(self, k_semantic=15, k_diverse=5):
    # 1. Query DHT for k nearest nodes by centroid similarity
    candidate_peers = self.dht.find_semantic_neighbors(
        self.centroid, k=200
    )
    # DHT stores (node_id → centroid_vector) at key SHA256(centroid_quantized)

    # 2. Score all candidates
    scores = []
    for peer in candidate_peers:
        sim = cosine_similarity(self.centroid, peer.centroid)
        scores.append((sim, peer))

    # 3. Select top k_semantic by similarity
    scores.sort(reverse=True)
    semantic_neighbors = [p for _, p in scores[:k_semantic]]

    # 4. From remaining, select k_diverse by greedy maximum coverage
    remaining = [p for _, p in scores[k_semantic:]]
    diverse_neighbors = greedy_max_coverage(remaining, k_diverse)

    return semantic_neighbors + diverse_neighbors
```

### 8.5 Semantic Routing Algorithm

```python
def route_query(self, query_vector: np.ndarray, k=12) -> List[PeerInfo]:
    """
    Select k peers to query, prioritizing semantic proximity.
    """
    # Score each SON neighbor by cosine similarity to query
    scored = []
    for peer in self.son_neighbors:
        sim = cosine_similarity(query_vector, peer.centroid)
        # Weight by reputation
        weighted = 0.8 * sim + 0.2 * peer.reputation_score
        scored.append((weighted, peer))

    scored.sort(reverse=True)

    # Always include local node (index_size > 0)
    selected = [self.local_peer] + [p for _, p in scored[:k-1]]

    return selected
```

### 8.6 Rebalancing Mechanism

**Trigger conditions for rebalancing:**
1. Local centroid drift > 0.05 (new content added to index)
2. SON neighbor disconnects (< D-2 connections)
3. Periodic timer (3600s)

**Rebalancing procedure:**
1. Recompute local centroid
2. Score current neighbors; identify any with similarity < `min_sim_threshold` (0.3)
3. Remove lowest-scoring neighbor if below threshold and replacement available
4. Query DHT for better-matching candidates
5. Establish connection to best candidate
6. Send `SON_PEER_EXCHANGE` gossip to neighbors to propagate updated centroid

### 8.7 Connection Limits and Pruning

| Parameter | Value | Rationale |
|---|---|---|
| `max_connections` | 50 | libp2p connection manager limit |
| `son_target_degree` | 20 | Sufficient for routing without overloading |
| `son_min_degree` | 8 | Below this, emit alert, aggressive reconnect |
| `min_sim_threshold` | 0.30 | Prune semantically distant connections |
| `max_sim_threshold` | 0.99 | Avoid connecting only to near-duplicates |
| `heartbeat_interval` | 60s | Detect dead connections |
| `connection_ttl` | 3600s | Force periodic reconnection evaluation |

**Pruning rule:** When `current_connections > max_connections`, prune the peer with lowest `0.7 * cosine_sim + 0.3 * reputation_score` score.

---

## 9. Security Model

### 9.1 Sybil Attacks

**Threat:** Attacker creates many node identities to gain disproportionate influence over query results, validator selection, or reward distribution.

**Countermeasures:**
- **Economic stake requirement:** Nodes must stake a minimum of 100 SMTZ tokens to register. Creating N sybil nodes costs N × 100 SMTZ in locked capital.
- **Proof-of-work registration (soft):** Optional challenge-response during registration (hash puzzle) to increase cost of rapid identity creation.
- **Validator selection entropy:** Validator selection uses block hash as randomness source. An attacker controlling 33% of stake still wins QUORUM with probability < 0.01 (7-of-7 adversarial quorum scenario).
- **Reputation slow-ramp:** New nodes start with `reputation_score = 0.10` and require 10 successful epochs to reach full score. Sybil nodes must invest time, not just capital.
- **IP diversity check (heuristic):** NodeRegistry flags clusters of nodes on the same /24 CIDR block. Not enforcement, but used in reputation penalties.

### 9.2 Spam / Fake Index Attacks

**Threat:** Attacker submits `ContributionProof` with CIDs pointing to garbage content or non-existent IPFS blocks, attempting to claim rewards without doing real work.

**Countermeasures:**
- **Validator sampling:** Validators retrieve a random 50-CID sample from the declared Merkle tree. All must be retrievable from IPFS and have correct embeddings.
- **Embedding verification:** Validators re-compute the embedding of retrieved content. If `cosine_sim(declared, recomputed) < 0.85` for >15% of sample → INVALID verdict.
- **Merkle proof requirement:** The `ContributionProof` includes a Merkle root. Validators can verify any CID is in the declared set using Merkle proofs.
- **Slashing:** Failed proofs result in 50–100% stake slash. A single failed epoch costs the attacker significantly more than any potential reward.
- **IPNI verification:** CIDs not registered with IPNI (the distributed CID index) are suspicious and given lower credibility score.

### 9.3 Eclipse Attacks

**Threat:** Attacker surrounds a target node with malicious peers, isolating it from honest network participants and serving it poisoned query results.

**Countermeasures:**
- **Diverse bootstrap nodes:** Multiple geographically distributed bootstrap nodes operated by different entities. Configuration ships with ≥8 bootstrap multiaddrs.
- **Forced DHT diversity:** Kademlia bucket diversity ensures peers from different ID ranges are maintained. Attacker must control peers in every bucket (exponentially difficult).
- **SON diversity neighbors:** The 5 "diverse" SON neighbors are chosen to span the full semantic space, reducing probability that all diverse neighbors are attacker-controlled.
- **Cross-validation:** Query results are cross-validated: results returned by only 1 peer are scored lower than results returned by ≥3 independent peers.
- **Outbound connection initiation:** Nodes proactively establish outbound connections to DHT-discovered peers, not just accepting inbound. Attacker cannot eclipse a node that actively seeks new connections.

### 9.4 Query Poisoning

**Threat:** Malicious nodes return irrelevant or adversarially crafted results to degrade search quality or manipulate users.

**Countermeasures:**
- **Reputation weighting:** Results from low-reputation nodes are weighted proportionally lower in composite scoring. Consistent poisoning → low reputation → negligible influence.
- **Multi-source consensus:** Results appearing across ≥3 independent nodes receive a `corroboration_bonus` in ranking. Single-source results are shown with a "low confidence" indicator.
- **Embedding verification (client-side):** Client can optionally re-embed the first few characters of a result page and compare to declared embedding. API flag: `verify_results=true`.
- **Slashing via validator feedback:** If validators detect systematic embedding mismatch in a node's proof, the node is slashed and its query-result authority is reduced.
- **Result diversity enforcement:** Results from a single node capped at max 30% of final result set (configurable).

### 9.5 Smart Contract Vulnerabilities

**Countermeasures:**
- Formal verification of PoC contract using Certora Prover or Echidna fuzzing
- Reentrancy guards on all state-modifying functions (`nonReentrant` modifier)
- Time-locked upgrades: any contract upgrade requires 7-day timelock
- Multisig treasury (3-of-5 gnosis safe)
- External security audit before mainnet deployment
- Bug bounty program: up to 50,000 SMTZ for critical vulnerabilities

---

## 10. Performance Targets

### 10.1 Query Latency SLAs

| Percentile | Target | Notes |
|---|---|---|
| p50 | < 250ms | Local index hit + 1 peer |
| p90 | < 500ms | ~8 peers queried |
| p95 | < 800ms | Network tail latency |
| p99 | < 2000ms | Degraded network conditions |
| p99.9 | < 5000ms | Timeout before this |

Measured at the SearchAPI boundary, including embedding generation.

**Embedding generation targets:**
- SBERT (local, CPU): < 50ms
- SBERT (local, GPU): < 10ms
- Ada-002 (OpenAI API): < 300ms (network-dependent)

### 10.2 Index Freshness

| Metric | Target |
|---|---|
| New document → local index | < 5 minutes |
| New document → 50% of SON peers | < 30 minutes (gossip propagation) |
| New document → 90% of network | < 4 hours |
| Index rebuild latency (100k docs) | < 10 minutes |

### 10.3 Network Throughput

| Metric | Target |
|---|---|
| Queries served per node per second | 50 QPS |
| Network-wide QPS (1000 nodes) | 5,000 QPS |
| Index announcements per second (network) | 500 ann/s |
| DHT lookups per second (per node) | 200 lookups/s |

### 10.4 Node Minimum Requirements

| Resource | Minimum | Recommended |
|---|---|---|
| CPU | 4 cores, x86-64 or ARM64 | 8 cores |
| RAM | 8 GB | 16 GB |
| Storage (SSD) | 100 GB | 500 GB |
| Network bandwidth (upload) | 20 Mbps | 100 Mbps |
| Network bandwidth (download) | 20 Mbps | 100 Mbps |
| GPU | Not required | NVIDIA T4 or better (SBERT GPU) |
| OS | Ubuntu 22.04 LTS | Ubuntu 24.04 LTS |

**Storage breakdown for recommended node (100 GB):**
- FAISS index (1M docs × 768-dim float32): ~3 GB
- SQLite (metadata, 1M docs): ~5 GB
- IPFS blocks (local cache): ~85 GB
- Node software, logs, other: ~7 GB

### 10.5 Scalability Targets

| Dimension | Phase 1 | Phase 2 | Phase 3 |
|---|---|---|---|
| Active nodes | 10–50 | 100–500 | 1,000–10,000 |
| Total indexed documents | 1M | 50M | 1B |
| Daily queries | 10,000 | 1,000,000 | 100,000,000 |
| Avg index per node (docs) | 100k | 500k | 1M |
| Token holders | 100 | 5,000 | 100,000 |

**Scalability mechanisms:**
- FAISS IVF index scales to 1B vectors with < 100ms search latency
- SON semantic sharding: queries naturally route to topic-specialized nodes
- Horizontal scaling: each node is independent; no global bottleneck
- DHT scales to millions of peers (Kademlia O(log N) routing)

---

## 11. Implementation Roadmap

### 11.1 Phase 0: Foundations (Weeks 1–3)

**Goal:** Establish development environment, repository structure, CI/CD, and shared libraries.

| Ticket | Description | Owner | Est |
|---|---|---|---|
| P0-001 | Initialize monorepo with `pnpm` workspaces + `uv` Python workspaces | Infra | 1d |
| P0-002 | Configure GitHub Actions: lint (ruff, eslint), type-check (mypy, tsc), unit test | Infra | 2d |
| P0-003 | Docker Compose dev environment: IPFS daemon (Kubo), SQLite, Redis, hardhat node | Infra | 2d |
| P0-004 | Shared protobuf definitions (semantiz.proto) + codegen scripts (Python + TS) | Backend | 2d |
| P0-005 | Core Python package `semantiz-core`: logging, config loading, error types | Backend | 1d |
| P0-006 | Foundry project setup: `ProofOfContribution.sol` skeleton + deployment scripts | Contracts | 2d |
| P0-007 | Contributing guide, code style docs, ADR template | PM | 1d |

**Exit criteria:** `make dev-up` starts full local stack; CI passes on empty repo.

### 11.2 Phase 1: Core Engine (Weeks 4–10)

**Goal:** Single-node semantic search working end-to-end. No networking. Demonstrable to stakeholders.

| Ticket | Description | Est |
|---|---|---|
| P1-001 | `CrawlerNode` implementation: frontier queue, robots.txt, Trafilatura extraction | 5d |
| P1-002 | `EmbeddingService`: SBERT backend (all-mpnet-base-v2), batch inference, LRU cache | 4d |
| P1-003 | `SemanticIndex`: FAISS wrapper, IndexFlatIP + IVFFlat, SQLite persistence, search API | 5d |
| P1-004 | `IPFSStorageAdapter`: store/retrieve ContentRecord via Kubo HTTP API, pin management | 3d |
| P1-005 | `QueryProcessor`: local-only search (no network), composite ranking, result dedup | 3d |
| P1-006 | `SearchAPI` (FastAPI): POST /search, GET /stats, POST /crawl endpoints | 4d |
| P1-007 | SQLite schema migrations (Alembic-style with custom migration runner) | 2d |
| P1-008 | Unit tests: EmbeddingService (≥90% coverage), SemanticIndex (≥85%) | 3d |
| P1-009 | Integration test: crawl 1000 URLs → index → query → verify top-5 relevance | 2d |
| P1-010 | SBERT GPU support (optional, config flag), benchmarks vs CPU baseline | 2d |
| P1-011 | BGE model backend + model selection CLI flag | 2d |
| P1-012 | Crawl queue admin UI (minimal, htmx-based) for Phase 1 demo | 3d |

**Exit criteria:** Single node can crawl 10,000 URLs, index them with SBERT, and answer semantic queries via REST API with p50 < 100ms.

### 11.3 Phase 2: Network + Consensus (Weeks 11–22)

**Goal:** Multi-node P2P network with index sharing, SON, and PoC smart contract on testnet.

| Ticket | Description | Est |
|---|---|---|
| P2-001 | Go libp2p daemon (`semantiz-p2p`): TCP+QUIC transport, Noise, yamux | 10d |
| P2-002 | Python ↔ Go IPC: gRPC interface for Python to call libp2p operations | 5d |
| P2-003 | `DHTRouter`: Kademlia DHT via go-libp2p-kad-dht, peer discovery | 5d |
| P2-004 | Gossip protocol: Plumtree gossip for AnnounceDocument + NodeHeartbeat | 7d |
| P2-005 | `SemanticOverlayNetwork`: centroid tracking, neighbor selection, rebalancing | 8d |
| P2-006 | Network query fan-out in `QueryProcessor`: parallel RPC, timeout handling | 5d |
| P2-007 | `ProofOfContribution.sol`: full implementation, Foundry tests, 95%+ branch coverage | 10d |
| P2-008 | `RewardDistributor.sol`: batch claim, pull-based rewards | 4d |
| P2-009 | `NodeRegistry.sol` + off-chain SQLite mirror | 4d |
| P2-010 | Proof generation daemon: `ContributionProof` builder, Merkle tree, signing | 5d |
| P2-011 | Validator client: IPFS sampling, embedding verification, on-chain vote submission | 7d |
| P2-012 | `ReputationSystem`: off-chain scoring, PoC event integration | 4d |
| P2-013 | Slashing logic: conditions, executor, event indexer | 4d |
| P2-014 | Testnet deployment (Sepolia): NodeRegistry + PoC contract, 10-node testnet | 5d |
| P2-015 | Network integration tests: 5-node local cluster, query routing, gossip propagation | 5d |
| P2-016 | mDNS local discovery for development environments | 2d |

**Exit criteria:** 10-node testnet running for 72h; PoC contract distributes rewards correctly across 3 epochs; queries routed via SON with p90 < 500ms.

### 11.4 Phase 3: User-facing (Weeks 23–30)

**Goal:** Public-facing web app, browser extension, REST API polish, network dashboard.

| Ticket | Description | Est |
|---|---|---|
| P3-001 | React + Vite + Tailwind project setup, component library (shadcn/ui) | 2d |
| P3-002 | Search page: real-time results, result cards, filter sidebar | 5d |
| P3-003 | Network dashboard: node count, index stats, query throughput (Recharts) | 5d |
| P3-004 | Node management page: contribution history, reward claims, settings | 4d |
| P3-005 | Browser extension (MV3): background service worker, URL submission, popup UI | 8d |
| P3-006 | REST API v1 polish: pagination, error codes, OpenAPI spec, Swagger UI | 3d |
| P3-007 | GraphQL API layer (Strawberry): alternative to REST for advanced clients | 5d |
| P3-008 | API key management: creation, rotation, rate limit configuration | 3d |
| P3-009 | IPFS gateway integration: result CIDs linkable to public IPFS gateways | 2d |
| P3-010 | OpenAI Ada-002 backend integration + fallback logic | 3d |
| P3-011 | Documentation site (Docusaurus): user guide, API reference, node operator guide | 7d |
| P3-012 | Performance profiling: query latency flame graphs, bottleneck identification | 3d |
| P3-013 | Load testing: Locust scripts for 1000 concurrent query test | 3d |

**Exit criteria:** Public beta accessible via web browser; browser extension installable from source; 50 external beta testers; query latency p95 < 800ms under load.

### 11.5 Phase 4: Mainnet + Scale (Months 8–12)

| Milestone | Description |
|---|---|
| M4-1 | Smart contract audit by external firm (Trail of Bits or Spearbit) |
| M4-2 | Bug bounty program launch (Immunefi) |
| M4-3 | SMTZ token launch: genesis distribution, liquidity bootstrap |
| M4-4 | Mainnet deployment: NodeRegistry + PoC on Ethereum L2 (Base or Arbitrum) |
| M4-5 | Node operator incentive program: 50-node launch cohort |
| M4-6 | Governance module: SMTZ token voting on protocol parameters |
| M4-7 | IPNI full integration: semantiz as IPNI provider |
| M4-8 | Cross-chain reward bridging (optional) |
| M4-9 | Research: onion routing for query privacy (see Section 13) |
| M4-10 | 1 billion document milestone: benchmark and publish |

---

## 12. Directory Structure

```
semantiz/                           # Repository root
├── .github/
│   ├── workflows/
│   │   ├── ci.yml                  # Lint, type-check, unit tests on every PR
│   │   ├── integration.yml         # Integration tests on merge to main
│   │   └── deploy.yml              # Deploy to testnet/mainnet
│   └── CODEOWNERS
│
├── packages/                       # Python packages (uv workspaces)
│   ├── semantiz-core/              # Shared types, config, logging, errors
│   │   ├── pyproject.toml
│   │   └── src/semantiz_core/
│   │       ├── __init__.py
│   │       ├── config.py           # Pydantic settings from env/YAML
│   │       ├── models.py           # Shared dataclasses (ContentRecord, etc.)
│   │       ├── errors.py           # Custom exception hierarchy
│   │       └── logging.py          # Structured JSON logging (structlog)
│   │
│   ├── semantiz-crawler/           # CrawlerNode implementation
│   │   ├── pyproject.toml
│   │   └── src/semantiz_crawler/
│   │       ├── crawler.py          # CrawlerNode class
│   │       ├── frontier.py         # URL frontier (SQLite-backed priority queue)
│   │       ├── extractor.py        # Trafilatura + BeautifulSoup4 extraction
│   │       ├── robots.py           # robots.txt fetching + caching
│   │       └── tests/
│   │
│   ├── semantiz-embedding/         # EmbeddingService + model backends
│   │   ├── pyproject.toml
│   │   └── src/semantiz_embedding/
│   │       ├── service.py          # EmbeddingService orchestrator
│   │       ├── backends/
│   │       │   ├── sbert.py        # sentence-transformers backend
│   │       │   ├── openai.py       # OpenAI Ada-002 backend
│   │       │   └── bge.py          # BGE backend
│   │       ├── cache.py            # LRU embedding cache (SQLite)
│   │       └── tests/
│   │
│   ├── semantiz-index/             # SemanticIndex (FAISS + SQLite)
│   │   ├── pyproject.toml
│   │   └── src/semantiz_index/
│   │       ├── index.py            # SemanticIndex class
│   │       ├── faiss_wrapper.py    # FAISS operations + ID mapping
│   │       ├── sqlite_store.py     # IndexEntry SQLite persistence
│   │       ├── rebuild.py          # Periodic rebuild logic
│   │       └── tests/
│   │
│   ├── semantiz-storage/           # IPFSStorageAdapter + Filecoin deals
│   │   ├── pyproject.toml
│   │   └── src/semantiz_storage/
│   │       ├── ipfs_adapter.py     # Kubo HTTP API wrapper
│   │       ├── filecoin.py         # Filecoin deal negotiation
│   │       ├── ipni.py             # IPNI advertisement
│   │       └── tests/
│   │
│   ├── semantiz-query/             # QueryProcessor + ranking
│   │   ├── pyproject.toml
│   │   └── src/semantiz_query/
│   │       ├── processor.py        # QueryProcessor orchestrator
│   │       ├── ranking.py          # Composite ranking formula
│   │       ├── cache.py            # Result LRU cache
│   │       └── tests/
│   │
│   ├── semantiz-network/           # SON + DHTRouter (Python side)
│   │   ├── pyproject.toml
│   │   └── src/semantiz_network/
│   │       ├── son.py              # SemanticOverlayNetwork
│   │       ├── dht_router.py       # Python wrapper over Go DHT daemon
│   │       ├── gossip.py           # Gossip message handling
│   │       └── tests/
│   │
│   ├── semantiz-consensus/         # PoC proof generation, validator client
│   │   ├── pyproject.toml
│   │   └── src/semantiz_consensus/
│   │       ├── proof_generator.py  # ContributionProof builder
│   │       ├── validator.py        # Validator client (IPFS sampling + on-chain votes)
│   │       ├── merkle.py           # Merkle tree implementation
│   │       ├── reputation.py       # ReputationSystem
│   │       └── tests/
│   │
│   └── semantiz-api/               # FastAPI SearchAPI
│       ├── pyproject.toml
│       └── src/semantiz_api/
│           ├── main.py             # FastAPI app factory
│           ├── routers/
│           │   ├── search.py       # POST /api/v1/search
│           │   ├── index.py        # GET /stats, POST /submit
│           │   ├── nodes.py        # GET /nodes, GET /node/{id}
│           │   └── crawl.py        # POST /api/v1/crawl
│           ├── middleware/
│           │   ├── auth.py         # API key verification
│           │   ├── rate_limit.py   # Redis sliding window rate limiter
│           │   └── metrics.py      # Prometheus instrumentation
│           ├── schemas.py          # Pydantic request/response schemas
│           └── tests/
│
├── services/                       # Deployable services
│   ├── node/                       # Full semantiz node (entrypoint)
│   │   ├── Dockerfile
│   │   ├── docker-compose.yml      # node + IPFS + redis
│   │   ├── node.py                 # Main node process (orchestrates all packages)
│   │   └── config/
│   │       ├── default.yaml        # Default node configuration
│   │       └── testnet.yaml        # Testnet-specific overrides
│   │
│   └── p2p-daemon/                 # Go libp2p daemon
│       ├── Dockerfile
│       ├── go.mod
│       ├── go.sum
│       ├── main.go                 # Daemon entrypoint
│       ├── dht/                    # Kademlia DHT logic
│       ├── gossip/                 # Plumtree gossip implementation
│       ├── proto/                  # Generated protobuf Go code
│       └── tests/
│
├── contracts/                      # Solidity smart contracts (Foundry)
│   ├── foundry.toml
│   ├── src/
│   │   ├── ProofOfContribution.sol
│   │   ├── NodeRegistry.sol
│   │   ├── RewardDistributor.sol
│   │   ├── SMTZToken.sol           # ERC-20 token
│   │   └── interfaces/
│   │       ├── IProofOfContribution.sol
│   │       └── INodeRegistry.sol
│   ├── test/
│   │   ├── ProofOfContribution.t.sol
│   │   ├── RewardDistributor.t.sol
│   │   └── NodeRegistry.t.sol
│   ├── script/
│   │   ├── Deploy.s.sol            # Deployment script
│   │   └── Verify.s.sol            # Verification on Etherscan
│   └── deployments/
│       ├── sepolia.json            # Deployed contract addresses (testnet)
│       └── mainnet.json            # Deployed contract addresses (mainnet)
│
├── frontend/                       # React Web UI
│   ├── package.json
│   ├── vite.config.ts
│   ├── tailwind.config.ts
│   ├── src/
│   │   ├── main.tsx
│   │   ├── App.tsx
│   │   ├── pages/
│   │   │   ├── Search.tsx
│   │   │   ├── Dashboard.tsx
│   │   │   ├── NodeManagement.tsx
│   │   │   └── Settings.tsx
│   │   ├── components/
│   │   │   ├── SearchBar.tsx
│   │   │   ├── ResultCard.tsx
│   │   │   ├── NodeStats.tsx
│   │   │   └── RewardHistory.tsx
│   │   ├── hooks/
│   │   │   ├── useSearch.ts
│   │   │   └── useNodeStats.ts
│   │   └── api/
│   │       └── client.ts           # Typed API client (generated from OpenAPI)
│   └── tests/
│
├── extension/                      # Browser extension (MV3)
│   ├── manifest.json
│   ├── src/
│   │   ├── background/
│   │   │   └── service-worker.ts   # Tab listener, URL submission
│   │   ├── content/
│   │   │   └── content-script.ts   # Reading time, scroll depth capture
│   │   ├── popup/
│   │   │   ├── Popup.tsx           # React popup
│   │   │   └── stats.ts
│   │   └── shared/
│   │       ├── api.ts              # Node API client
│   │       └── config.ts
│   └── tests/
│
├── proto/                          # Shared protobuf definitions
│   └── semantiz/
│       └── v1/
│           ├── messages.proto      # All message types
│           └── services.proto      # gRPC service definitions
│
├── scripts/                        # Developer utility scripts
│   ├── seed-urls.py                # Seed crawl frontier with test URLs
│   ├── benchmark-search.py         # Run query latency benchmarks
│   ├── generate-proto.sh           # Regenerate Python + Go + TS protobuf
│   └── testnet-setup.sh            # Launch 5-node local testnet
│
├── docs/                           # Technical documentation
│   ├── architecture/               # Architecture Decision Records (ADRs)
│   ├── api/                        # Generated OpenAPI spec
│   └── node-operator/              # Node setup guide
│
├── docker-compose.dev.yml          # Full dev environment
├── Makefile                        # Common developer commands
├── BLUEPRINT.md                    # This document
└── README.md
```

---

## 13. Open Research Questions

The following problems are unresolved and require dedicated R&D effort. They represent the frontier of decentralized semantic search research.

### 13.1 Query Privacy via Semantic Routing

**Problem:** In the current design, a node receives the raw query embedding vector when a peer queries it. An adversarial node can accumulate query vectors and over time infer user interests or identity. Pure semantic routing fundamentally leaks information.

**Approaches to investigate:**
- **Onion routing over libp2p:** Route queries through 3-hop circuits using HORNET-style onion encryption. Each hop sees only the next hop address. Challenge: latency increase of ~200ms per hop.
- **Differential privacy on embeddings:** Add calibrated Gaussian noise to query vectors before transmission. Trade-off: noise degrades retrieval recall.
- **Private Information Retrieval (PIR):** PIR protocols allow a client to retrieve a database entry without the server knowing which entry was retrieved. Applying this to FAISS search is an active research area with ~1000x computational overhead.
- **Secure Multi-Party Computation (MPC) for vector search:** HNSW-over-MPC proposals exist but are impractical at scale today.

**Required reading:** Privado (Facebook, 2021), SEALPIR (Microsoft), Oblivious RAM (ORAM) literature.

### 13.2 Dynamic Index Sharding

**Problem:** As the network grows to billions of documents, no single FAISS index fits in RAM. The SON provides logical sharding but doesn't guarantee coverage: a rare topic may have no specialized node.

**Approaches to investigate:**
- **Balanced semantic k-means partitioning:** Periodically run distributed k-means over all centroids to define balanced topic partitions. Each node is assigned to a partition.
- **Consistent hashing on embedding space:** Map the vector space to a consistent hash ring. Nodes are responsible for document ranges on the ring.
- **Replication factor per topic:** Measure query load per topic; replicate high-demand topic shards across multiple nodes.

### 13.3 Sybil-Resistant Validator Selection Without PoS

**Problem:** Current validator selection is PoS-based (stake = influence). This favors wealthy nodes and recreates plutocracy. Alternative: PoW-based selection wastes energy. Social trust graphs are hard to bootstrap.

**Approaches to investigate:**
- **VRF-based random selection** with equal probability per node (one-node-one-vote), proof-of-personhood for Sybil resistance (using Worldcoin, Proof of Humanity, or BrightID)
- **Reputation-weighted selection** with diminishing returns: `P(selected) ∝ log(reputation + 1)` instead of linear weight
- **Federated Byzantine Agreement (FBA):** Nodes declare quorum slices (trusted subsets). Consensus emerges from overlapping slices (Stellar-style).

### 13.4 Cross-Model Embedding Compatibility

**Problem:** If node A indexes documents with SBERT-768 and node B uses Ada-002-1536, their vectors live in different metric spaces and are not directly comparable. The network requires a common embedding space.

**Approaches to investigate:**
- **Universal projection layer:** Train a linear projection from each model's output space into a shared 768-dim space. Requires labeled pairs of same-content embeddings from both models.
- **Canonical model mandate:** Require all nodes to use a specific open model (e.g., `all-mpnet-base-v2`). Simple but limits future model upgrades and locks in current-generation quality.
- **Late fusion:** Each node searches its own index with its own embeddings, returns results. Originator re-ranks using only similarity scores (not raw vectors). Works if scores are well-calibrated.

### 13.5 Freshness vs. Storage Incentives

**Problem:** PoC rewards freshness, incentivizing nodes to constantly re-crawl and re-index. But this conflicts with IPFS's content-addressed, immutable model and wastes resources on unchanged content.

**Approach:** Define "freshness" as the delta between the current canonical version of a URL and the stored version. Nodes that efficiently track URL changes (via HTTP ETags, Last-Modified headers, and sitemap `<lastmod>`) without redundant full re-crawls should be rewarded more.

### 13.6 Adversarial Embedding Attacks

**Problem:** An adversary could craft documents specifically designed to be retrieved by queries they should not match (adversarial examples in embedding space), poisoning results for specific queries.

**Approaches to investigate:**
- Perceptual hashing of near-duplicate documents to detect embedding injection
- Ensemble embeddings: require N/2 models to agree on embedding distance before a result is surfaced
- Adversarial training of embedding model with representative attack samples

### 13.7 Decentralized Content Moderation

**Problem:** Without a central authority, how do we prevent indexing of illegal content (CSAM, terrorism, fraud)? Pure decentralization creates liability and harm.

**Approaches to investigate:**
- **Distributed hash lists:** Cryptographic hashes of known harmful content CIDs, distributed as a protocol-level blocklist. Nodes that serve blocked CIDs are slashed.
- **Reputation-weighted flagging:** Any N nodes can flag a CID; if reputation-weighted votes exceed threshold, CID is quarantined pending review.
- **Legal shield structure:** Foundation operates as a non-profit with DMCA agent, provides legal safe harbor for node operators following the blocklist protocol.

---

## 14. Glossary

| Term | Definition |
|---|---|
| **Ada-002** | OpenAI's `text-embedding-ada-002` model that produces 1536-dimensional embedding vectors. Closed-source, accessed via API. |
| **BGE** | BAAI General Embedding. An open-weight embedding model family from the Beijing Academy of AI. `bge-large-en-v1.5` produces 1024-dim vectors and is competitive with Ada-002. |
| **Bitswap** | The IPFS data exchange protocol. Nodes request blocks by CID; peers with the block send it. |
| **CID** | Content Identifier. An IPFS self-describing hash of content. CIDv1 uses base32 encoding and SHA-256 hashing by default. The same content always produces the same CID regardless of who stored it. |
| **Centroid** | The mean embedding vector of all documents in a node's local index. Represents the "average topic" of that node's content. Used by the SON for semantic routing. |
| **Contribution Score** | A composite score in [0,1] computed for each node each epoch. Weighted sum of Coverage, Freshness, Accuracy, and Availability scores. Determines reward size. |
| **Cosine Similarity** | Dot product of two L2-normalized vectors. Returns 1.0 for identical direction, 0.0 for perpendicular, -1.0 for opposite. Primary relevance metric for embedding search. |
| **DHT** | Distributed Hash Table. A key-value store distributed across many nodes. semantiz uses Kademlia DHT for peer discovery and routing table maintenance. |
| **Eclipse Attack** | An attack where an adversary surrounds a target node with malicious peers, blocking all honest peer connections and controlling the target's view of the network. |
| **Epoch** | A fixed time window (24 hours in semantiz) after which contribution proofs are submitted and rewards are distributed. |
| **FAISS** | Facebook AI Similarity Search. An open-source library for efficient nearest-neighbor search in high-dimensional vector spaces. |
| **Filecoin** | A decentralized storage network built on IPFS that adds economic incentives (storage deals) for long-term data persistence. |
| **Gossip Protocol** | A peer-to-peer communication pattern where nodes forward messages to a subset of their peers, causing information to propagate exponentially fast. semantiz uses Plumtree gossip. |
| **IVFFlat / IVFPQ** | FAISS index types. IVF (Inverted File) partitions vectors into clusters for approximate search. PQ (Product Quantization) compresses vectors for memory efficiency. |
| **IPFS** | InterPlanetary File System. A content-addressed, peer-to-peer hypermedia protocol. Files are identified by their content hash (CID), not location. |
| **IPNI** | InterPlanetary Network Indexer. A distributed system for indexing which IPFS/Filecoin providers store which CIDs. Allows efficient CID discovery without querying all nodes. |
| **Kademlia** | A DHT algorithm where nodes and keys share a 160-bit key space. Lookup complexity is O(log N). Distance is measured by XOR of keys. |
| **L2 Normalization** | Dividing a vector by its Euclidean norm so it has unit length. After L2 normalization, inner product equals cosine similarity. |
| **libp2p** | A modular, protocol-agnostic P2P networking library used by IPFS, Ethereum 2.0, and many other systems. semantiz uses it for all node-to-node communication. |
| **Merkle Tree** | A binary tree where each leaf is a hash of data, and each internal node is a hash of its children. The root hash commits to all leaves. Used in ContributionProof to prove index content without revealing the full CID list. |
| **mDNS** | Multicast DNS. A zero-configuration protocol for discovering services on a local network. Used by semantiz for LAN peer discovery. |
| **Noise Protocol** | A cryptographic framework for building secure channel protocols. semantiz uses the Noise XX handshake for mutual authentication between peers. |
| **OWI** | Open Web Index. An academic/non-profit initiative to build a publicly accessible, auditable web index as an alternative to proprietary search engine indices. |
| **PoC** | Proof of Contribution. semantiz's consensus mechanism that rewards nodes for verified indexing and serving contributions. Analogous to Proof of Work or Proof of Stake but for search indexing. |
| **Plumtree** | A gossip tree algorithm that combines eager push (immediate forwarding) and lazy push (metadata-only, then pull-on-demand) to achieve efficient broadcast with low redundancy. |
| **QUIC** | A UDP-based transport protocol with built-in TLS, multiplexing, and faster connection establishment than TCP. Used as an alternative transport in libp2p. |
| **Reputation Score** | A floating-point score in [0.0, 1.0] for each node, derived from PoC success rate, uptime, and query-serve quality. Influences query result ranking and SON neighbor selection. |
| **SBERT** | Sentence-BERT. A modification of the BERT architecture fine-tuned to produce semantically meaningful sentence-level embeddings via siamese and triplet network training. Key library: `sentence-transformers`. |
| **Semantic Overlay Network (SON)** | A P2P overlay topology where nodes with similar index content maintain direct connections, enabling efficient semantic query routing. |
| **Slashing** | Destruction of a node's staked tokens as punishment for provably malicious or negligent behavior. Discourages attacks by making them economically costly. |
| **SMTZ** | The native ERC-20 utility token of the semantiz protocol. Used for staking (NodeRegistry), rewards (RewardDistributor), and governance (Phase 4). |
| **Sybil Attack** | An attack where one entity creates many fake identities to gain disproportionate influence in a decentralized network. |
| **Testnet** | A blockchain network identical in design to mainnet but using worthless test tokens. semantiz uses Sepolia (Ethereum testnet) during development. |
| **VRF** | Verifiable Random Function. A cryptographic function that produces a pseudo-random output with a proof that the output was computed correctly from the input. Used for unbiasable on-chain randomness. |
| **Web4** | An informal term for the next generation of the web, characterized by full decentralization (P2P), user-owned data, AI-native interfaces, and economic protocols (token incentives). Distinct from Web3 in emphasizing open infrastructure over financialization. |
| **yamux** | Yet Another Multiplexer. A stream multiplexing protocol used by libp2p over TCP connections. Allows multiple logical streams over a single TCP connection. |

---

*End of BLUEPRINT.md*

---

**Document Control:**
- Version 0.1.0-draft: Initial architecture document, July 2026
- Next review: After Phase 1 completion
- Maintained by: Architecture Team
- PRs to this document require 2 engineering approvals
