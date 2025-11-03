Purpose
- Policies for agent behavior

Inputs
- AAP best practices and repo governance

Outputs
- Consistent, safe agent actions

Rules
- Must be idempotent
- Must use check_mode where possible
- Prefer existing roles in 03-aap/collections/.../roles/
- Must NOT modify 01-chef-src/
- Must write rationale to ai-pipeline/runs/<timestamp>/notes.md
