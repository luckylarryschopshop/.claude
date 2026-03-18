---
name: webapp
description: Standard web application framework — agents, phases, and skill loading for browser-based products.
---

# Framework: Web Application

## Agent Roster
| Agent | Phase | Trigger |
|-------|-------|---------|
| PM | 1 | Project start — define requirements |
| UX | 1–2 | User flows and information architecture |
| Architect | 2 | System design before implementation |
| Designer | 2 | Design system and component specs |
| Database | 2 | Schema design (parallel with Architect) |
| Backend | 3–4 | API implementation |
| Frontend | 3–4 | UI implementation (parallel with Backend) |
| Security | 4 | Pre-launch review |
| DevOps | 4–5 | CI/CD and deployment |
| Tester | All | After every phase — mandatory gate |

## Phase Structure

### Phase 1 — Discovery
Goal: Requirements and user flows defined and approved.
Agents: PM → Analyst (optional) → UX
Skills: SKILL-requirements, SKILL-project-mgmt, SKILL-ux-research
Done when: User stories with acceptance criteria written; UX flows approved; tech requirements documented

### Phase 2 — Architecture and Design
Goal: Technical architecture and visual design language settled.
Agents: Architect + Designer + Database (parallel)
Skills: SKILL-system-design, SKILL-visual-design, SKILL-database, SKILL-security (threat model)
Done when: ADRs written; schema drafted; design tokens defined; TESTER APPROVED

### Phase 3 — Core Implementation
Goal: Core user journeys working end-to-end.
Agents: Backend + Frontend (parallel)
Skills: SKILL-code-quality, SKILL-tdd, SKILL-api-design, SKILL-logging, SKILL-frontend
Done when: Critical user stories implemented; TESTER APPROVED

### Phase 4 — Complete and Secure
Goal: All user stories complete; security review passed; production-ready.
Agents: Backend + Frontend (remaining stories) → Security → Tester
Skills: SKILL-security, SKILL-security-audit
Done when: All acceptance criteria met; security report shows no Critical/High findings; TESTER APPROVED

### Phase 5 — Deploy
Goal: Production deployment with monitoring.
Agents: DevOps → Tester (smoke tests)
Skills: SKILL-devops
Done when: App deployed; smoke tests green; monitoring active; TESTER APPROVED

## Skill Loading Table
| Phase | Load these skills |
|-------|-------------------|
| 1 | SKILL-requirements, SKILL-project-mgmt |
| 2 | SKILL-system-design, SKILL-visual-design, SKILL-database |
| 3 | SKILL-code-quality, SKILL-tdd, SKILL-api-design, SKILL-logging, SKILL-frontend |
| 4 | SKILL-security, SKILL-security-audit |
| 5 | SKILL-devops |

## Typical Stack Decisions (Not Prescriptive)
- Frontend: React / Next.js / Vue — choose based on team expertise
- Backend: Node.js / Python / Go — choose based on use case
- Database: PostgreSQL as default; Redis for caching
- Auth: Use an established provider (Auth0, Clerk, Supabase Auth) before building custom
- Hosting: Vercel/Netlify for frontend; Railway/Render/Fly.io for backend (start simple)
