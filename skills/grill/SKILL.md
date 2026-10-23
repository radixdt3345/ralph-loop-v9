---
name: "ralph-loop-v9:grill"
slug: grill
description: >
  Called by /start (Step 2). Do not invoke directly — type /start to begin.
  Runs deep project discovery: one-question-at-a-time interrogation across 5 rounds.
  Produces docs/00_discovery.md and a Clarity Summary passed to /write-prd.
---

# /grill — Forge Discovery

Run immediately when triggered. No preamble. Start with Round 1.

Produces two mandatory outputs:
1. **`docs/00_discovery.md`** — the confirmed project record (read by /constitution,
   /qmd, and /start Step 3)
2. **Clarity Summary** — structured in-context summary passed directly to /write-prd

---

## Phase 1 — Open Listen

Ask ONE opening question:

> "Tell me what you want to build. Don't worry about structure — just describe it
> in your own words."

Wait for the response. Then do the **Architect Review** before asking anything else.

---

## Architect Review (After Round 1)

Before narrowing into requirements, act as a senior architect:

1. **Restate what you heard** in 3–4 bullet points. Confirm understanding.
2. **Expand the problem space** — name 2–3 things the user didn't mention but probably
   needs (e.g. auth, audit trail, multi-tenancy, mobile, offline, rate limiting).
3. **Challenge the approach** — propose 2–3 alternative architectures or scope
   boundaries. Don't just accept the first framing.
4. **Flag the top 3 risks** — technical, operational, compliance, or scope risks.
5. **Ask one clarifying question** about the most significant unknown.

Wait for the user's response, then continue to Round 2.

---

## Round 2 — Domain + Roles

Ask questions one at a time. Wait for each answer before the next.

Cover:
- Bounded contexts / domains (what are the main areas of the system?)
- User roles (who uses this? what can each role do? what can't they do?)
- Multi-tenancy (single tenant, multi-tenant, hybrid?)
- External users vs internal only
- Estimated user volume and growth curve

Build the **Role Registry** as you go:
| Role | Description | Key Permissions | Key Restrictions |

---

## Round 3 — Features + Per-Feature RBAC

For each major feature area identified in Round 2:

1. Ask what it does (one feature at a time)
2. Ask who can do it (which roles)
3. Ask what data it touches
4. Ask edge cases and error states
5. Ask any compliance or regulatory requirements

Build the **Permission Matrix** as you go:
| Resource / Action | [Role 1] | [Role 2] | [Role N] |

Flag any HITL items (features that need explicit user review in /prd-to-features).

---

## Round 4 — Technical Decisions

Ask one question at a time. Cover:

- **Frontend**: framework, component library, SSR/SPA/mobile?
- **Backend**: language, framework, API style (REST/GraphQL/gRPC)?
- **Database**: relational/document/both? specific technology? ORM?
- **Infrastructure**: cloud provider, containers, serverless?
- **Auth**: JWT/session/OAuth/SSO? MFA?
- **CI/CD**: GitHub Actions/GitLab CI/Azure Pipelines?
- **Third-party integrations**: payments, email, analytics, monitoring?
- **Branch strategy**: single main or dev→main tiered?
- **Deployment environments**: local, staging, prod only? More?

For each choice, note any constraints or hard requirements stated.

---

## Round 5 — Edge Cases + Non-Functional Requirements

Cover:
- Performance targets (response time, throughput, concurrent users)
- Availability requirements (uptime SLA, downtime tolerance)
- Data retention and archival
- Offline capability or graceful degradation
- Internationalisation / localisation
- Accessibility requirements (WCAG level?)
- Security certifications or compliance (SOC2, GDPR, HIPAA, PCI?)

---

## Final Confirmation — ⛔ HARD STOP

Present a complete summary:

```
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
FORGE DISCOVERY — FINAL CONFIRMATION
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━

Project: [name]
One-line description: [description]

Domains: [list]

Role Registry:
[table]

Permission Matrix:
[table]

Features confirmed:
[numbered list with per-feature RBAC]

Technical stack:
[list of decisions]

Non-functional requirements:
[list]

Risks flagged:
[list from architect review]
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
Type YES to confirm, or correct anything above.
```

Do NOT proceed until the user types YES or confirms corrections.
After corrections, re-present the updated summary and ask again.

---

## Output 1 — Write `docs/00_discovery.md`

After confirmation, write the discovery record:

```markdown
# Discovery Record — [Project Name]
Generated: [date]
Status: CONFIRMED

## Project Overview
[One paragraph description]

## Domains
[One section per domain with description and owned resources]

## Role Registry
[Full table]

## Permission Matrix
[Full table]

## Features
[One section per feature with:
- Description
- RBAC (which roles, which actions)
- Data touched
- Edge cases
- Compliance notes]

## Technical Decisions
[All Round 4 decisions with rationale]

## Non-Functional Requirements
[All Round 5 items]

## Risks
[Top 3–5 risks from architect review]

## Open Questions
[Anything marked DEFERRED or needing follow-up]
```

---

## Output 2 — Clarity Summary (in context)

After writing `docs/00_discovery.md`, produce the following in the conversation
(this is consumed by /write-prd immediately after):

```
CLARITY SUMMARY — [Project Name]

Core purpose: [one sentence]
Primary users: [roles list]
Must-have features: [numbered list with acceptance criteria pointers]
Must-not features: [explicit out-of-scope items]
Key constraints: [tech decisions, compliance, performance]
Risks for PRD: [items /write-prd should flag as HITL]
```

Then hand control back to /start (which calls /write-prd next).
