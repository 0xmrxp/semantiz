# Changelog

All notable changes to the semantiz architecture and documentation are recorded here.

Format: [Keep a Changelog](https://keepachangelog.com/en/1.1.0/)  
Versioning: [Semantic Versioning](https://semver.org/spec/v2.0.0.html)

---

## [Unreleased]

---

## [0.2.0] - 2026-07-14

### Added
- Phase 4 (Tokenomics) added to roadmap: ERC-20 `$SMZ` token, liquidity bootstrapping, DAO governance, validator staking
- Phase 5 (Scale) added to roadmap: index sharding, WebRTC browser tier, cross-chain bridging, OWI corpus integration
- `RESEARCH.md`: deep technical analysis of all five academic references
  - Semantica (arXiv:2502.10151) — full algorithm extraction with benchmarks
  - DS⁴ (Springer ECIR 2013) — dual-facet routing insights
  - Open Web Index — scale metrics, corpus architecture, reusable classifiers
  - IPNI — advertisement chain protocol, IPLD schemas, API spec
  - In-Browser Agentic Web — browser-tier peer design

### Changed
- Roadmap updated: Phase 1 marked complete, Phase 2 and Phase 3 marked in progress
- Phase numbering shifted: old Phase 4 (Scale) moved to Phase 5 to accommodate new Tokenomics phase

---

## [0.1.0] - 2026-07-14

### Added
- `BLUEPRINT.md`: full technical architecture specification (14 sections, ~10,600 words)
  - Executive summary and differentiator table
  - System architecture with ASCII layer diagrams and data flow diagrams
  - Component specifications for all 14 components
  - Data models (7 schemas + protobuf definitions)
  - API contracts (6 endpoints)
  - P2P network protocol (libp2p, Kademlia DHT, Plumtree gossip)
  - Proof of Contribution mechanism with Solidity pseudocode
  - Semantic Overlay Network design (k-means trie, soft clustering)
  - Security model (Sybil, spam, eclipse, query poisoning)
  - Performance targets (p50/p95/p99 SLAs, hardware requirements)
  - Implementation roadmap (Phase 0–4, ticket-level tasks)
  - Annotated monorepo directory structure
  - Open research questions (7 unresolved problems)
  - Glossary (30+ domain terms)
- `README.md`: professional README with badges, architecture diagrams, tech stack table, research reference table
- `LICENSE`: Apache License 2.0
