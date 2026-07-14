# Research Foundation: semantiz

Deep technical analysis of the five academic references that underpin the semantiz architecture.
Each section covers: core approach, key algorithms and data structures, benchmarks, architectural
insights for semantiz, and known limitations.

---

## Table of Contents

1. [Semantica — arXiv:2502.10151](#1-semantica--arxiv25021051)
2. [DS⁴ — Distributed Social and Semantic Search](#2-ds--distributed-social-and-semantic-search)
3. [Open Web Index (OWI) / OpenWebSearch.EU](#3-open-web-index-owi--openwebsearcheu)
4. [IPNI — InterPlanetary Network Indexer](#4-ipni--interplanetary-network-indexer)
5. [In-Browser Agentic Web](#5-in-browser-agentic-web)
6. [Cross-Reference Synthesis](#6-cross-reference-synthesis)

---

## 1. Semantica — arXiv:2502.10151

**Full title:** Decentralized Search Using a LLM-Guided Semantic Tree Overlay  
**Authors:** Petru Neague, Quinten Stokkink (Delft University of Technology); Naman Goel (University of Oxford); Johan Pouwelse (Delft University of Technology)  
**Published:** February 14, 2025  
**arXiv:** https://arxiv.org/abs/2502.10151  
**Code:** https://github.com/pneague/Semantica  

### Core Technical Approach

Semantica is a fully distributed peer-to-peer search algorithm that uses pre-trained LLM embeddings
to construct a Semantic Overlay Network (SON). The central insight is that document semantics have
**predictive capacity**: nodes sharing similar document interests can be co-located in an
embedding-space tree, making semantic queries resolvable by immediate neighbors without global
traversal.

The system is classified along four axes in the paper — all four must be satisfied simultaneously,
and no prior work achieved this:

| Property | Semantica | DSI | De-DSI | Chord DHT | Graph Diffusion |
|---|---|---|---|---|---|
| Semantic Search (embedding-based) | Yes | Yes | Yes | No | Partial |
| Predictive Cache (topology encodes proximity) | Yes | No | No | No | No |
| Distributed (fully P2P) | Yes | No | Partial | Yes | Yes |
| No Training Required | Yes | No | No | Yes | Yes |

**Dataset:** AOL4PS — 187,521 websites, 12,907 users, 1,339,101 queries, 3 months (2006). After
filtering: 6,978 active users. Document embeddings use BERT (768-dimensional).

### Key Algorithms

#### Algorithm 1: Dynamic Tree Construction with Soft Clustering

**Parameters:**
- `M = 50` — max users per leaf before split
- `k = 2` — fixed k-means split factor
- `Δ` — soft-clustering threshold for cloning users into multiple branches

**Procedure:**
```
for each user u_i (random order):
    compute U_i = mean(BERT(d) for d in D_i)   # user embedding
    traverse tree root → leaf by centroid distance
    at each split-node with children n1, n2:
        d1 = ||U_i - C_n1||₂
        d2 = ||U_i - C_n2||₂
        if |d1 - d2| < Δ: clone u_i into BOTH subtrees
        else: route to closer child
    assign u_i to leaf
    if |leaf.users| > M: k-means split → two child nodes
```

**Clone distribution at Δ = 0.001 (practical sweet spot):**

| Δ | Median clones | Mean clones |
|---|---|---|
| 0 | 1 | 1.000 |
| 1e-4 | 1 | 1.026 |
| 1e-3 | 1 | 1.322 |
| 3e-3 | 2 | 2.756 |
| 5e-3 | 5 | 7.450 |

#### Algorithm 2: Semantic Neighbor Discovery via Expansion Rounds

Each node maintains two lists:
- `known-users` — all discovered peers (~208 mean degree)
- `closest-users` — top-50 by cosine similarity to the node's own embedding

**Initialization:** BFS on nearby leaf nodes until `n_cc` peers gathered. Compute cosine similarity
to self. Store top `n_cu` in closest-users.

**Expansion rounds (up to `r_max`):** Each round, every node queries a random peer from
closest-users, asking for a closer peer not yet known. If closer: add to known-users, recompute
closest-users. O(N) messages per round.

#### Algorithm 3: Chain-Hop Query Processing

```
query Q → embedding vector
hop 0: initiator u_q searches local documents
if not found:
    u_next = argmax_{v ∈ closest-users} cosine_sim(Q, U_v)
    forward to u_next
repeat until found or hop limit ℓ reached
```

Complexity: O(ℓ) messages per query; O(k + d) computation per hop.

### Performance Benchmarks

**Experiment 1 — Closest User Recall (top-50 peer identification):**
- Semantica (Δ=0.001, 10 expansion rounds): **recall ≈ 35**
- Random baseline (Barabási–Albert): **recall ≈ 5**
- **Result: 10× more semantically similar users discovered**

**Experiment 2 — Document Retrieval at 2 hops:**
- Semantica (Δ=0.003, 10 rounds): **12.75%** of required documents
- All baselines (Graph Diffusion, random): **< 6%**
- **Result: 2× more relevant documents retrieved at same network cost**

**Experiment 3 — Minimum hop distance:**
Semantica shifts many documents from distance >1 to distance =1 (immediate neighbor). Trade-off:
small fraction become unreachable in tight clusters (disconnected sub-graphs).

### Architectural Insights for semantiz

1. **Binary k-means trie as the primary SON topology.** Insert cost O(N log N). Maps directly to
   hierarchical topic clustering.
2. **Soft clustering with Δ ≈ 0.001.** Multi-topic nodes register in multiple leaf clusters.
   Practical exponential cap at this threshold for 768-dim BERT embeddings.
3. **Two-list neighbor architecture.** `known-users` for exploration, `closest-users` for routing.
   Per-hop cost bounded to O(k) cosine comparisons.
4. **Chain-hopping as primary retrieval primitive.** Simpler and better than diffusion-based
   alternatives for short queries (ℓ ≤ 50 hops).
5. **Custodian architecture for decentralized split-nodes.** One peer stores centroids per
   split-node; chain from root to leaves. semantiz should replicate custodians (primary +
   secondary) against churn.
6. **Hybrid topology (semantic + 5% random long-range links).** Prevents disconnected sub-graphs
   while preserving semantic locality.
7. **Ranking at the query initiator.** Results aggregated and scored locally — correct design for
   a privacy-preserving system.

### Limitations

1. No tree rebalancing under node churn — explicitly left as future work
2. Tight semantic clusters can disconnect sub-graphs (small-world links needed)
3. Custodian failure unhandled — backup election needed
4. Static embedding model assumed — incompatible embeddings across versions
5. No Sybil resistance
6. No privacy analysis — user embeddings expose browsing interests if shared in the clear
7. All experiments on 6,978 users; millions-scale unvalidated

---

## 2. DS⁴ — Distributed Social and Semantic Search

**Full title:** DS⁴: A Distributed Social and Semantic Search System  
**Authors:** B. Kontominas, P. Raftopoulou, C. Tryfonopoulos, E. Petrakis  
**Published:** ECIR 2013, Springer LNCS 7814, pp. 820–821  
**DOI:** https://link.springer.com/chapter/10.1007/978-3-642-36973-5_96  

*Note: Full paper is behind a Springer paywall. The project page (users.uop.gr/~petra/DS4/) is
offline as of July 2026. Analysis is based on the abstract, citing works, and ECIR 2013 context.*

### Core Technical Approach

DS⁴ is a decentralized, privacy-aware search system that combines two orthogonal peer organization
dimensions simultaneously:

1. **Social routing** — queries propagate along the social graph (friend-of-friend links)
2. **Semantic routing** — queries propagate toward nodes with high semantic similarity to the query

Both channels run in parallel. Results are merged at the query initiator. The system requires no
centralized component and is described as scalable, adaptive, and general across content types.

The dual-facet design addresses a fundamental coverage gap in pure semantic search: a node that
has the desired document but is semantically distant from the query can still be reached via the
social routing path, if it is socially connected to the query issuer.

### Key Design Properties

- **Privacy by design:** No shared global index; each node controls its own content
- **Adaptive:** Cluster assignments update as user content evolves
- **Automatic:** No manual configuration for cluster formation
- **Scalable:** Horizontal scaling, no centralized coordinator

### Architectural Insights for semantiz

1. **Dual-facet routing as optional extension.** A social-graph layer over the SON topology
   provides a fallback for semantically distant but socially connected content holders.
2. **Social connections as trust signals.** Documents vouched for by trusted social connections
   receive a ranking boost — analogous to PageRank derived from social proximity.
3. **Privacy through local storage.** No shared index means no single point of privacy violation.
   This is the core privacy guarantee of semantiz.
4. **Adaptive cluster membership.** As node content evolves, periodic re-computation of the node
   embedding and potential re-insertion into the SON tree is required.

### Limitations

1. No large-scale empirical validation (2-page ECIR demo paper)
2. Dual-path messaging doubles network overhead — priority routing preferred
3. Social graph cold-start problem for new users
4. 2013 content representations (TF-IDF) — would benefit from modern LLM embeddings
5. Project unmaintained since at least 2023

---

## 3. Open Web Index (OWI) / OpenWebSearch.EU

**Project:** OpenWebSearch.EU  
**EU Grant:** #101070014, €8.5M over 42 months  
**Partners:** 14 institutions, 7 European countries  
**Website:** https://openwebsearch.eu/  
**Dashboard:** https://dashboard.ows.eu/  

### Core Technical Approach

OpenWebSearch.EU builds an open, sovereign, non-commercial European web index as public research
infrastructure. The project produces:

1. **Open Web Index (OWI):** Large-scale web crawl corpus in WARC/Parquet formats
2. **OWSAI:** Extensible search/analysis platform on top of OWI

OWI is infrastructure, not a search engine. Third parties build applications on top of it.
The design philosophy: European digital sovereignty requires a publicly controlled web corpus
not subject to commercial API terms or opaque ranking.

### Scale Metrics (dashboard, July 2026)

| Metric | Value |
|---|---|
| URLs crawled | 9.14 billion |
| Hosts | 28 million |
| Languages | 185 |
| Total raw data | 1,514.56 TiB |
| Crawl rate | ~1 TiB/day |
| Structured index size | 35.09 TiB |
| Public datasets | 1,875 |

### Key Technical Components

**Data format:** WARC (ISO 28500) for raw crawl; Parquet + DuckDB for structured querying.

**OWILIX / owi-cli:**
- `pull`: retrieve OWI shards from central store
- `push`: contribute community-generated index data
- Backend: iRODS for parallel distributed data management
- Query: DuckDB + Parquet locally

**OWI-shards:** Thematic partitions of the index with metadata:
- Geo-coordinates, topics, genre classification per document

**Resilipipe GPU (Kassel + Passau):** GPU-accelerated embedding pipeline on OWI content.
Content quality classifiers:
- Ad detector: validation F1 = **0.82**
- Phishing classifier: validation F1 = **0.84**
- LLM-generated content detector: validation F1 = **0.94**

**MosaicRAG (Graz TU):** RAG pipeline on top of OWI — summarisation, re-ranking, chat.

**CIFF toolkit (v0.2.2):** Common Index File Format for inverted index interchange, compatible
with Anserini, Terrier, PISA.

### Architectural Insights for semantiz

1. **WARC + Parquet two-layer corpus architecture.** Raw archive (WARC) + query-ready derived
   index (Parquet with embedding vectors). semantiz adopts this pattern exactly.
2. **Thematic shards = SON leaf clusters.** OWI's index slices map directly to leaf nodes in
   the Semantica tree. Each leaf cluster = one thematic shard.
3. **OWI as bootstrapping corpus.** 9.14B URLs and 35 TiB of indexed content provide an
   immediate bootstrapping corpus for semantiz Phase 1 without cold-starting from zero.
4. **Resilipipe classifiers, reusable out-of-box.** Content quality filtering (ad detection,
   phishing, LLM-generated content) can be integrated directly under CC BY 4.0.
5. **CIFF for BM25 interoperability.** Lexical index baseline using Anserini/Terrier for hybrid
   retrieval (BM25 + dense vector) without building from scratch.
6. **iRODS for large shard distribution.** Semantiz inter-node shard distribution can adopt
   iRODS-style distributed data fabric for large corpora.
7. **RAG pipeline over vector index.** MosaicRAG pattern: search → re-rank → LLM synthesis.
   semantiz query tier can optionally return synthesized answers rather than raw URLs.

### Limitations

1. Access restricted to research and development only (not a public search product)
2. No semantic (vector) search layer — OWI is keyword-indexed only
3. Centralized crawl infrastructure and storage (not decentralized)
4. No real-time indexing (~1 TiB/day means significant freshness lag)
5. No decentralized contribution model (push is centrally mediated)
6. European legal jurisdiction; cross-jurisdictional content governance unresolved

---

## 4. IPNI — InterPlanetary Network Indexer

**Specification:** https://github.com/ipni/specs/blob/main/IPNI.md  
**IPFS Concepts:** https://docs.ipfs.tech/concepts/ipni/  
**Public Indexer:** https://cid.contact/  
**Delegated Routing Proxy:** https://delegated-ipfs.dev/routing/v1  

### Core Technical Approach

IPNI is an alternate content routing system for IPFS and Filecoin, designed to overcome the
scalability limits of the Kademlia DHT for large-volume content providers. The DHT requires
re-announcing every CID every 24 hours; for Filecoin providers with billions of CIDs this is
operationally infeasible. IPNI replaces this with a **persistent, append-only advertisement chain**
that indexers process incrementally.

**Three-actor model:**
1. **Content Providers** — publish advertisement chains announcing their content
2. **IPNI Nodes (Indexers)** — ingest chains, maintain `multihash → provider` mappings, serve queries
3. **Retrieval Clients** — query indexers to discover providers, then fetch directly from providers

### Key Data Structures (IPLD Schemas)

#### Advertisement

```
type Advertisement struct {
    PreviousID  optional Link    # CID of prior ad; absent on genesis
    Provider    String           # libp2p peer.ID
    Addresses   [String]         # multiaddrs (always overwritten, not diff)
    Signature   Bytes            # signed by provider's libp2p key
    Entries     Link             # EntryChunk linked list or HAMT
    ContextID   Bytes            # logical namespace for updates/deletes
    Metadata    Bytes            # varint-prefixed protocol encoding; < 100 bytes
    IsRm        Bool             # deletion flag
    ExtendedProvider optional ExtendedProvider
}
```

**ContextID semantics:**
- Same ContextID + new entries → additive
- Same ContextID + new metadata → metadata update for all CIDs in that namespace
- Same ContextID + `IsRm=true` → remove all CIDs in that namespace

#### EntryChunk

```
type EntryChunk struct {
    Entries [Bytes]          # list of multihashes
    Next    optional Link    # next chunk in linked list
}
```

Constraints: each chunk < 4 MB; max 400 chunks per advertisement; max ~40 million multihashes
per advertisement.

#### Metadata Protocol Encoding

| Protocol | uvarint | Transport |
|---|---|---|
| Bitswap | 0x0900 | transport-bitswap |
| Filecoin Graphsync | 0x0910 | transport-graphsync-filecoinv1 |
| HTTP IPFS Trustless Gateway | 0x0920 | transport-ipfs-gateway-http |
| HTTP Filecoin Piece | 0x0930 | transport-filecoin-piece-http |

### HTTP Transport API

**Advertisement serving:**
```
GET  /ipni/v1/ad/{CID}    # fetch advertisement or entries by CID (immutable, CDN-cacheable)
GET  /ipni/v1/ad/head     # fetch SignedHead (current chain tip)
```

**Query interface:**
```
GET  /cid/{cid}                              # lookup by CID
GET  /multihash/{multihash}                  # lookup by multihash
GET  /routing/v1/providers/{cid}             # Delegated Routing V1 (standard)
```

Response format: `application/json` (full FindResponse) or `application/x-ndjson` (streaming).

### Ingestion Algorithm

```
1. Indexer receives Announce Message (via Gossipsub or HTTP PUT /announce)
2. Fetches advertisement at announced CID from publisher
3. Walks chain backward via PreviousID links
4. Stops at previously-seen advertisement OR chain genesis
5. Processes forward (oldest → newest)
6. Stores multihash → provider mappings
7. Content-addressed immutability → efficient incremental sync
```

### Performance vs DHT

| Property | IPNI | Amino DHT |
|---|---|---|
| Re-announcement | Once (persistent chain) | Every 24 hours |
| Large-volume publisher scalability | High | Constrained |
| CDN-cacheable objects | Yes (immutable CIDs) | No |
| Real-time peer discovery | No | Yes |

### Architectural Insights for semantiz

1. **Advertisement chain as the decentralized index update protocol.** Each semantiz node
   maintains its own advertisement chain for index changes. Immutable, content-addressed,
   append-only. Indexer nodes ingest incrementally.
2. **ContextID = semantic cluster namespace.** A leaf node in the Semantica tree = one ContextID.
   Metadata encodes the cluster centroid CID + embedding model identifier. All CIDs in that leaf
   are published under this ContextID.
3. **Separate Publisher and Provider roles.** A peer can index and announce content hosted by
   others. semantiz should adopt: semantic indexer nodes announce; storage nodes serve.
4. **Delegated Routing V1 as the standard query API.** `GET /routing/v1/providers/{cid}` provides
   ecosystem compatibility with the broader IPFS/Filecoin stack.
5. **Gossipsub topic for announcement propagation.** Use `/semantiz/ingest/mainnet` as the
   production gossipsub topic, matching IPNI's pattern exactly.
6. **Separate ingest port from query port.** Ingest (authenticated, writable) on TCP port N;
   query (public, read-only) on TCP port N+1.
7. **HAMT for large entry sets.** When a node's content set exceeds ~10K CIDs, switch from
   EntryChunk linked lists to HAMT for efficient random access.
8. **CDN-cacheable advertisement objects.** Immutable CID-addressed ads are safe to cache
   at CDN edges — near-CDN latency for frequently-accessed index metadata.

### Limitations

1. Not suitable for hot/real-time content (optimized for static large-volume, like Filecoin cold storage)
2. Opt-in only — content not explicitly advertised is invisible
3. Cold re-sync cost for nodes that fall significantly behind the chain
4. CBOR encoding edge cases for ExtraData and OrigPeer (spec TODOs)
5. Provider validity policies are non-normative — inconsistent across indexers
6. No semantic search capability — purely a multihash → provider lookup system
7. Public cid.contact indexer is a potential centralization point

---

## 5. In-Browser Agentic Web

**Full title:** In-Browser Agentic Web: A Decentralized Approach to Information Access  
**Venue:** Open Search Symposium 2025 (OSSYM 2025)  
**Zenodo:** https://zenodo.org/records/17229737  
**Authors:** Saber Zerhoudi, Michael Granitzer (University of Passau)  
**Funding:** OpenWebSearch.EU (#101070014)  

*Note: PDF binary extraction failed in the analysis environment. The following synthesizes from
Zenodo metadata, OSSYM 2025 program context, and the authors' research history at University of
Passau.*

### Core Technical Approach

The paper proposes running AI-powered search agents **directly within the browser**, eliminating
centralized search servers. The browser becomes the full execution environment:

- Crawling (via Service Worker fetch interception)
- Embedding (ONNX Runtime Web / WebGPU / Transformers.js)
- Indexing (OPFS-backed HNSW vector database)
- Querying (local ANN search + optional WebRTC peer fan-out)

Every browser instance becomes an autonomous semantiz node. Decentralization is achieved not
through dedicated server infrastructure, but through the aggregate of millions of browser agents.

### Key Technical Components

**In-browser embedding pipeline:**
- **ONNX Runtime Web** — quantized sentence transformers (e.g., all-MiniLM-L6-v2, ~22M params,
  384-dim output) compiled to ONNX, running as a Web Worker
- **WebGPU** (Chrome 113+) — GPU-accelerated inference without WebGL overhead
- **Transformers.js** (HuggingFace) — JavaScript library for in-browser ONNX models

Expected embedding latency:
- CPU (MiniLM): ~50–200ms per document
- WebGPU: ~5–20ms per document

**In-browser vector index:**
- **IndexedDB** — persistent key-value store; suitable for pre-computed embeddings keyed by CID
- **OPFS (Origin Private File System)** — higher-performance browser filesystem, available all
  major browsers since 2023; suitable for large HNSW indexes
- **Flat cosine scan** — viable for < 10,000 vectors in WASM (~10ms)
- **HNSW in WASM** — for corpora > 10K documents (~5ms for 100K vectors)

**Content acquisition:**
- Browser History API (user permission required) — automatic indexing of browsed pages
- Service Worker fetch interception — silent background indexing as user browses
- Active pull from OWI shards (via OWILIX-style pull) — bootstrapping from the OWI corpus

**Browser P2P:**
- **WebRTC data channels** — browser-native P2P communication (1–10 MB/s data throughput)
- **libp2p WebRTC transport** — cross-tier browser-to-server communication
- **Signaling server** — stateless STUN/TURN relay for NAT traversal (only required for handshake)

### Architectural Insights for semantiz

1. **Service Worker as the persistent browser-side agent.** Survives tab closures within the
   browser process lifetime. Intercepts page fetches, computes embeddings, maintains local
   vector index, participates in P2P gossip — without user action after initial install.
2. **Progressive indexing via browsing history.** Personalized, privacy-preserving local index
   that also contributes to the collective network's topical coverage.
3. **Browser nodes are non-custodian leaf peers.** Browser node process lifecycle is
   unreliable (browsers terminate Service Workers to reclaim resources). Custodian roles are
   reserved for persistent server nodes only.
4. **OPFS for large in-browser indexes.** Near-native I/O performance; arbitrarily large files.
   Correct backend for HNSW indexes on browser-tier nodes.
5. **WASM embedding for semantic consistency.** All browser nodes compute compatible embeddings
   regardless of OS, browser, or hardware — key to network-wide semantic consistency.
6. **Tiered peer architecture.** Server nodes: persistent, large storage, full embedding pipeline,
   custodian-eligible. Browser nodes: intermittent, limited storage, lightweight, non-custodian.
7. **libp2p WebRTC transport for cross-tier communication.** Enables direct browser-to-server
   communication over the same libp2p protocol stack.
8. **OWI shard bootstrapping for new browser agents.** Pull a small OWI thematic shard on first
   install to seed the local index before browsing history accumulates.

### Limitations

1. **Browser storage quota uncertainty** — IndexedDB/OPFS quotas are browser-dependent and can
   be revoked. `navigator.storage.persist()` mitigates but does not eliminate the risk.
2. **WebRTC NAT traversal requires STUN/TURN** — introduces a small degree of centralization.
3. **Embedding model versioning** — different browser agent versions using different models
   produce incomparable embedding spaces. Migration strategy required from the outset.
4. **Service Worker lifecycle** — browser can terminate at any time; not suitable for custodian
   roles or long-running expansion round participation.
5. **Privacy vs. contribution tension** — sharing embedding vectors can reveal user browsing
   interests via embedding inversion attacks. Differential privacy mechanisms needed.
6. **CORS restrictions** — browser agents can only proactively fetch CORS-enabled content.
   Severely limits proactive crawling versus server-side nodes.
7. **No persistent peer identity without IndexedDB** — if browser data is cleared, the peer's
   libp2p key pair is lost with no re-entry mechanism.
8. **Performance on low-end mobile** — an optional "lite mode" (smaller model, smaller index)
   is required for broad participation on constrained hardware.

---

## 6. Cross-Reference Synthesis

Combining all five references, the following design decisions are most actionable for semantiz:

### Topology (Semantica + DS4)

- Binary k-means trie as the primary SON, with custodians at each split-node
- `Δ ≈ 0.001` for soft clustering with 768-dim BERT embeddings (practical clone cap)
- `known-users ~208`, `closest-users = 50` as operational parameters
- Chain-hopping as the primary query routing primitive (O(ℓ), `ℓ ≤ 50` covers most queries)
- ~5% random long-range connections per node to prevent disconnected sub-graphs
- Optional social-graph routing layer (DS4) as a Phase 2+ extension for trust-enhanced routing

### Indexing (IPNI + OWI)

- IPNI advertisement chain model for decentralized index updates
- ContextID = leaf node in the Semantica tree; metadata = cluster centroid CID + model ID
- Gossipsub topic `/semantiz/ingest/mainnet` for announcement propagation
- Delegated Routing V1 HTTP API (`/routing/v1/providers/{cid}`) for ecosystem compatibility
- Separate ingest port (authenticated) from query port (public)
- HAMT for entry sets > 10K CIDs; EntryChunk linked lists for smaller sets
- WARC (raw archive) + Parquet with embedding vectors (query-ready) — two-layer corpus

### Content (OWI)

- Bootstrap from OWI corpus (9.14B URLs, 35 TiB) to avoid cold-start
- Reuse Resilipipe GPU classifiers for content quality filtering (Apache/CC BY 4.0)
- CIFF format for inverted index interoperability with Anserini/Terrier (BM25 baseline)
- Parquet + DuckDB for local node query-ready index

### Browser Tier (In-Browser Agentic Web)

- Service Worker as the persistent browser-side semantiz agent
- `all-MiniLM-L6-v2` via ONNX Runtime Web / Transformers.js (384-dim, ~22M params)
- OPFS for browser-side HNSW storage
- WebRTC data channels for browser P2P; libp2p WebRTC transport for cross-tier
- Browser nodes: non-custodian leaf peers only

### Security and Governance

- ed25519 libp2p key pairs as universal node identifiers across all tiers
- IPNI-style advertisement signing for index authenticity
- Backup custodian election (primary + secondary) for split-node resilience
- Periodic tree rebalancing protocol (open problem — must be designed for semantiz)
- DAO or nonprofit governance structure from the outset (informed by OWI consortium model)
- Differential privacy for embedding sharing (open research problem)

---

*Analysis performed July 2026. Semantica PDF fully extracted (74,340 characters). IPNI analyzed
from the full GitHub specification. OWI analyzed from project website, dashboard metrics, and
Zenodo community records. DS4 analyzed from abstract and contextual sources (full text paywalled,
project page offline). In-Browser Agentic Web analyzed from Zenodo metadata and author publication
history (PDF binary extraction failed in analysis environment).*
