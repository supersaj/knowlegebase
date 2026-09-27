---
name: product-context-architecture
description: Standard for the payments department's product knowledge base (Product Context Layer) - what it contains (product discovery, rules, C4 architecture, decisions, operations), where each piece lives, current, interim and target states, use with or without BMAD, and how to keep it current. Use when creating, reviewing, updating or querying a product's knowledge base, deciding where knowledge belongs, modelling architecture states, or promoting knowledge after a release or epic.
---

# Product Knowledge Base Standard

## 1. Purpose

One governed, version-controlled knowledge base per product that people and AI agents use to understand, build, change and run payments systems safely.

- It records **what is true about the product**, **who it serves**, **why it is built this way** and **how to run it**.
- It does not replace the SDLC. BMAD, Spec Kit, Jira or any process keeps its own working artifacts.
- Every fact has **one owner and one home**. Everything else links to it.

## 2. What it contains

| Area | Knowledge | ID | Owner |
|---|---|---|---|
| Product | Overview: vision, value, stakeholders, success measures, boundaries | `overview.md` | Product owner |
| Product | Personas: customers, operators, partners | `PER-*` | Product owner |
| Product | Customer and operator journeys (current and target) | `JRN-*` | Product owner |
| Product | Use cases | `UC-*` | Product owner |
| Product | Pain points with evidence, severity and resolution status | `PAIN-*` | Product owner |
| Product | Capabilities | `CAP-*` | Product owner |
| Product | Business rules | `BR-*` | Product owner |
| Product | Quality attributes: availability, latency, throughput, security, resilience targets | `NFR-*` | Product architect |
| Product | Data: key entities, classification, retention | `data/` | Product architect |
| Product | Compliance applicability: which obligations and controls apply | `compliance.md` | Product owner + compliance |
| Product | Glossary | `TERM-*` | Product owner |
| Architecture | C4 model: context, containers, components, deployment, key flows | `architecture/model/` | Product architect |
| Architecture | Architecture states: current, interim, target | `states.yaml` | Product architect |
| Architecture | Decisions | `ADR-*` | Product architect |
| Architecture | Narratives: security, resilience, integration, data flow | `NAR-*` | Product architect |
| Architecture | API and event contracts | OpenAPI / AsyncAPI beside the service | Service team |
| Operations | Runbooks, support model, incident learnings | `RB-*` | Operations lead |
| Quality | Test strategy, test data rules, rule-to-test traceability | `quality/` | QA lead |
| Agents | `AGENTS.md`, product skills, golden questions | — | Product architect |

Shared payments knowledge lives in the **domain tier**, not in products: rails, message formats and mappings, code sets, calendars, obligations (`OBL-PAY-*`), controls (`CTL-PAY-*`), approved patterns (`PAT-PAY-*`), domain glossary.

**Not stored here (linked instead):**

| Item | Lives in | How it connects |
|---|---|---|
| Roadmap and backlog, feature requests | Product management tool (Jira) | `product.yaml` links; epics cite `PAIN-*`, `UC-*`, `CAP-*` |
| Raw customer feedback, research notes, survey data | Research repository | Cited as evidence in `PAIN-*` and `JRN-*` |
| In-flight PRDs, epics, stories, specs | SDLC framework folders | Cite IDs; promoted at close |
| Deployments, incidents, tickets, runtime state | Systems of record | Queried through MCP |
| Secrets, customer or production data | Nowhere in the knowledge base | — |

## 3. Structure

Three tiers. Products reference upper tiers by ID and pinned version, never by copy.

| Tier | Repository | Holds |
|---|---|---|
| Enterprise | `enterprise-context` | This standard, schemas, AI and data policies, governance skills |
| Domain | `payments-domain-context` | Rails, message formats, code sets, obligations, controls, patterns, domain glossary and skills |
| Product | Each product repo | Everything product-specific (below) |

