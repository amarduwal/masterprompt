---
name: project-bootstrap
description: Set up persistent Claude Code context for a project (CLAUDE.md, path-scoped rules, project skills, settings, hooks, plugins, reviewer agent) plus tier-sized infrastructure, assessed against 111 production concerns. Reads the docs from project-docs. Works for new and existing projects, web or mobile.
argument-hint: "[optional notes]"
disable-model-invocation: true
---

If the user typed anything after the command, treat it as extra context for Phase 0: $ARGUMENTS

# MASTER PROMPT: Bootstrap persistent project context for Claude Code

Run this at the root of any repository: new or existing, web or mobile or both.

**Run order:** `/project-docs` first (brief, PRD, SRS, UX, architecture, tech spec, security, test plan, build plan), then this prompt. This one reads those docs, sets up Claude Code and the infrastructure to build exactly what they describe, then starts on the first task in the plan.

---

## Mission

Set up this repository so every future Claude Code session, and every teammate who clones it, starts with full context: the stack, the rules, the hard constraints, the architecture decisions, the current status, and the workflows. After this runs, nobody re-explains the project. Knowledge lives in git, not in someone's head or a chat history.

You will produce:

1. **Team memory:** `CLAUDE.md`, `.claude/rules/*.md`, `docs/STATUS.md`, `docs/DECISIONS.md`.
2. **Project skills** (slash commands) in `.claude/skills/`.
3. **Shared settings** in `.claude/settings.json`: permissions, hooks, plugins.
4. **A reviewer subagent** in `.claude/agents/`.
5. **Infrastructure and docs**, only for what this project actually needs.

## Ground rules for this run

- **Stop at every CONFIRM GATE and wait for my answer.** Write no files before Gate 2 is approved.
- **Never overwrite an existing file.** Read it, propose a merge, show the diff, wait. Existing CI, Docker, deploy, and lint configs are augmented, never replaced.
- **Describe the project as it is**, not as you'd like it. Anything inferred but unverified is marked `(assumed)` until I confirm it.
- **Size to the tier.** A solo MVP gets no Kubernetes, no Terraform, no service mesh. Every generated file must trace back to an ACTIVE item or a need I confirmed.
- **No secrets anywhere.** Env var names only. Never read the values in `.env` files. Read `.env.example` if it exists.
- **Ask before** `git init`, any commit, any push, installing global tools, installing plugins, adding MCP servers, or adding dependencies.
- **No AI attribution** in commits, PRs, code comments, or generated docs. No `Co-Authored-By`, no "Generated with".
- **No placeholders.** No `TODO: fill in`, no `<your value>`. If a value is unknown, ask, or write the rule without it.
- **Re-runnable.** If this runs on a repo that already has these files, update them in place. Never create duplicates.

---

## PHASE 0: DETECT (silent, no questions yet)

Read before asking anything:

- **Project docs first.** If `docs/README.md` indexes a docs set (`BRIEF`, `PRD`, `SRS`, `UX`, `ARCHITECTURE`, `TECH_SPEC`, `SECURITY`, `TEST_PLAN`, `PLAN`, `features/`), read the README, BRIEF, SRS sections 2 and 6, ARCHITECTURE, TECH_SPEC, SECURITY, and PLAN before anything else. The docs are the source of truth for intent and decisions. The code is the source of truth for what exists. If there are no docs, say so and recommend running `/project-docs` first. Continue without them only if I say so.
- Root listing (two levels deep), `README*`, the rest of `docs/`.
- Manifests and lockfiles: `package.json`, `pnpm-workspace.yaml`, `turbo.json`, `nx.json`, `pubspec.yaml`, `app.json`, `app.config.*`, `eas.json`, `android/`, `ios/`, `*.xcodeproj`, `build.gradle*`, `Podfile`, `go.mod`, `pyproject.toml`, `requirements*.txt`, `Cargo.toml`, `composer.json`, `Gemfile`, `*.csproj`.
- Tooling: lint, format, typecheck, and test configs (`tsconfig*.json`, `analysis_options.yaml`, `eslint.config.*`, `biome.json`, etc.).
- Delivery: `.github/workflows/`, `.gitlab-ci.yml`, `bitbucket-pipelines.yml`, `codemagic.yaml`, `fastlane/`, `Dockerfile*`, `docker-compose*`, `vercel.json`, `netlify.toml`, `wrangler.toml`, `wrangler.jsonc`, `firebase.json`, `supabase/`, `terraform/`, `k8s/`, `helm/`, `charts/`.
- Data: `prisma/`, `drizzle/`, `migrations/`, ORM configs, schema files.
- Existing AI config: `CLAUDE.md`, `CLAUDE.local.md`, `AGENTS.md`, `.claude/`, `.mcp.json`, `.cursorrules`, `.cursor/rules/`, `.github/copilot-instructions.md`, `.windsurfrules`. Their content is input. Fold it in, lose nothing.
- `git log --oneline -30` for commit style, `git branch -a` for branch strategy.

