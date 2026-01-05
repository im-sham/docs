# Documentation Changelog

All notable documentation updates will be logged here.

## [2026-01-05] Documentation Overhaul - Phase 6 & 7

### Added
- `docs/features/document-intelligence.mdx` - Document processing and entity extraction
- `docs/features/knowledge-graph.mdx` - Graph Intelligence (Phase 6) documentation
- `docs/features/data-connectors.mdx` - Data Connector Infrastructure (Phase 7A) documentation
- New "Features" tab in navigation for core capabilities

### Changed
- `docs/docs.json` - Reorganized navigation with Features tab, moved quickstart to Overview
- `docs/index.mdx` - Complete rewrite as modern SaaS landing page with CardGroups
- `docs/guides/quickstart.mdx` - Rewritten for SaaS users (sign in → upload → query flow) with developer self-hosted in accordion
- `docs/investor/traction.mdx` - Updated to reflect Phase 6 complete, Phase 7A complete, 114 passing tests

### Removed
- Developer-focused installation as primary flow (moved to accordion)
- `docs/guides/advanced-configuration.mdx` from main navigation

---

## [2026-01-02] Streamlit Deprecation

### Removed
- `ui/` directory (Streamlit frontend) - React Tactical HUD is now the only frontend
- `streamlit` dependency from pyproject.toml

### Changed
- Updated all documentation to remove Streamlit references
- Updated multi_tenancy.md to reference React frontend instead of Streamlit

---

## [2026-01-02] Documentation Sprint

### Added
- `docs/guides/analytics.mdx` - KPI tracking and ROI analysis documentation
- Features navigation group in docs.json

### Changed
- `docs/index.mdx` - Updated tech stack (React 19 + Vite, Hybrid RAG Router)
- `docs/guides/quickstart.mdx` - Added React frontend startup instructions
- `docs/architecture/overview.mdx` - Updated diagram and tech stack for React + Router
- `docs/architecture/data-flow.mdx` - Added Hybrid RAG Router flow diagram
- `docs/guides/design-partner-onboarding.md` - Updated for key-based multi-tenancy

### Fixed
- Removed stale Streamlit-only references (now listed as legacy option)

## [Unreleased]

### Added
- Initial documentation structure with Mintlify
- Business case and market positioning
- Architecture overview with system diagram
- Investor materials (pitch narrative, traction)
- Quickstart guide

---

## Format

Each entry should include:
- **Date** in YYYY-MM-DD format
- **Added/Changed/Removed** sections as needed
- Brief description of documentation changes