```
<product-repo>/
├── AGENTS.md                      short, human-verified agent instructions
├── product.yaml                   identity, owners, links (roadmap, backlog, research), domain pin, SDLC mapping
├── product-context/
│   ├── INDEX.md                   generated index of every artifact, never edited by hand
│   ├── overview.md                vision, value, stakeholders, success measures, boundaries
│   ├── personas/                  PER-*   customers, operators, partners
│   ├── journeys/                  JRN-*   customer and operator journeys, current and target
│   ├── use-cases/                 UC-*    what each persona does with the product
│   ├── pain-points/               PAIN-*  known problems, evidence, severity, resolution
│   ├── capabilities/              CAP-*   what the product does for the business
│   ├── business-rules/            BR-*    what must always be true
│   ├── quality-attributes/        NFR-*   availability, latency, throughput, security, resilience
│   ├── data/                      key entities, data classification, retention
│   ├── compliance.md              applicable obligations and controls (by domain ID)
│   └── glossary/                  TERM-*  product-only terms
├── architecture/
│   ├── model/                     C4 model, one file per state, plus states.yaml
│   ├── views/                     generated C4 diagrams, never hand-edited
│   ├── narratives/                NAR-*   security, resilience, integration, data flow
│   └── decisions/                 ADR-*
├── operations/
│   ├── runbooks/                  RB-*
│   ├── support-model.md           on-call, escalation, user-facing service levels
│   └── incident-learnings/        lessons promoted from post-incident reviews
├── quality/
│   └── test-strategy.md           approach, test data rules, rule-to-test traceability
├── evaluation/golden-questions.yaml
├── .agents/skills/                product skills
├── services/*/openapi.yaml …      contracts beside the code they describe
└── <SDLC folders, e.g. _bmad/, _bmad-output/>   owned by the SDLC framework
```

**How product knowledge links up:** persona → journey → use case → capability → business rule → C4 element → test. Pain points attach to the journeys, use cases or personas they affect, and to the epic that addresses them.

## 4. Core rules

1. **One source per fact.** Graph, search index, generated docs and agent memory are derived and rebuildable.
2. **Reference, don't copy.** Link by ID (`BR-PMTHUB-042`, `OBL-PAY-011`) instead of restating text.
3. **Front matter on every artifact:** `id`, `type`, `status`, `owner`, `review_by`, `provenance`, `relations`. CI, the catalog and the graph read it.
4. **Humans approve.** AI drafts as `status: proposed`. Only a CODEOWNER review makes it `active`.
5. **Live facts are queried**, never copied into files.
6. **`AGENTS.md` stays small:** commands, hard constraints, pointers. No generated overviews.
7. **Nothing sensitive:** synthetic examples only; no customer data, secrets or production messages. Pain-point evidence is summarized and linked, never pasted.
8. **Conflicts are reported, not resolved silently.** Precedence: regulation and policy → active rules and accepted ADRs → architecture baseline → in-flight SDLC artifacts → code → generated docs.

## 5. Architecture (C4)

- Architecture is described at C4 levels: **system context**, **containers**, **components** (only where they explain a decision), **deployment**, and **dynamic views** for critical payment flows.
- It is kept as **one model**, not hand-drawn diagrams. All C4 diagrams in `views/` are generated from it.
- The department stores the model in **FINOS CALM** format. CALM adds machine-checkable controls and approved patterns. Conventions: `references/calm-conventions.md`.
- Model elements link to rules (`BR-*`), decisions (`ADR-*`), quality attributes (`NFR-*`) and controls (`CTL-PAY-*`) by ID.
- Element IDs match catalog names and stay the same in every state.

## 6. Current, interim and target state

| State | Meaning | Changes when |
|---|---|---|
| **Current** | What runs in production today | A release goes live |
| **Interim** | A planned intermediate step: coexistence, bridge, parallel run | Its plan changes; removed after it goes live |
| **Target** | The approved end state | An ADR changes the target |

Rules:
- Each state is one architecture model file; `states.yaml` lists them in order with status (`live`, `planned`, `retired`), planned window, delivering epics, approving ADRs and exit criteria.
- Exactly one current state. Interim and target exist only when they differ from current.
- Temporary elements (bridges, adapters, dual writes) are marked transitional, with a sunset date and the ADR that allows them.
- Journeys, rules, ADRs and runbooks describe the current state by default. Future-only knowledge sets `state: target` or `state: transition-NN` and a future `effective_from`. A target journey is the customer view of the target architecture.
- **At go-live:** the delivered state becomes current, `states.yaml` is updated, target journeys that are now live become current, transitional runbooks are activated or retired. Git keeps history.

Which state to use:

| Task | Read |
|---|---|
| How does it work? Troubleshooting, operations | Current |
| Implement a slice | Diff: current → the interim state that slice delivers |
| Design, roadmap, new epic | Target, plus the next interim state, target journeys, open pain points |
| Audit, impact of a regulation change | Current, then target |

Scenarios: a **stable product** has current only; a **greenfield product** has target only until first release; a **modernization** has current, one or more interim states, and target.

## 7. Using it with BMAD