Then print **What I found** (20 lines max): project docs found and their status, every conflict between the docs and the code, new or existing, platforms, language and framework with versions, package managers, monorepo layout, backend (own, BaaS, serverless, none), database, hosting and release targets, CI/CD, tests, conventions observed, existing AI config, and anything that looks broken or contradictory.

An empty repo, or one with only a README, is a new project. Say so and go to Phase 1.

---

## PHASE 1: DISCOVERY

Ask only what Phase 0 and the project docs could not answer. With a full docs set, that is usually only question 14. Number the questions. Next to each, state the default you would use, and tell me I can answer `default` to any of them (production defaults for a small team shipping a real product).

1. **Name and one-sentence description.**
2. **Platform:** web (SPA, SSR, static, PWA), mobile (iOS, Android, or both; native, Flutter, React Native/Expo), both, or backend/API only. If both: shared codebase or separate?
3. **Language, framework, versions.** For a new project, suggest a stack with one line of reasoning if I'm unsure.
4. **Backend:** own service, BaaS (Firebase, Supabase, Appwrite), serverless functions, or none (static content only)?
5. **Scale tier now:**
   - **T1 Solo/MVP:** 1-3 devs, under 1k users, single region, no compliance burden
   - **T2 Growth:** 3-10 devs, 1k-100k users, uptime starts to matter
   - **T3 Scale:** 10+ devs, 100k+ users, multi-region, compliance (SOC 2, HIPAA, PCI)
6. **Data sensitivity:** PII, health, financial, children's data, location? Which jurisdictions (GDPR, CCPA, COPPA, HIPAA, PCI)?
7. **Offline:** must any part work with no network? Which part?
8. **The number-one product constraint:** the one thing every decision protects. Example: "a user can open the app with no account and no signal and start in under five seconds."
9. **Hard boundaries:** what is forbidden in code, copy, or features. Examples: "no health claims", "no user accounts", "no product analytics".
10. **Budgets:** startup time, bundle or app size, API latency.
11. **Existing infra:** cloud provider, database, hosting, CI, Docker, error tracking?
12. **Team conventions:** commit format, branch strategy, code style, writing style. Default: Conventional Commits, trunk-based with short-lived branches, plain writing with no em dashes and no AI tells.
13. **Release:** app stores, web host, cadence, who is allowed to ship.
14. **Claude Code setup:**
    - a. Does the team also use Cursor, Codex, or Copilot? If yes, instructions go in `AGENTS.md` and `CLAUDE.md` imports it.
    - b. MCP servers to share with the team (GitHub, Sentry, database, Figma, Linear)?
    - c. Plugins or third-party skills you already use?

### CONFIRM GATE 1. Wait for answers.

---

## PHASE 2: ASSESS

Classify every item in the **REFERENCE LIST** (1-111) and the **CLIENT LIST** (C1-C26) into exactly one tier:

- **ACTIVE:** implement now. Name the specific choice and where it lives (file, package, service).
- **MANAGED:** a provider handles it (Cloudflare DDoS, Vercel TLS, Supabase backups, store CDNs). Name the provider and what we still own (config, limits, a toggle).
- **PLANNED:** not now. Give a measurable trigger: "p95 API latency over 400 ms", "a second backend service", "50k DAU", "first customer asks for SOC 2".
- **EXCLUDED:** does not apply. One-sentence reason. Future sessions must not suggest it without new information.

With project docs: take thresholds from the SRS non-functional requirements and PLANNED triggers from the scaling path in `ARCHITECTURE.md`, and cite the NFR or ADR ID in the decision column.

**Evaluate each item from the client's side too.** Most of the 111 are written for backends, but many have a client form:

