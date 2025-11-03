# Chef → Ansible (AAP) Enterprise Migration

This repository implements a three-lane model for a Chef→Ansible migration pipeline:

- 01-chef-src/: Immutable source lane (read-only for agents)
- 02-ansible-draft/: AI/multi-agent draft lane (safe to overwrite)
- 03-aap/: Enterprise, AAP-conformant lane (requires human review; do not auto-overwrite)

Promotion is one-way: 01 → 02 → 03.

All generated Ansible must be idempotent. This repo is designed for ingestion into a vector database for RAG workflows.