| BMAD step | Knowledge base role |
|---|---|
| Analysis, product brief | Reads overview, personas, journeys, open pain points. Cites `PAIN-*` and `JRN-*` as the problem. |
| PRD | Reads use cases, capabilities, business rules, NFRs, obligations. Cites IDs, never restates. |
| Architecture | Reads current and target C4 model, ADRs, domain patterns. Proposes a new interim or target model plus ADRs. |
| Readiness gate | No contradiction with active rules, accepted ADRs, NFRs or obligations; model validation passes. |
| Stories | Acceptance criteria cite `BR-*`, `NFR-*` and `ADR-*` IDs. |
| Retrospective | **Promotion:** new or changed rules, ADRs, use cases, journeys, runbooks via PR; pain points marked resolved; or an explicit "none". |
| Release go-live | Delivered state becomes current. |
| After promotion | Archive completed epics and stories so agents don't treat them as current requirements. |

- BMAD's own project-context block in `AGENTS.md` stays in its marked section. Neither tool edits the other.
- Configure BMAD through its supported customization mechanism, not by editing core files.
- Details: `references/bmad-integration.md`.

## 8. Using it without BMAD

The same four touchpoints apply to any process (Jira epics, Spec Kit, Scrum):

1. **Before design:** read the baseline (pain points, journeys, current and target architecture, rules, ADRs).
2. **Before build:** run the same readiness check.
3. **At epic or feature close:** run promotion.
4. **At go-live:** update states.

Record the process and its artifact locations under `sdlc:` in `product.yaml`.

## 9. Keeping it current

| Trigger | Action | Owner |
|---|---|---|
| Every PR | CI: validator, model validation, secret scan, regenerate `INDEX.md` and C4 views | Automated |
| Epic or feature close | Promotion PR (or "none") | Product architect |
| Release go-live | Promote state to current; update journeys and runbooks | Product architect + ops |
| Customer research, complaints, ops feedback | Add or update `PAIN-*` with linked evidence | Product owner |
| Incident | Update runbook; record the learning; check rules, NFRs and ADRs involved | Operations lead |
| Scheme or regulation change | Domain release; products upgrade their pin and review impact | Domain architect + compliance |
| `review_by` reached | Review, re-date, deprecate or supersede | Artifact owner |
| Quarterly | Scorecard and golden-question run; fix the lowest scores | Product architect |

Stale knowledge is a defect: overdue obligations fail CI; other overdue artifacts warn.

## 10. How agents use it

1. Start at `AGENTS.md` → `product.yaml` → `product-context/INDEX.md`.
2. Pick the architecture state for the task (section 6).
3. Retrieve by ID and path first; use catalog or graph for relationships; MCP for live state; search only for unstructured history.
4. Read `active` artifacts by default. Ignore `superseded`, `deprecated` and `retired` unless the task is historical.
5. Cite IDs for every business, regulatory or architectural claim.
6. Draft new knowledge as `proposed`; never mark it `active`.

## 11. Operating modes (for agents running this skill)

- **A. Assess** (read-only): inventory existing knowledge, owners, duplicates, gaps; run the validator; write `context-assessment.md` with the scorecard from `references/quality-and-evaluation.md`.
- **B. Bootstrap:** after A, create `product.yaml`, folders, `AGENTS.md`, overview, personas, journeys, pain points, the current C4 model and key rules from evidence (docs → research → contracts → code), all `proposed`; add the golden-question set; open a PR.
- **C. Map an SDLC framework:** fill `sdlc:` in `product.yaml`; wire the four touchpoints in section 8.
- **D. Promote:** extract candidates from a finished epic or release; update existing artifacts before creating new ones; PR using `assets/templates/promotion-checklist.md`.
- **E. Audit:** run `scripts/validate_context.py`; check model-to-code drift and rule-to-test coverage on a sample; report blockers first.
- **F. Evaluate:** run the golden questions; track retrieval accuracy, correctness and stale-source hits over time.

## 12. Reference files

| File | Use for |
|---|---|
| `references/repository-layout.md` | Tier repos, folders, skills locations, `AGENTS.md` sections |
| `references/artifact-schemas.md` | Front matter, IDs, statuses, review cadence, per-type fields |
| `references/calm-conventions.md` | How the C4 model is stored in CALM: element types, metadata, controls, flows, states |
| `references/bmad-integration.md` | BMAD and other frameworks in detail |
| `references/payments-domain-profile.md` | Rails, obligations, domain skills |
| `references/governance-and-controls.md` | RACI, approvals, data handling, skill and MCP governance |
| `references/retrieval-and-graph.md` | Catalog, knowledge graph, MCP, search |
| `references/quality-and-evaluation.md` | Scorecard, CI gates, golden questions, metrics |
| `assets/templates/` | `AGENTS.md`, `product.yaml`, persona, journey, use case, pain point, business rule, ADR, architecture model, `states.yaml`, promotion checklist |
| `scripts/validate_context.py` | CI checks for front matter, architecture models and states; builds `INDEX.md` and the graph |
