# AZ-305 Study Guide Update Prompt

Use this prompt when you want an AI assistant to review, correct, and improve the study guides in this repository.

## Prompt

You are updating the AZ-305 study guide markdown files in this repository.

Your goals are:

1. Keep the guides accurate against current Microsoft Learn documentation.
2. Preserve the existing structure, tone, and exam-focused style unless a clear improvement is needed.
3. Prefer targeted edits over broad rewrites.
4. Keep the content optimized for AZ-305 exam preparation, not general Azure documentation.

### Repository scope

Review and update these files as needed:

- `README.md`
- `application-architecture-study-guide.md`
- `bcdr-study-guide.md`
- `compute-study-guide.md`
- `cosmos-db-study-guide.md`
- `data-integration-study-guide.md`
- `identity-governance-study-guide.md`
- `messaging-study-guide.md`
- `migrations-study-guide.md`
- `monitoring-study-guide.md`
- `networking-study-guide.md`
- `relational-data-study-guide.md`
- `storage-study-guide.md`

### Update rules

1. Use Microsoft Learn as the primary source of truth for service capabilities, retirement dates, SKU names, defaults, and exam-scope topics.
2. Focus especially on drift-prone items:
   - retirement and deprecation dates
   - preview vs GA status
   - SKU and tier names
   - SLA expectations
   - default behaviors
   - RBAC and identity behavior
   - networking and load-balancing decisions
3. Keep content concise and high value.
4. Do not add generic filler.
5. Do not remove useful exam traps, decision tables, or scenario guidance unless they are inaccurate.
6. When correcting facts, prefer the smallest edit that fixes the issue.
7. If a topic is not commonly tested or is outside AZ-305 scope, do not expand it unless it is needed for accuracy.

### Expected workflow

1. Read the relevant markdown files before editing.
2. Identify any outdated or weak sections.
3. Verify factual changes against Microsoft Learn.
4. Update only the files that need changes.
5. Preserve headings and link structure when possible.
6. After editing, run a validation pass for markdown or formatting issues.
7. Summarize:
   - what changed
   - which files were updated
   - what was verified against Microsoft Learn
   - any remaining items that should be rechecked closer to exam day

### Content priorities

Prioritize improvements that help with exam answers, especially:

- choosing between Azure Front Door, Traffic Manager, Load Balancer, and Application Gateway
- RBAC scope, role assignment behavior, and control plane vs data plane
- Key Vault, Policy, management groups, and governance patterns
- HA/DR decision points and RTO/RPO tradeoffs
- storage, relational data, and Cosmos DB selection guidance
- monitoring, agents, logs, alerts, and cost tradeoffs
- migration service selection
- messaging and integration service selection

### Style requirements

1. Keep the writing direct and study-oriented.
2. Use bullet points, tables, and short decision rules where helpful.
3. Prefer “When this is the right answer” style guidance for exam scenarios.
4. Keep exam-trap sections sharp and easy to scan.
5. Avoid unnecessary repetition across files.

### Output requirements

At the end, provide:

1. A short summary of the updates made.
2. A list of files changed.
3. A short list of any facts that are time-sensitive and may need future verification.

### Reusable request template

Use this request when starting an update:

"Review the AZ-305 study guide markdown files in this repository and update any outdated, inaccurate, or weak sections. Verify factual changes against Microsoft Learn. Preserve the current exam-focused structure and make only targeted edits. Prioritize networking/load balancing, RBAC, governance, BCDR, data platform decisions, monitoring, and other date-sensitive Azure details. After editing, summarize what changed, what was verified, and what may need rechecking later."