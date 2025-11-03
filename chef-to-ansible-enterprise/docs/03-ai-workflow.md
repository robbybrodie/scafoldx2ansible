Purpose
- Define agent workflow and responsibilities

Inputs
- Repo lanes and analysis outputs

Outputs
- Coordinated multi-agent execution

Rules
- Agent 01 – Discovery
  Inputs: 01-chef-src/
  Outputs: ai-pipeline/runs/*/discovery.json
  May overwrite?: No
- Agent 02 – Chef→Ansible Converter
  Inputs: discovery.json
  Outputs: 02-ansible-draft/
  May overwrite?: Yes
- Agent 03 – AAP Normaliser
  Inputs: 02-ansible-draft/
  Outputs: 03-aap/collections/…
  May overwrite?: No
- Agent 04 – Security/Lint
  Inputs: 03-aap/
  Outputs: 02-ansible-draft/reports/
  May overwrite?: Yes
- Agent 05 – Doc+Controller Packager
  Inputs: 03-aap/
  Outputs: 03-aap/controller/*.yml and docs
  May overwrite?: No