- **Retries, backoff, idempotency:** retry only idempotent calls, with jittered backoff; send idempotency keys on writes.
- **Rate limiting:** honor `429` and `Retry-After`; debounce user-triggered requests.
- **Caching and invalidation:** HTTP cache, local DB, ETags, versioned manifests.
- **Rollbacks and canary on mobile:** a shipped binary cannot be rolled back. Use staged rollout (Play) and phased release (App Store), a remote kill switch, a minimum-supported-version gate, and OTA updates where the platform allows them.
- **Feature flags:** remote config with safe bundled defaults that work offline.
- **Clock skew:** never trust the device clock for anything the server decides. Store time zones with timestamps.
- **Network partitions:** offline queue with replay and explicit conflict rules.
- **Memory leaks, thread safety:** dispose listeners, controllers, subscriptions; keep heavy work off the UI thread.
- **Cold starts:** app cold start and serverless cold start are both real budgets.
- **TLS:** no cleartext traffic (iOS ATS, Android network security config).
- **JWT rotation:** refresh tokens in Keychain/Keystore, never in plain storage or `localStorage`.
- **Health checks, probes, service discovery, leader election:** EXCLUDED for a client app with no backend we operate.

### REFERENCE LIST (111 items)

**Networking and Traffic:** 1 Rate Limiting, 2 Load Balancing, 3 Reverse Proxies, 4 API Gateways, 5 DNS, 6 TCP vs UDP, 7 HTTP/2 and HTTP/3, 8 gRPC, 9 CORS, 10 CDN, 11 Edge Caching

**Resilience:** 12 Circuit Breakers, 13 Timeouts, 14 Retries, 15 Exponential Backoff, 16 Idempotency, 17 Dead Letter Queues, 18 Backpressure

**Messaging and Real-Time:** 19 Message Queues, 20 Pub/Sub, 21 Event-Driven Architecture, 22 Webhooks, 23 WebSockets, 24 Long Polling, 25 Server-Sent Events

**Consistency and Concurrency:** 26 Distributed Transactions, 27 Saga Pattern, 28 CAP Theorem, 29 Eventual Consistency, 30 Optimistic Locking, 31 Pessimistic Locking, 32 Distributed Locks, 33 Race Conditions, 34 Deadlocks

**Database and Storage:** 35 Indexing, 36 Query Optimization, 37 N+1 Queries, 38 Connection Pooling, 39 Read Replicas, 40 Sharding, 41 Partitioning, 42 Replication, 43 Database Migrations, 44 Schema Versioning

**Caching:** 45 Caching, 46 Cache Invalidation, 47 Build Caching

**Compute and Runtime:** 48 Memory Leaks, 49 Garbage Collection, 50 Thread Safety, 51 Cold Starts, 52 Serverless Limits

**Scaling:** 53 Autoscaling, 54 Horizontal Scaling, 55 Vertical Scaling

**Deployment and Release:** 56 CI/CD, 57 Docker, 58 Kubernetes, 59 Service Discovery, 60 Blue-Green Deployment, 61 Canary Releases, 62 Rolling Deployments, 63 Rollbacks, 64 Feature Flags

**Observability:** 65 Health Checks, 66 Liveness and Readiness Probes, 67 Monitoring, 68 Logging, 69 Distributed Tracing, 70 Metrics, 71 Alerts, 72 SLOs, 73 SLIs, 74 Error Budgets, 75 Observability

**Security:** 76 Secrets Management, 77 IAM, 78 OAuth, 79 JWT Rotation, 80 TLS, 81 Encryption at Rest, 82 Encryption in Transit, 83 WAF, 84 DDoS Protection, 85 CSRF, 86 SQL Injection, 87 XSS, 88 SSRF

**Reliability and DR:** 89 Disaster Recovery, 90 Backups, 91 Failover, 92 Multi-Region Deployment, 93 Chaos Engineering, 94 Leader Election

**Performance:** 95 Latency, 96 Throughput, 97 P99 Latency, 98 Tail Latency

**Infrastructure:** 99 Infrastructure as Code, 100 Terraform, 101 Helm Charts

**Operations:** 102 Cron Jobs, 103 Cost Optimization, 104 Production Incidents, 105 On-Call, 106 Postmortems

**API Design:** 107 API Versioning, 108 Semantic Versioning

**Dependencies:** 109 Dependency Hell

