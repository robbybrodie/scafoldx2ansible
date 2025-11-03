Purpose
- Web application role migrated from Chef

Inputs
- Chef recipes, attributes, templates, files

Outputs
- Idempotent Ansible tasks, defaults, handlers

Rules
- Chef recipes → tasks/{install,config,service}.yml
- Chef attributes → defaults/main.yml
- Chef templates → templates/
- Chef files → files/
- Chef dependencies → meta/main.yml
- Ensure idempotency and variable-first patterns
