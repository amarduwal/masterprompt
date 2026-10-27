---
name: project-docs
description: Generate a full project documentation set (brief, PRD, SRS, UX, architecture, tech spec, security, test plan, build plan, feature specs) through a gated interview. Works for new and existing projects, web or mobile. Run before project-bootstrap.
argument-hint: "[your project idea, notes, or a path to them]"
disable-model-invocation: true
---

If the user typed anything after the command, treat it as their brain dump for Phase 1: $ARGUMENTS

# DOCS PROMPT: Generate the project documentation set

Run this first, in an empty folder or at the root of an existing repo. It produces the product and engineering docs. Then run `/project-bootstrap`, which reads these docs and sets up Claude Code and the infrastructure to build exactly what they describe.

---

## Mission

Produce the documents a team needs before writing code:

- **What** we are building and **why**: `BRIEF`, `PRD`
- **Exactly what it must do, and how well**: `SRS`
- **How it looks and flows**: `UX`
- **How it is built**: `ARCHITECTURE`, `TECH_SPEC`
- **How it stays safe**: `SECURITY`
- **How we prove it works**: `TEST_PLAN`
- **The order we build it in**: `PLAN`, `features/`
- **Why we chose what we chose**: `DECISIONS`

Every doc has two readers: a teammate new to the project, and a Claude Code session with no other context. Write for both: dense, specific, no filler.

## Ground rules for this run

- **Stop at every CONFIRM GATE and wait for my answer.**
- **Never invent facts.** If something is unknown, ask, or record it in the Open Questions table in `docs/README.md` (question, why it matters, the default you would use, who decides). Never write a guess as if it were decided.
- **Every requirement is testable.** Banned unless followed by a number or a condition: fast, quick, scalable, robust, secure, reliable, user-friendly, intuitive, seamless, modern, easy, as needed, where possible, etc.
- **Stable IDs.** `F-01` features, `US-001` user stories, `FR-001` functional requirements, `NFR-001` non-functional requirements, `ADR-001` decisions, `R-01` risks, `T-001` tasks. Never renumber. A dropped item is marked `Withdrawn` and its ID is never reused.
- **Each fact lives in one place.** Other docs link to it by ID. User stories and acceptance criteria live only in feature specs. "The system shall" statements live only in the SRS. The stack lives only in `TECH_SPEC`. Duplicated facts drift apart.
- **Size to the project.** An MVP's SRS can be three pages. A section that does not apply gets one line: `Not applicable: <reason>`. Never pad.
- **Diagrams in Mermaid**, so they render on GitHub and diff in review.
- **Writing style:** plain and direct. No em dashes or en dashes as punctuation, no curly quotes, no marketing tone, no AI tells. No AI attribution anywhere.
- **Re-runnable.** If docs already exist, update them in place, keep every ID, and show the diff before overwriting.
- **Ask before** `git init` or any commit.

---

## PHASE 0: DETECT (silent, no questions yet)

- **Code:** if the repo has code, read the manifests, folder structure, routes or screens, database schema, and CI config. Summarize briefly.
- **Existing docs:** `README*`, `docs/`, wiki exports, specs, design links, issue tracker exports, `CLAUDE.md`, `AGENTS.md`, `.cursorrules`.
- **Anything I attached or pasted** with this prompt: notes, sketches, links, screenshots.

Pick the mode:

- **New project:** the docs describe the intended product.
- **Existing project:** the docs describe the product **as built**, then the intended next state. Mark each functional requirement `Built`, `Partial`, or `Not built`. Derive features from routes and screens, the data model from the schema, and the architecture from the structure. Intent (why, for whom, what success means) cannot come from code, so ask. Never propose replacing the stack unless I ask.

Print **What I found** (15 lines max): mode, code summary, existing docs, gaps.

---

## PHASE 1: THE IDEA (round 1)

First, invite a brain dump: "Describe the project in your own words. Paste notes, links, competitor apps, sketches, anything." Then ask only what is still missing from the list below. Number the questions and show your proposed default next to each. I can answer `default` to any of them.

