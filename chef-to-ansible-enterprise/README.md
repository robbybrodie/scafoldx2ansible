# Chef → Ansible (AAP) Enterprise Migration

This repository implements a three-lane model for a Chef→Ansible migration pipeline:

- 01-chef-src/: Immutable source lane (read-only for agents)
- 02-ansible-draft/: AI/multi-agent draft lane (safe to overwrite)
- 03-aap/: Enterprise, AAP-conformant lane (requires human review; do not auto-overwrite)

Promotion is one-way: 01 → 02 → 03.

All generated Ansible must be idempotent. This repo is designed for ingestion into a vector database for RAG workflows.

## Architecture (overview)
- Multi-source ingestion: Chef, Puppet, and legacy Ansible (upstream) feed into a discovery stage.
- Multi-agent pipeline (discovery → convert → normalise → security/lint → docs+controller) writes to lanes 02 and 03.
- Human-in-the-loop gates: promotions from 02 → 03 happen via reviews/PRs; agents propose changes but do not auto-overwrite production.
- Message bus: `ai-pipeline/exchanges/*.json` carries agent handoffs; artifacts are indexed into a vector DB for retrieval-augmented reasoning.
- Quality gates: tree-sitter/static analysis, schema validation, ansible-lint, policy checks prevent monoliths and enforce role-based structure.

## How to use this repo (flow)
1. Place source inputs into `01-chef-src/` (Chef cookbooks/roles/envs/data_bags) and/or point agents at Puppet and legacy Ansible inputs.
2. Run Agent 01 (Discovery) to emit `ai-pipeline/runs/<ts>/discovery.json` and exchange messages under `ai-pipeline/exchanges/`.
3. Run Agent 02 (Convert) to produce role-first content in `02-ansible-draft/` (SAFE TO OVERWRITE).
4. Run Agent 03 (AAP Normaliser) to populate `03-aap/collections/...` and inventories/controller-as-code (REQUIRES REVIEW).
5. Run Agent 04 (Security/Lint) to write reports into `02-ansible-draft/reports/` and annotate PRs.
6. Run Agent 05 (Docs+Controller Packager) to generate controller YAML/docs, open PRs for promotion.
7. Human reviews PRs; on approval, promote 02 → 03. CI (`.github/workflows/validate-ansible.yml`) validates on push.

## Diagrams (Mermaid)

### Flow (multi-source → multi-agent → AAP)
```mermaid
flowchart TD
  Chef[Chef Cookbooks / Environments / Data Bags] --> A1[Agent 01: Discovery]
  Puppet[Puppet Manifests / Hiera] --> A1
  Legacy[Legacy Ansible Playbooks] --> A1

  A1 -- discovery.json --> Bus[(ai-pipeline/exchanges/*.json)]
  A1 -. index .-> RAG[(Vector DB)]

  Bus -- inputs --> A2[Agent 02: Convert (source → draft roles)]
  A2 -- draft roles/playbooks --> DRAFT[02-ansible-draft/]
  A2 -. index .-> RAG

  DRAFT --> A3[Agent 03: AAP Normaliser]
  A3 -- collection, inventories, controller yaml --> PROD[03-aap/]

  PROD --> A4[Agent 04: Security/Lint]
  A4 -- reports --> Reports[02-ansible-draft/reports/]

  PROD --> A5[Agent 05: Docs + Controller Packager]
  A5 -- PRs & bundles --> Gate[Promotion Gate (Human Review)]
  Reports --> Gate

  Gate -- approve --> PROD
  Gate -- request changes --> A2
```

### Sequence (human-in-the-loop collaboration)
```mermaid
sequenceDiagram
  participant Dev as Human Reviewer
  participant D as Agent 01 Discovery
  participant C as Agent 02 Converter
  participant N as Agent 03 Normaliser
  participant S as Agent 04 Security/Lint
  participant P as Agent 05 Packager
  participant Bus as Exchanges JSON
  participant RAG as Vector DB
  participant Git as Git (02-draft / 03-aap)

  D->>Bus: write discovery.json
  D-->>RAG: index discovery artifacts
  C->>Bus: read discovery.json
  C->>Git: write 02-ansible-draft roles/playbooks
  C-->>RAG: index draft changes
  N->>Git: read 02; write AAP collection to 03-aap
  S->>Git: lint 03-aap; write reports → 02-ansible-draft/reports
  P->>Git: generate controller YAML, docs; open PR
  Dev->>Git: review PR (promotion 02→03)
  Dev-->>Bus: approval or change-request event
  Git-->>RAG: index promoted content
```

### Components and quality gates
```mermaid
flowchart LR
  subgraph StaticAnalysis
    TS[tree-sitter parsers\n(Chef/Puppet/Ansible)]
    YAML[YAML Schemas\n(JSON Schema)]
    Lint[ansible-lint]
    SecScan[Secret Scanners\n(truffleHog/gitleaks)]
    Policy[Policy Engine\n(OPA/Rego)]
    Semgrep[Semgrep (YAML & code)]
  end

  Agents[Multi-Agent Orchestrator] --> StaticAnalysis
  StaticAnalysis --> Reports[(02-ansible-draft/reports)]
  Agents --> Bus[(ai-pipeline/exchanges)]
  Bus --> RAG[(Vector DB)]
  Agents --> Draft[(02-ansible-draft)]
  Agents --> Prod[(03-aap)]
```

## Source-specific mapping (summary)
- Chef → Role-first structure:
  - Recipes → `tasks/{install,config,service}.yml`
  - Attributes → `defaults/main.yml`
  - Templates → `templates/`
  - Files → `files/`
  - Dependencies → `meta/main.yml`
- Puppet → Role-first structure:
  - Classes/manifests → `tasks/*.yml` grouped by capability
  - Class parameters → `defaults/main.yml`; Hiera → `group_vars/` and `host_vars/`
  - ERB/EPP templates → `templates/`; files → `files/`
- Legacy Ansible (monolithic playbooks) → decompose into roles:
  - Extract variables to `defaults/`; split tasks by concern; add handlers for restarts
  - Keep playbooks thin; prefer `roles:` over large task lists

## Human-in-the-loop touchpoints
- Proposal PRs: Agent 05 opens PRs for promotions and docs/controller changes.
- Review gates: human approval required for any changes under `03-aap/`.
- Recommendation loops: agents emit rationale and suggestions into `ai-pipeline/runs/<ts>/notes.md`; reviewers can accept/apply.

## Anti-monolith guarantees
- Role-first policy: playbooks should primarily include roles; avoid large inline task lists.
- Size/complexity checks: enforce max task file length and task count; suggest refactors into `include_tasks`.
- Variable-first policy: converters MUST write values into `defaults/` (not hard-code in tasks).

## Static analysis & quality
- Parse sources with tree-sitter (Chef, Puppet, Ansible) to build structured inventories for agents.
- Validate YAML with JSON Schema (`ai-pipeline/schemas/*`).
- Lint with `ansible-lint`; scan secrets with gitleaks/truffleHog; enforce policies with OPA/Rego.
- Index artifacts into a vector DB for RAG; agents retrieve prior decisions and mappings.

## Extending the agents
- Add new agent specs in `ai-pipeline/agents/` and define handoffs in `ai-pipeline/exchanges/`.
- Keep draft writes under `02-ansible-draft/` (SAFE TO OVERWRITE); production writes to `03-aap/` require PRs.