**Network Failure Modes:** 110 Network Partitions, 111 Clock Skew

### CLIENT LIST (web and mobile concerns the reference list misses)

C1 Offline support and sync conflict rules, C2 On-device and secure storage (Keychain, Keystore, no secrets in the bundle), C3 Bundle and app size budgets, C4 Startup time and Core Web Vitals (LCP, INP, CLS), C5 Accessibility (WCAG 2.2 AA, screen readers, font scaling, reduced motion), C6 Internationalization and localization, C7 Deep links and universal/app links, C8 Push notifications, C9 App store policy compliance (iOS privacy manifest, Play Data safety, ATT, in-app account deletion), C10 Forced update, minimum version, kill switch, C11 OTA updates, C12 Crash reporting and error tracking, C13 Product analytics and consent, C14 Privacy policy and user data export/deletion, C15 Certificate pinning, C16 Content Security Policy and security headers, C17 SEO and social metadata, C18 Rendering strategy (CSR, SSR, SSG, ISR) and hydration, C19 Service worker and PWA caching, C20 Image, font, and media optimization, C21 Dependency updates and supply chain (lockfiles, Renovate/Dependabot, audits), C22 Test strategy (unit, component/widget, e2e with Playwright, Maestro, Patrol, or Detox), C23 Design tokens, theming, dark mode, C24 Environments and build flavors (dev, staging, prod), C25 Code signing and release keys, C26 Background execution limits, battery, and data use

### Output