1. **Name** and one-sentence description.
2. **Problem:** who has it, how they solve it today, why that is not good enough.
3. **Users:** primary and secondary. Countries and languages.
4. **Core value:** the one thing the product must do brilliantly.
5. **Platforms:** web, iOS, Android, desktop. Which ships first?
6. **Business model:** free, one-time purchase, subscription, ads, B2B, internal tool.
7. **Number-one product constraint:** the one thing every decision protects. Example: "a user can open the app with no account and no signal and start in under five seconds."
8. **Hard boundaries:** what the product must never do or become. Examples: "no health claims", "no user accounts", "no ads".
9. **Goals** with measurable targets, and **non-goals**.
10. **Data:** what user data we collect. Any PII, health, financial, children's, or location data? Which jurisdictions (GDPR, CCPA, COPPA, HIPAA, PCI)?
11. **Offline:** must any part work with no network? Which part?
12. **Team and timeline:** who builds it, how many people, their skills, any deadline, monthly infrastructure budget.
13. **Scale tier** now and expected in 12 months:
    - **T1 Solo/MVP:** 1-3 devs, under 1k users, single region, no compliance burden
    - **T2 Growth:** 3-10 devs, 1k-100k users, uptime starts to matter
    - **T3 Scale:** 10+ devs, 100k+ users, multi-region, compliance
14. **References:** competitors or products we like, and what we do differently.
15. **Givens:** existing infrastructure, accounts, domains, or tech the team prefers or must use.

Write `docs/BRIEF.md` (contents in Phase 4) and show it.

### CONFIRM GATE 1. Brief approved.

---

## PHASE 2: SCOPE (round 2)

From the approved brief, propose:

- **Personas:** two to four.
- **User journeys:** the top three to five, as numbered steps.
- **Feature table:** `F-ID | Name | Purpose (one line) | Persona | Priority (Must, Should, Could, Won't) | Release (MVP, v1.x, Later)`.
- **The MVP line:** the smallest feature set that delivers the core value and tests the riskiest assumption. Justify it in three lines.
- **Out of scope:** explicit list.

Existing project: the table starts with what is built (marked `Built`), then proposes what comes next.

### CONFIRM GATE 2. Feature list and MVP line approved.

---

## PHASE 3: TECHNICAL DIRECTION (round 3)

Propose, with every choice tied to the brief (constraints, team skills, budget, tier):

- **Stack per layer:** client, backend or BaaS, database, hosting, auth, file storage, notifications, payments, error tracking, analytics (or none). Real package names and versions.
- **Real alternatives:** for each choice where a real alternative exists, a short table of two options with tradeoffs and a recommendation.
- **Architecture style:** client plus BaaS, monolith API, serverless, offline-first with static content, etc. One paragraph on why, and what would make us change it.
- **Top three technical risks**, and how to retire each early (spike, prototype, load test).

Existing project: describe the current stack and architecture, flag real problems, and propose changes only where a requirement forces them.

### CONFIRM GATE 3. Stack and architecture approved. Each approved choice becomes an ADR.

---

## PHASE 4: GENERATE

```
docs/
  README.md          index, reading order, doc status, open questions
  BRIEF.md
  PRD.md
  SRS.md
  UX.md
  ARCHITECTURE.md
  TECH_SPEC.md
  SECURITY.md
  TEST_PLAN.md
  PLAN.md
  DECISIONS.md
  LAUNCH.md          only for a public release
  features/
    F-01-<slug>.md   one per MVP feature, plus Should features whose details are known
  api/
    openapi.yaml     only if we run our own API
```

Every doc starts with:

```
# <Title>
Status: Draft | Last updated: YYYY-MM-DD | Covers: <IDs or scope>
```

### `README.md`

- What this project is (one line, link to BRIEF).
- **Reading order** for a new teammate, and for a Claude session starting a task: `CLAUDE.md`, `STATUS.md`, the task's feature spec, the SRS IDs it cites.
- Doc table: `Doc | Purpose | Status | Last updated`.
- ID conventions.
- **Open Questions:** `# | Question | Why it matters | Default if unanswered | Owner | Status`.

### `BRIEF.md`

One-liner. Problem. Users. Core value. Number-one product constraint. Goals with targets. Non-goals. Success metrics. Business model. Constraints (team, timeline, budget, legal). Hard boundaries, each with a one-line reason. Scale tier now and in 12 months. Competitors and how we differ. Assumptions to validate. Glossary of domain terms.

### `PRD.md`

- **Personas:** name, context, goals, frustrations, tech comfort, accessibility needs.
- **User journeys:** numbered steps, the happy path plus the most likely failure.
- **Feature table** from Gate 2, each row linking to `features/F-xx-<slug>.md`.
- **MVP definition** and the assumption it tests.
- **Success measurement per feature.** If there is no analytics, say how else we will know (store reviews, support email, usability sessions).
- **Out of scope.**
- **Release plan:** MVP, v1.x, Later.

### `SRS.md`

