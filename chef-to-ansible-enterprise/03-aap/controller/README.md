Purpose
- AAP Controller-as-Code definitions

Inputs
- Reviewed playbooks and inventories from 03-aap

Outputs
- Projects, credentials, inventories, job_templates YAML

Rules
- Minimal job_template fields:
  - name
  - inventory
  - project
  - playbook
  - execution_environment
  - credentials
  - survey (optional)
  - labels
- Agent 05 may add new job templates but MUST NOT remove human-authored ones
