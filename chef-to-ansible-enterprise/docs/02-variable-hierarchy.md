Purpose
- Establish variable precedence and authoring rules

Inputs
- Role defaults, inventories, surveys

Outputs
- Deterministic variable resolution

Rules
- Precedence:
  1. role defaults/main.yml
  2. inventory group_vars
  3. host_vars
  4. controller survey
- Converter agents MUST emit vars into defaults/ first; do not hard-code in tasks
