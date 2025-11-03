Purpose
- Message exchange between agents

Inputs
- Discovery and analysis artifacts

Outputs
- JSON envelopes per step

Rules
- Use deterministic keys; keep paths relative to repo root

Example
{
  "from": "01-discovery",
  "to": "02-chef-to-ansible",
  "source_paths": ["01-chef-src/cookbooks/appserver"],
  "analysis_paths": ["ai-pipeline/analysis/chef/appserver.json"],
  "constraints": ["idempotent", "use-existing-role-structure"]
}