1. **Introduction:** purpose, scope, definitions (link the glossary), references.
2. **Overall description:** product perspective, user classes, operating environment (minimum OS versions, browsers and versions, devices, screen sizes), constraints (design, regulatory, store policy), assumptions and dependencies.
3. **Functional requirements**, grouped by feature: `FR-ID | The system shall ... | F-ID | Priority | Verify by (test, inspection, demo) | Status`.
4. **External interfaces:** UI (link UX), software (third-party APIs and SDKs, and what data crosses each one), hardware (camera, GPS, sensors), communications (HTTP, push, WebSockets).
5. **Data requirements:** entities (link the TECH_SPEC data model), retention, deletion, export, backup.
6. **Non-functional requirements.** Each one gets a number and a verification method:
   - **Performance:** cold start, LCP and INP for web, API p95 and p99, frame rate.
   - **Capacity:** users, requests per second, data volume at launch and at 12 months.
   - **Availability and reliability:** uptime target, RPO, RTO, offline behavior.
   - **Security and privacy:** link SECURITY.
   - **Accessibility:** WCAG 2.2 AA, screen readers, 200% font scaling.
   - **Compatibility:** the OS and browser matrix.
   - **Localization.**
   - **Usability:** for example, "a first-time user completes `<core task>` in under 60 seconds without help."
   - **Size budgets:** web bundle, app download size.
   - **Maintainability and observability:** test coverage floor, crash-free sessions target, logging.
   - **Compliance.**
7. **Traceability:** `US-ID | FR-IDs | Test`.

### `UX.md`

- **Navigation map** (Mermaid flowchart).
- **Screen inventory:** `Screen | Purpose | F and FR IDs | States`. States always cover empty, loading, error, offline, success, and permission denied where relevant.
- **Key flows:** screen sequences for each journey.
- **Design system:** tokens (colors with light and dark values and contrast ratios, type scale, spacing, radius, elevation, motion durations), core components, platform conventions (Material 3, Apple HIG), responsive breakpoints for web.
- **Accessibility rules.**
- **Content:** voice and tone, banned words, error message pattern, empty state pattern.
- **Wireframes:** link Figma if it exists. Otherwise, ASCII wireframes for the three to five core screens.

### `ARCHITECTURE.md`

- **Style** and rationale (link the ADR).
- **System context diagram** (Mermaid): actors, our system, external systems.
- **Container diagram:** apps, services, databases, storage, queues, CDN, third parties.
- **Sequence diagrams** for the top three journeys, including the failure path.
- **Data flow and trust boundaries:** where data enters, where it is validated, where it is stored, what leaves the device.
- **Deployment view:** environments (dev, staging, prod), hosting, regions.
- **Cross-cutting concerns:** auth, offline and sync, caching, error handling, configuration and feature flags, logging and monitoring.
- **Quality attributes:** `NFR-ID | How the architecture meets it`.
- **Scaling path:** what changes at the next tier, and the measurable trigger for each change. `/project-bootstrap` turns these into PLANNED items.

### `TECH_SPEC.md`

- **Stack table:** `Layer | Choice | Version | Why | Rejected alternative`.
- **Repo layout and module boundaries** (annotated tree).
- **Data model:** entities, fields, types, constraints, indexes, relationships. ERD in Mermaid. ID strategy. Migration strategy. The on-device schema too, if the app works offline.
- **API:** style (REST, GraphQL, gRPC, BaaS SDK), auth, versioning, error format, pagination, rate limits, idempotency. An endpoint table, or `api/openapi.yaml` as the contract.
- **Client state:** state management, data fetching, and caching.
- **Offline and sync:** what is cached, the conflict resolution rule, the queue and replay.
- **Third-party services and SDKs:** what each does, what data it receives, its cost at launch.
- **Configuration:** env var names (never values), build flavors.
- **Error handling and logging strategy.**
- **Performance budgets** from the SRS, and how each one is measured.
- **Engineering conventions:** language and lint config, formatting, naming, folder pattern, commit format, branching, code review. `/project-bootstrap` turns these into `CLAUDE.md` rules.
- **Cost estimate:** monthly infrastructure cost at launch and at 10x usage.

### `SECURITY.md`

- **Data inventory:** `Data | Classification (public, internal, PII, sensitive) | Source | Stored where (device, server, third party) | Encrypted at rest / in transit | Retention | Deletion path`.
- **Threat model:** assets, actors, entry points. For each entry point: the top threats (STRIDE) and their mitigations.
- **AuthN and AuthZ:** roles and a permissions matrix.
- **Secrets:** where they live, who can access them, how they rotate.
- **Compliance checklist** for the jurisdictions named in the brief. App store privacy requirements: iOS privacy manifest, Play Data safety, ATT, in-app account deletion.
- **Privacy policy inputs:** what the policy must disclose.
- **Incident response:** contacts and severity levels.