1. One table per category: `# | Item | Tier | Decision, provider, trigger, or reason | Where it lives`.
2. A count table: category by tier, with totals.
3. **The generation plan:**
   - Every file to create, and every existing file to modify (one line on what changes).
   - The skills from 3.4 you will generate, and any you will drop with the reason.
   - Plugins, third-party skills, and MCP servers you propose: five at most, one line of justification each, with the exact install command. Only propose ones you have confirmed exist (marketplace listing or the repo's README). "None" is a valid answer.
   - New dependencies, each justified against the ladder in section 3 of `CLAUDE.md`.

### CONFIRM GATE 2. Wait for corrections and approval. Apply corrections and re-show only what changed.

---

## PHASE 3: GENERATE

### 3.1 Where knowledge lives

Each fact goes in exactly one place. Duplicated rules drift apart.

| Layer | File | In git | Loaded |
|---|---|---|---|
| Core rules | `CLAUDE.md` (or `AGENTS.md`, with `@AGENTS.md` as line 1 of `CLAUDE.md`, if the team uses other AI tools) | yes | every session |
| Topic and path rules | `.claude/rules/*.md` | yes | every session, or only when matching files are touched (`paths:`) |
| Current status | `docs/STATUS.md` (current milestone and next tasks from `docs/PLAN.md`), imported by `CLAUDE.md` | yes | every session |
| Product and engineering docs | `docs/README.md` (index), `BRIEF`, `PRD`, `SRS`, `UX`, `ARCHITECTURE`, `TECH_SPEC`, `SECURITY`, `TEST_PLAN`, `PLAN`, `features/` | yes | on demand (linked) |
| Decisions | `docs/DECISIONS.md` | yes | on demand (linked) |
| Assessment | `docs/INFRASTRUCTURE.md` | yes | on demand (linked) |
| Workflows | `.claude/skills/<name>/SKILL.md` | yes | when invoked |
| Reviewer | `.claude/agents/reviewer.md` | yes | when delegated |
| Permissions, hooks, plugins | `.claude/settings.json` | yes | every session |
| Shared MCP servers | `.mcp.json` | yes | every session, after each person approves |
| Personal overrides | `CLAUDE.local.md`, `.claude/settings.local.json` | **no** (gitignored) | every session, for that person only |

Claude's auto memory (`~/.claude/projects/.../memory/`) is local to one machine and never reaches teammates. Anything a teammate needs goes in the files above.

### 3.2 `CLAUDE.md`

**Hard limit: 200 lines.** It loads into every session, so every line has a cost. Content needed only for some files goes in a path-scoped rule. Long reference material goes in `docs/` and is linked by path, not imported.

Put `@docs/STATUS.md` near the top (plus `@AGENTS.md` on line 1 if 14a was yes). Imports are ignored inside backticks and code fences, so write them as bare lines.

With project docs, `CLAUDE.md` turns them into rules and links to them. It never copies them. The number-one constraint and boundaries come from BRIEF, budgets from the SRS NFRs, stack and conventions from TECH_SPEC. Put the source ID next to each rule (`(NFR-004)`, `(ADR-002)`) so a session can look up the reasoning.

Sections, in this order. Omit a section only if it truly does not apply:

- **0. Role and mindset.** The senior role per platform, naming the real stack. Production over convenience. Existing conventions over personal preference. The number-one constraint (Q8), written as a sentence every decision gets tested against.
- **1. Project context.** Name, one-liner, platforms and release order. Stack with versions and real package names. Package managers and where each applies ("never npm inside `app/`"). Architecture in two lines. What exists and what deliberately does not (no backend? no admin panel?). Docs index: `docs/README.md`.
- **2. Workflow.** State the assumption and proceed; ask only when genuinely ambiguous. Large work in steps. After changes: what changed, why, and the exact commands to verify it. Never claim success without running the checks. Before a task, read its feature spec (`docs/features/F-xx-*.md`) and the FR and NFR IDs it cites. A change in behavior updates that spec in the same commit. At session end or task completion, run `/handoff`.
- **3. Code quality.** Where shared code lives. Folder convention (feature-first or layer-first, whichever the repo already uses). Naming. No dead code, no commented-out blocks. The dependency ladder: does it need to exist, is it already here, does the stdlib or platform do it, can an installed package do it, can it be one line. Type safety rules for this language.
- **4. Hard boundaries.** From Q6, Q8, Q9. Each a flat "never", with a pointer to its `DECISIONS.md` entry. "If a task crosses one, stop and cite the decision before writing anything."
- **5. Security and privacy.** Secrets policy. Input validation with real limits (lengths, ranges). What must never appear in logs, URLs, crash reports, or analytics. The complete inventory of outbound network calls ("adding one is a decision: flag it first"). Minimal OS and cloud permissions.
- **6. Git.** Commit format with two realistic examples. Branch strategy. One logical change per commit. No AI attribution. Never force-push a shared branch.
- **7. Testing and verification.** Test priority (pure logic, then components, then e2e/device). The done checklist as exact commands (lint, typecheck, test, build). Platform checks: light and dark, 200% font scale, screen reader, small screen. Mobile: verify background behavior by actually backgrounding the app.
- **8. Performance.** Hard budgets as numbers. What to pre-warm, what to lazy-load. Measure before optimizing.
- **9. Scope.** Do what is asked. No unrelated refactors or restyling. Out-of-scope bugs get flagged in the summary, not fixed inline. No unrequested features, dependencies, or config.
- **10. Errors and UX.** Degrade, don't block. Plain user-facing errors, no codes or stack traces. No silent catches. Confirm and offer undo for destructive actions.
- **11. Accessibility.** Labels, traversal order, AA contrast, touch targets (44pt iOS, 48dp Android, 24px web minimum), reduced motion, font scaling.
- **12. Writing style.** Punctuation and tone rules for code comments, commits, docs, and UI copy.
- **13. Infrastructure.** ACTIVE and MANAGED items, one line each with location. Then: "PLANNED and EXCLUDED items are in `docs/INFRASTRUCTURE.md`. Never propose an EXCLUDED item without new information. When a PLANNED trigger is met, run `/scale-up`."
- **14. Layout.** Annotated tree, two or three levels, real paths.
- **15. Conventions.** Shared vs feature boundaries. Storage rules (what goes where, and only there). ID stability. Derived vs stored data. Checksums and integrity. Copy tone.
- **16. Environment.** Fresh-clone setup commands. Env var names (never values) and where each is used. Secrets manager. How a new developer gets credentials. Plugin install commands.
- **17. Keeping this file true.** "When I correct you on something that will come up again, propose a one-line addition to the right file (`CLAUDE.md`, a rule, or `DECISIONS.md`) in the same reply. Record decisions with `/decision` as they are made. Never keep team knowledge only in auto memory."

**Quality bar.** Match the tone and specificity of these lines. Do not copy their content:

- "State: `flutter_riverpod`, no codegen." Not "use a state management library."
- "Cold start under 2 seconds on a mid-range Android device, profile build, airplane mode." Not "keep startup fast."
- "Crash reporting: Sentry. No product analytics of any kind."
- "Before calling anything done: `flutter analyze` clean, `flutter test` green, `flutter build apk --debug` succeeds."
- "A dotLottie file over 120 KB is a production defect, not a performance nit."
- "The streak is derived from session history at read time, never stored. If you find one being written, that is a bug."
- "Implement what the task asks. Do not refactor, restyle, or improve unrelated code."

Banned in generated rules: "consider", "try to", "where possible", "best practices", "as needed". Every rule must be checkable.

### 3.3 `.claude/rules/`

```markdown
---
paths:
  - "apps/mobile/**/*.{dart,yaml}"
---
# Mobile rules
...
```

`paths` is the only frontmatter field rules read. A rule with no `paths` loads every session, so keep unscoped rules few and short. Create only what applies:

- `security.md` (unscoped): the ACTIVE security items as rules.
- `<platform>.md`, one per app or package, scoped to its folder: framework idioms, state, storage, navigation, platform checks.
- `backend.md`, scoped to server paths: timeouts on every outbound call, retry policy, idempotency, rate limits, log fields, health endpoint contract.
- `database.md`, scoped to schema, migration, and query paths: migration rules, indexing, N+1, locking strategy.
- `testing.md`, scoped to test paths.
- `infra.md`, scoped to CI, Docker, and IaC paths.

### 3.4 Skills: `.claude/skills/<name>/SKILL.md`

**A project skill replaces any built-in command with the same name.** Never use: `init`, `review`, `security-review`, `code-review`, `simplify`, `verify`, `run`, `memory`, `config`, `plan`, `status`, `context`, `doctor`, `hooks`, `agents`, `skills`, `team-onboarding`, `loop`, `schedule`, `batch`. Run `/help` and check every name against the full list before creating it.

Template:

```markdown
---
name: new-feature
description: Scaffold a feature folder following this project's conventions. Use when starting a new feature or screen.
argument-hint: <feature-name>
disable-model-invocation: true
---
# /new-feature

Arguments: $ARGUMENTS

Read first: CLAUDE.md sections 3, 14, 15.

Steps:
1. (exact commands for this repo's stack)

Output: (what, and where)

Stop and ask if: (conditions)
```

Every skill uses the exact commands for this repo (not generic ones), references `CLAUDE.md` sections by number, and states its outputs and file paths. Set `disable-model-invocation: true` on anything that writes beyond the task, deploys, or touches infrastructure.

| Skill | Does | Claude may auto-invoke |
|---|---|---|
| `bootstrap` | This prompt, condensed: re-detect, diff against `docs/INFRASTRUCTURE.md`, propose updates, gate, apply. | no |
| `next-task` | Pick the first unchecked task in `docs/PLAN.md` whose dependencies are done, confirm it with me, implement it from its feature spec, run the section 7 checks, then `/handoff`. | no |
| `handoff` | End of session: tick finished tasks in `docs/PLAN.md`, update `docs/STATUS.md` (done, in progress, next, blockers; under 40 lines), record decisions made, flag any doc that no longer matches the code, list uncommitted work, suggest a commit message. | no |
| `decision` | Append an ADR to `docs/DECISIONS.md`: id, date, context, decision, consequences, reversal cost. | **yes** |
| `new-feature` | For a feature not yet in the docs: write `docs/features/F-xx-<slug>.md` using the same template as the existing specs, add it to the PRD feature table and `PLAN.md`, wait for approval, then scaffold code and test files per sections 3, 14, 15. | no |
| `rules-check` | Review `git diff HEAD` against `CLAUDE.md` and `.claude/rules/` using the `reviewer` subagent. Violations by section number with `file:line`. | no |
| `infra-audit` | Check every ACTIVE and MANAGED item against the actual code and config. Report drift and gaps. | no |
| `security-audit` | Whole-repo audit of items 76-88 and C2, C9, C14, C15, C16: hardcoded secrets (`gitleaks` if installed, otherwise grep patterns), injection, XSS, SSRF, CORS, CSRF, TLS, and this stack's dependency audit command. Built-in `/security-review` only covers pending changes; this covers the whole repo. | no |
| `perf-audit` | Measure the section 8 budgets: bundle or app size, N+1 queries, missing indexes, cold start, Core Web Vitals. Numbers, not opinions. | no |
| `scale-up` | For a PLANNED item whose trigger is met: show evidence the trigger fired, write the ADR, implement, move the item to ACTIVE everywhere it is listed. | no |
| `incident` | Create `docs/incidents/YYYY-MM-DD-<slug>.md` (timeline, impact, detection, mitigation), run this project's health and log checks, then a blameless postmortem with action items and owners. | no |
| `release` | Pre-release checklist, version bump (semver, plus build number on mobile), changelog since the last tag, this project's build and submit/deploy commands, rollback steps. Never tags, pushes, or uploads without asking. | no |
| `onboard` | Only if built-in `/team-onboarding` is unavailable: write `ONBOARDING.md` from `CLAUDE.md`, the repo, and the setup commands. | no |

Drop any skill whose job does not exist in this project yet (for example, `scale-up` when nothing is PLANNED).

### 3.5 `.claude/agents/reviewer.md`

```markdown
---
name: reviewer
description: Read-only reviewer. Checks a diff against CLAUDE.md and .claude/rules. Use for /rules-check and before calling a task done.
tools: Read, Grep, Glob, Bash
model: inherit
---
```

Body: never edits files; uses Bash only for `git diff`, `git log`, and the read-only checks. Checks hard boundaries, security, scope creep, budgets, missing tests, and conventions. Output per finding: severity, `CLAUDE.md` section, `file:line`, a one-line fix.

### 3.6 `.claude/settings.json`

```json
{
  "permissions": {
    "allow": [
      "Bash(git status *)",
      "Bash(git diff *)",
      "Bash(git log *)"
    ],
    "deny": [
      "Read(./.env)",
      "Read(./.env.local)",
      "Read(./.env.*.local)",
      "Read(./.env.production)",
      "Read(./secrets/**)",
      "Read(**/*.jks)",
      "Read(**/*.keystore)",
      "Read(**/*.p12)",
      "Read(**/*.p8)",
      "Read(**/*.mobileprovision)",
      "Bash(git push --force *)",
      "Bash(git push -f *)"
    ]
  },
  "hooks": {
    "PostToolUse": [
      {
        "matcher": "Edit|Write",
        "hooks": [{ "type": "command", "command": "<format the edited file>" }]
      }
    ]
  }
}
```

- **allow:** add this project's real lint, typecheck, test, format, and build commands in the form `Bash(npm run test *)`. Never allow push, deploy, publish, migrations against a real database, or anything destructive.
- **deny:** deny wins over allow, so `Read(./.env.*)` would also block `.env.example`. List the real env files instead. A `Read` deny also blocks editing and shell reads of that path. Add the signing and credential files this stack actually has.
- **Format hook:** reads the JSON on stdin, takes `tool_input.file_path`, runs this stack's formatter on that one file only when the extension matches, and always exits 0 so a formatter failure never blocks work. If `jq` might be missing on teammates' machines, put the logic in `.claude/hooks/format.sh` instead of inline.
- **Plugins approved at Gate 2:** add `"enabledPlugins": { "<plugin>@<marketplace>": true }`, plus `extraKnownMarketplaces` for any marketplace other than the official `claude-plugins-official`. Committing this does not download anything, so section 16 of `CLAUDE.md` and the README must list the per-person install command: `claude plugin install <plugin>@<marketplace> --scope project`.
- **Vendor skills:** if the stack's vendor publishes official agent skills (some do, installable with `npx skills add <owner>/<repo>`), propose them at Gate 2 and install them at project scope so they are committed.

### 3.7 `.mcp.json`

Only servers approved at Gate 2. Credentials through `${VAR}` expansion, never inline. Add each variable to `.env.example` and section 16.

### 3.8 `.gitignore`

Add `CLAUDE.local.md`, `.claude/settings.local.json`, the real env files, signing files, and the stack's standard ignores. Merge with the existing file; don't reorder it.

### 3.9 Infrastructure (ACTIVE items only, sized to tier)

**Every project:**
- CI on every PR: install, lint, typecheck, test, build. Cache dependencies and build outputs. Pin action versions.
- `.env.example` with names and a one-line comment each.
- Renovate or Dependabot config, if C21 is ACTIVE.

**Web:** security headers and CSP in the host config (C16), a bundle size check in CI (C3), preview deploys if the host supports them, a `Dockerfile` only if self-hosted.

**Mobile:** build flavors for dev, staging, prod (C24); signing through CI secrets, never in the repo (C25); store builds in CI with fastlane, EAS, or Codemagic, whichever fits the stack; crash reporting set up (C12); remote config with bundled defaults and a minimum-version gate (C10); iOS privacy manifest and Play Data safety notes (C9).

**Own backend, T1:** multi-stage `Dockerfile`, `docker-compose.yml` for local dependencies, migration tooling, a `/health` endpoint, timeouts on every outbound call, rate limiting middleware, structured logs with a request id.

**T2 adds:** deploy workflow (staging, then production behind manual approval), reverse proxy or gateway config, cache layer, metrics and alerts, an SLO and error budget doc, backups with a tested restore.

**T3 adds:** IaC (`terraform/`), `k8s/` or Helm charts, OpenTelemetry tracing, a multi-region and failover plan, chaos tests, on-call rotation and SLO dashboards.

### 3.10 Docs

If `/project-docs` already produced `ARCHITECTURE.md`, `SECURITY.md`, or `DECISIONS.md`, update those files. Never create parallel versions. Add ADRs only for decisions not already recorded, and continue the existing numbering.

- `docs/STATUS.md`: current milestone from `PLAN.md`, done, in progress, next, blockers. Under 40 lines. Imported by `CLAUDE.md`.
- `docs/DECISIONS.md`: ADR log. Seed it from this run: the stack, the tier, every hard boundary, and every EXCLUDED item worth defending. Each entry: id, date, context, decision, consequences, reversal cost.
- `docs/INFRASTRUCTURE.md`: the full Phase 2 tables.
- `docs/ARCHITECTURE.md`: text diagram, data flow, trust boundaries, external services. Only if there is more than one moving part.
- `docs/RUNBOOK.md`: deploy, rollback, rotate a secret, restore a backup, with real commands. Only if something is deployed.
- `docs/SECURITY.md`: threat model (assets, actors, entry points) and data inventory (what is stored where, how it is encrypted, how long it is kept). Required if Q6 named any sensitive data.

### 3.11 New projects only

- Scaffold with the framework's official generator (`npm create vite@latest`, `flutter create`, `npx create-expo-app`, and so on), then shape it to the section 14 layout. Don't hand-write what a generator produces.
- `README.md`: one-liner, setup, commands, plugin install commands, links to `CLAUDE.md` and `docs/`.
- Ask before `git init` and before the first commit.

---

## PHASE 4: VERIFY

1. `CLAUDE.md` is under 200 lines. No section contradicts another or a rules file. No fact lives in two places. No placeholders, no banned vague words.
2. Every ACTIVE item maps to a file, config, or rule. Every generated file maps to an ACTIVE item or a confirmed need.
3. Every skill references real `CLAUDE.md` sections and real paths. Run each read-only command in each skill and confirm it works in this repo.
4. `jq . .claude/settings.json` parses (and `.mcp.json` if present). Edit one file to trigger the format hook and confirm it ran. Try to read `.env` and confirm it is denied.
5. Existing projects: run lint, typecheck, test, and build, and report the real output. Nothing that passed before may fail now.
6. Grep every generated file for anything that looks like a key, token, or password.
7. **Cold-read test:** spawn a subagent with no context. It reads only `CLAUDE.md` (and what it imports) and answers: What is this project? How do I run the tests? What is the number-one constraint? Where does new feature code go? Where is the spec for the next task in `PLAN.md`? Name three forbidden things. What should I read before starting work? Any wrong or missing answer means `CLAUDE.md` has a gap. Fix it and re-run the test.

---

## PHASE 5: REPORT

1. Count table: category by ACTIVE, MANAGED, PLANNED, EXCLUDED, with totals.
2. Every file created or modified, one line each.
3. The five PLANNED triggers most likely to fire first.
4. **How the team uses this:** each skill with one line, the plugin install commands, and what belongs in `CLAUDE.local.md`.
5. A suggested commit message. Do not commit.
6. Offer to start the first unchecked task in `docs/PLAN.md`, following the `/next-task` flow.

---

## EXECUTION ORDER

1. Phase 0, then print What I found.
2. Phase 1 questions.
3. **CONFIRM GATE 1.**
4. Phase 2 tables and generation plan.
5. **CONFIRM GATE 2.**
6. Phase 3, then Phase 4, then Phase 5.
7. Offer to start the first task from `docs/PLAN.md`.

Begin with Phase 0.
