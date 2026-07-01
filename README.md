# huge_knowledge

A personal knowledge base. Notes, write-ups, and references I capture as I work on things — so I don't have to re-derive them next time.

Each file is a self-contained note on one topic.

## Index

### Auth & Identity
- [OAuth: HTTPS Website vs MCP Server](./oauth-website-vs-mcp.md) — actor model, flow diagrams, and contrasts between browser-based OAuth and MCP server OAuth.
- [Getting a Permanent Facebook Page Access Token](./facebook-permanent-page-access-token.md) — token types, the three-stage flow, and how to derive a non-expiring Page token for backend services.

### Infra & Networking
- [How ngrok Works: Serving localhost Without Port Forwarding](./ngrok-tunneling.md) — outbound tunnel model, usage, and which traffic hops are encrypted vs. visible to ngrok.

### AI / LLM / MCP
- [Organizing a GitHub Repo for Agentic Coding](./agentic-coding-repo-organization.md) — folder structure, issue templates, the bug-fix / design-develop / testing workflows, and whether you still need Jira and Confluence.
- [The DEV–QA Contract: Keeping QA a Gatekeeper in the Agentic Era](./dev-qa-contract-agentic-era.md) — how QA can fix bugs with coding agents without losing independence; separation of powers via DEV-owned `spec.md` and QA-owned `test-cases.md`, the spec-change escalation rule, and the prompt/hook/CI enforcement layers.

### Finance & Markets
- [Reading the Fed Through Futures](./fed-funds-futures-and-fedwatch-handbook.md) — Fed funds futures mechanics, EFFR, how the CME FedWatch tool turns prices into rate-decision probabilities (with a worked binary probability tree), the "Big Three" macro data releases, and FRED/CME data extraction.

<!-- Add more sections as the library grows, e.g.:
### Infra & Deployment
### Data & Pipelines
### Tooling & Workflow
-->

## Notes

- [The Index Rebalance Playbook (SpaceX, Tesla, DigitalOcean)](notes/index_rebalance_playbook.md)