### `TEST_PLAN.md`

- **Strategy:** the test pyramid, with tools per layer.
- **Environments and test data.**
- **Device and browser matrix.**
- **FR coverage:** each FR mapped to a test type and location.
- **NFR verification:** how and when each measurable NFR is checked (CI budget check, Lighthouse, accessibility audit, load test).
- **Release acceptance:** what must pass before any release.
- **Manual QA checklist:** what automation misses, such as backgrounding, a poor network, screen readers, and 200% font.

### `PLAN.md`

- **Milestones:**
  - M0: setup (repo, CI, run `/project-bootstrap`, a skeleton deployed end to end).
  - M1: retire the riskiest technical assumption.
  - M2 to Mn: MVP features in dependency order.
  - Last: launch prep.
- **Per milestone:** goal, feature IDs, exit criteria (checkable), and the verification commands or demo.
- **Tasks** as checkboxes, `- [ ] T-001 <task> (F-01, FR-003) [S]`. Each task fits one working session and appears in dependency order. Size each one S, M, or L.
- **Definition of done**, applying to every task.
- **Risk register:** `R-ID | Risk | Likelihood | Impact | Mitigation | Owner | Status`.
- **No calendar dates** unless I gave a deadline.

### `DECISIONS.md`

One ADR per decision from Gates 1 to 3: `ADR-ID`, title, date, status, context, decision, alternatives considered, consequences, reversal cost. Include every hard boundary and every significant thing we chose not to build.

### `features/F-xx-<slug>.md`

The working spec a single Claude session implements from. Self-contained apart from ID links:

- Goal (one line) and persona.
- User stories: `US-ID: As a <persona>, I want <action>, so that <outcome>.`
- Acceptance criteria per story, as Given / When / Then, including the failure and offline cases.
- FRs and NFRs covered (IDs, not copies).
- UX: screens, states, exact copy strings.
- Data: entities read and written, validation limits.
- Network calls.
- Edge cases.
- Tests to write, mapped to acceptance criteria.
- Out of scope for this feature.
- Open questions.

### `LAUNCH.md` (public products only)

- **Pre-launch checklist:** legal, store, infrastructure, support.
- **Store listing draft:** name, subtitle, description, keywords, screenshot list, category, age rating. For web: domain, SEO, social cards.
- **Privacy policy and terms:** drafts, or the inputs a lawyer needs.
- **Support channel** and contact.
- **First-week watch list** after launch.

---

## PHASE 5: REVIEW

1. **Consistency:** no doc contradicts another on stack, scope, or numbers. List every conflict and fix it.
2. **Traceability:**
   - Every Must feature has stories.
   - Every story has acceptance criteria.
   - Every FR maps to a story and a test.
   - Every NFR has a number and a verification method.

   Print any orphans.
3. **Quality:** no banned vague words, no placeholders, and every unknown appears in Open Questions.
4. **Feasibility:** the MVP fits the team and timeline in the brief. If it does not, say what to cut.
5. **Cold-read test:** spawn a subagent with no context. It reads only `docs/README.md` and what it links to, then answers:
   - What are we building, and for whom?
   - What is in the MVP?
   - What is the stack?
   - What must F-01 do, and how will we know it is done?
   - What is forbidden?
   - What is the first task?

   Any wrong or missing answer is a gap. Fix it and re-run the test.

### CONFIRM GATE 4.

Show:
- The docs created.
- The counts: features, stories, FRs, NFRs, ADRs, tasks, open questions.
- The open questions that block the MVP.

Wait for approval, then set each approved doc's status to `Approved`.

---

## PHASE 6: HAND OFF

1. Print the next step: "Run `/project-bootstrap`. It reads these docs, skips the questions they answer, and sets up `CLAUDE.md`, skills, settings, and infrastructure to match. Then it starts on the first task in `PLAN.md`."
2. Suggest a commit message. Do not commit.

---

## EXECUTION ORDER

1. Phase 0, then print What I found.
2. Round 1 questions, then write BRIEF. **CONFIRM GATE 1.**
3. Round 2 scope. **CONFIRM GATE 2.**
4. Round 3 technical direction. **CONFIRM GATE 3.**
5. Phase 4: generate.
6. Phase 5: review. **CONFIRM GATE 4.**
7. Phase 6: hand off.

Begin with Phase 0.
