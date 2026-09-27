# Changelog

All notable changes to this project will be documented in this file.

The format is based on [Keep a Changelog](https://keepachangelog.com/en/1.1.0/),
and this project adheres to [Semantic Versioning](https://semver.org/spec/v2.0.0.html).

---

## [Unreleased]

### Added
- **Architecture (RFC 0001):** Documented Hexagonal Architecture (Ports & Adapters) defining pure domain logic in `src/core/` and decoupled adapters in `src/adapters/` (CLI, MCP, and REST).
- **Architecture (RFC 0001):** Established Python 3.12+ language standard across the editorial ecosystem.
- **Architecture (RFC 0001):** Added Section 3.1.1 formalizing explicit resource and responsibility boundaries across `editorial-engine`, `editorial-content`, and `editorial-ops`.
- **Architecture (RFC 0001):** Added Section 3.2.3 detailing the Option 1 Embedded Multi-Extractor Micro-Pipeline with Extractor Registry (`trafilatura`, `pdfplumber`, `pypdf`, `ebooklib`), Canonical Output Contract, and uniform port exposure.
- **Architecture (RFC 0001):** Articulated the two-mode editorial lifecycle (Mode 1: The Discovery Engine vs. Mode 2: The Publishing Factory) with the formal Convergence Gate and time-boxed research requirement.

---

## [0.1.0] - 2026-09-27

### Added
- Initialized standalone `editorial-engine` repository.
- Added standard `.gitignore` for Python virtual environments, test caches, and build artifacts.
- Created `architecture/rfcs/0001_multi_plane_repository_topology.md` detailing multi-plane repository decomposition.

---

### Session Metadata
- **Active Conversation UUID:** `16f6fc7a-3b98-4470-bbe9-41bdb5aba68e`
