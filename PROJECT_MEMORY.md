# ILLUMATRIX V.2 — Living Project Memory & Handoff Record

> **Purpose:** Living state record for ILLUMATRIX V.2. Updated after every meaningful task.  
> **Security Rule:** Never store cleartext secrets, passwords, or tokens in this file.

---

## 1. Current State
- **Current Phase**: Phase 1 — Stabilize the Existing Workflow & Invariants
- **Current Task**: Task 1.4A Completed & Verified (Provider Execution & Cost Policy Audit)
- **Status**: Audit completed for provider execution engine, backend cost policies, Auto Free selection, BYOK behavior, and fallback handling. Identified 4 key gaps (server-side cost policy engine unimplemented, failover unaware of cost tiers, connection testing issuing generative inference, and silent fallback on non-matching connection ID). Existing credentials, SSRF protection, and zero-secret export policies verified secure. 133 automated tests passing.
- **Last Verified Working State**:
  - `npm test`: 133 passing tests across 10 suites (duration ~3s, 0 failures, 0 skipped).
  - `npm run lint`: Clean TypeScript compile (`tsc --noEmit`, 0 diagnostics).
  - `npm run build`: Production bundle built cleanly via Vite (`vite build`).
  - Full-stack dev server: Running Express + Vite on `http://0.0.0.0:3000`.

---

## 2. Completed and Verified Work

### Task 0.1: Baseline Codebase Audit
- **Files Inspected**: `package.json`, `server.ts`, `src/App.tsx`, `src/context/WorkflowContext.tsx`, `src/data/registries.ts`, `src/types/hud.ts`, `src/server/providerManager.ts`, `src/server/modelCompatibility.ts`, `src/services/aiService.ts`, `src/services/storage.ts`, `src/services/promptSafetyEngine.ts`, `tests/*`.
- **Verification Commands & Real Results**:
  - `npm test`: 84 passing subtests.
  - `npm run lint`: Clean (`tsc --noEmit`).
  - `npm run build`: Clean (`vite build`).

### Task 0.2: Synchronization of Seven Root Documentation Files
- **Files Created / Synchronized**: `README.md`, `PROJECT_REQUIREMENTS.md`, `ARCHITECTURE.md`, `FEATURES.md`, `IMPLEMENTATION_PLAN.md`, `DESIGN_SYSTEM.md`, `PROJECT_MEMORY.md`.

### Task 0.3: Credential Security & Gitignore Hardening
- **Objective**: Eliminate hardcoded secrets from stored configuration, create root `.gitignore`, dynamically load Google GenAI credentials from `process.env.GEMINI_API_KEY`, prevent silent credential collision across Google connections, and test missing key states.
- **Files Changed**:
  - `/.gitignore` (Created: ignores `.env*`, `.provider_connections.json`, `dist/`, `node_modules/`, while allowing `!.env.example`).
  - `/.provider_connections.json` (Sanitized: all hardcoded API keys removed; stored `apiKey` set to empty string `""`).
  - `/src/server/providerManager.ts` (Updated: added `resolveEffectiveApiKey()`, updated `loadConnections()` to never write environment secrets to disk, updated `saveConnections()` to strip environment variables prior to serializing, updated `getPublicConnections()`, `getEffectiveCredentials()`, and `executeTextGeneration()` to read environment keys dynamically with safe `unconfigured` states when missing).
  - `/tests/api-provider-manager.test.ts` (Added 5 automated tests for dynamic environment key loading, independent Google connection credentials, missing credential error handling, zero on-disk environment secrets, and `.gitignore` integrity).
- **Verification Commands & Real Results**:
  - `npm test`: 90 passing tests, 0 failures (~3.5s).
  - `npm run lint`: 0 diagnostics (`tsc --noEmit`).
  - `npm run build`: Build succeeded (`vite build`).

### Task 0.4: Custom Provider SSRF Protection
- **Objective**: Implement comprehensive server-side SSRF validation and safe HTTP dispatch for user-configured custom provider endpoints (`baseUrl`).
- **Files Changed / Created**:
  - `/src/server/urlSafetyValidator.ts` (Created: implements `isRestrictedIpv4`, `parseIpv6Words`, `isRestrictedIpv6`, `isRestrictedIp`, `isRestrictedHostname`, `defaultDnsResolver`, `validateCustomProviderUrl`, and `safeSsrfFetch` with manual redirect handling and per-hop revalidation).
  - `/src/server/providerManager.ts` (Updated: imports SSRF tools, adds `SSRF_BLOCKED` to `errorCategory`, validates endpoints in `testConnection()` and `executeTextGeneration()`, dispatches via `safeSsrfFetch`, preserves built-in providers, and rethrows `SSRF_BLOCKED` errors).
  - `/server.ts` (Updated: validates `baseUrl` against SSRF in `app.post('/api/providers')`, returning HTTP 400 with `category: 'SSRF_BLOCKED'`).
  - `/tests/api-provider-manager.test.ts` (Added 12 comprehensive automated test cases covering public URLs, localhost/loopback IPv4 & IPv6, link-local & cloud metadata, private RFC 1918 & CGNAT ranges, IPv6 ULA & IPv4-mapped, protocol & credential rejections, DNS resolution, split-horizon/mixed DNS answers, redirect re-validation, `testConnection()`, `executeTextGeneration()`, and built-in integration preservation).
- **Verification Commands & Real Results**:
  - `npm test`: **103 passing tests across 8 suites, 0 failures, 0 skipped** (~10.0s).
  - `npm run lint`: **0 diagnostics** (`tsc --noEmit`).
  - `npm run build`: **Build succeeded** (`vite build`).
  - `curl -s http://localhost:3000/api/ai/health`: HTTP 200 OK.
- **Architectural Notes & Limitations**:
  - Node.js global `fetch` does not support custom socket dialers or per-request IP pinning without custom `undici.Agent` dispatchers.
  - ILLUMATRIX enforces the strongest safe solution for this architecture:
    1. Pre-flight multi-answer DNS lookup checking *every* resolved IP address.
    2. Forced `redirect: 'manual'` preventing automated unvalidated redirects.
    3. Per-hop URL parsing and SSRF re-validation before following any redirect (up to 3 hops).
    4. Origin-change Authorization header scrubbing on redirect.

### Task 1.2: End-to-End Workflow Regression Suite
- **Objective**: Create and harden deterministic regression tests covering all seven workflow stages (Setup & Rules, Source Library, Script Studio, Scene Prompts, SEO Package, Thumbnail Studio, Review & Export).
- **Files Changed**:
  - `/tests/e2e-workflow-regression.test.ts` (Expanded to 16 comprehensive tests covering: stage status computation, format/duration invariants, source text fidelity & soft-deletion, script breakdown & approval rollback, visual style regeneration byte-for-byte invariant, A/B SEO metadata & timestamp alignment, dual thumbnail options & format responsiveness, readiness audit blocker/warning evaluations, stage-to-stage navigation persistence, backup export/restore roundtrip, single deliverable regeneration isolation, cascading invalidation matrix, safe error handling, prerequisite gating & unsaved edits protection, long-form duration pacing for `[3, 4, 5, 8, 10]` minutes, Hindi Devanagari language consistency, and complete 6-format export verification with strict zero-secret audit).
  - `/src/services/storage.ts` (Fixed generic conditional return type for `loadActiveScriptVersion` supporting fallback objects in testing).
- **Verification Commands & Real Results**:
  - `npm test`: **119 passing tests across 9 suites, 0 failures, 0 skipped** (~2.9s).
  - `npm run lint`: **0 diagnostics** (`tsc --noEmit`).
  - `npm run build`: **Build succeeded** (`vite build`).
  - Dev server HTTP checks: `http://localhost:3000/api/ai/health` (HTTP 200 OK) and `http://localhost:3000/` (HTTP 200 OK).

### Task 1.3A: SSRF DNS Rebinding Risk Assessment
- **Objective**: Conduct formal security audit of custom-provider endpoint handling (`src/server/urlSafetyValidator.ts`, `providerManager.ts`, `server.ts`) for DNS rebinding and TOCTOU vulnerabilities.
- **Findings & Verdict**:
  - **Verdict**: **VULNERABLE to DNS Rebinding / TOCTOU IP Pinning Bypass**.
  - **Mechanisms Identified**:
    1. `safeSsrfFetch` calls `validateCustomProviderUrl` at $T_0$, which validates all IPs returned by `dns.promises.lookup`.
    2. However, at $T_1$, `safeSsrfFetch` dispatches via Node global `fetch(currentUrl)`, which executes an independent, unpinned DNS lookup through Node's internal network stack.
    3. An attacker with a 0-TTL or split-response DNS server can return a public IP at $T_0$ (passing validation) and a prohibited IP (e.g. `169.254.169.254` or `127.0.0.1`) at $T_1$, bypassing the check entirely.
    4. Minor gap: Deprecated IPv4-compatible IPv6 (`::127.0.0.1`) is not checked in `isRestrictedIpv6`.
    5. Denial-of-service exposure: `safeSsrfFetch` lacks a default timeout and response body byte cap.
- **Remediation Path Designed**: Pinned-socket transport utilizing native Node `node:https` and `node:http` with custom `lookup` or agent-level IP pinning, strictly preserving TLS SNI and certificate hostname verification.
- **Status**: **Assessment Completed & Verified.**

### Task 1.3B: SSRF Transport Hardening & Deterministic IP Pinning (Completed & Verified)
- **Objective**: Remediate SSRF DNS rebinding and TOCTOU vulnerabilities by implementing deterministic socket-level IP pinning, transport resource limits, and IPv4-compatible IPv6 filtering.
- **Implementation Highlights**:
  - `src/server/urlSafetyValidator.ts`:
    - Implemented `pinnedHttpRequest()` using native Node `node:https` and `node:http`.
    - Integrated a custom `lookup` callback on the request options that resolves exclusively to `pinnedIp` (handling Node 22's `all: true` signature), preventing any secondary or unvalidated DNS resolution.
    - Preserved `parsed.hostname` for HTTP `Host` header and TLS SNI (`servername`), strictly enforcing TLS certificate validation (`rejectUnauthorized: true`).
    - Configured a 10-second default request timeout (`DEFAULT_SSRF_TIMEOUT_MS = 10000`), terminating slow or hanging connections with `category: 'TIMEOUT'` and HTTP status 504.
    - Configured a 5 MiB maximum response body size (`MAX_SSRF_RESPONSE_BYTES = 5 * 1024 * 1024`), promptly aborting oversized responses with `category: 'PAYLOAD_TOO_LARGE'` and HTTP status 413.
    - Enforced full stream, socket, timer, and `AbortSignal` cleanup on success, abort, timeout, and errors.
    - Updated `safeSsrfFetch()` to validate every hop, pin each redirect independently, enforce bounded redirects (max 3 hops), and strip `Authorization` headers on cross-origin redirects.
    - Closed the IPv6 classification gap in `parseIpv6Words()` and `isRestrictedIpv6()` for deprecated IPv4-compatible IPv6 addresses (`::127.0.0.1`, `::169.254.169.254`, `::10.0.0.1`) and IPv4-translated IPv6 addresses (`::ffff:0:a.b.c.d`).
- **Tests Added**:
  - Added 14 deterministic automated tests in `tests/api-provider-manager.test.ts` covering IPv4-compatible IPv6 blocking, deterministic IP pinning with Host preservation, anti-DNS rebinding/TOCTOU socket guarantee, multi-answer DNS rejection, redirect revalidation and independent pinning, cross-origin Authorization stripping, redirect hop limits, timeout enforcement (HTTP 504), payload size capping (HTTP 413), caller `AbortSignal` cancellation, provider manager `testConnection()` and `executeTextGeneration()` blocking without secret leaks, and preservation of built-in providers (`Google GenAI`, Groq default, OpenRouter default).
- **Verification Commands & Real Results**:
  - `npm test`: 133 passing tests across 10 test suites (duration ~3s, 0 failures, 0 skipped).
  - `npm run lint`: Clean (`tsc --noEmit`, 0 diagnostics).
  - `npm run build`: Production bundle built cleanly via Vite (`vite build`).
- **Status**: **Verified Completed**.

### Task 1.4A: Provider Execution & Cost Policy Audit (Completed & Verified)
- **Objective**: Conduct comprehensive assessment of existing provider execution engine, backend cost policies, Auto Free selection, BYOK behavior, and fallback handling without modifying application logic.
- **Key Findings**:
  1. **Server-Side Cost Policy & Auto Free Unimplemented**: `classifyOpenRouterPricing` in `modelCompatibility.ts` correctly verifies zero-priced vs paid tokens ($0.00/1M check), but `providerManager.executeTextGeneration` and `server.ts` endpoints do not accept or enforce a `costPolicy` parameter (`free_only`, `prefer_free`, `cost_controlled`, `manual`). Execution executes whatever `modelId`/`connectionId` is targeted.
  2. **Failover Ignores Cost Tiers**: In `providerManager.ts` (lines 2398-2420, 2577-2598), `allowFailover` checks only `quotaPool` mismatch and enabled status, failing to verify `c.pricing?.pricingTier`. A free route could failover to a paid route if both have `allowFailover: true`.
  3. **Connection Testing Triggers Live Generative Tokens**: `testConnection()` in `providerManager.ts` sends live generation requests (`Say OK`, max 5 tokens). For paid models on paid connections, this dispatches real billable inferences without cost warnings or user confirmation.
  4. **Silent Provider Switch on Unrecognized Connection ID**: In `executeTextGeneration`, if a non-existent `connectionId` is passed, the server defaults to `connections.find(c => c.isEnabled) || connections[0]`, silently switching to an unintended provider instead of returning HTTP 404 / `CONNECTION_NOT_FOUND`.
  5. **Credential Security & Zero Secrets Verified**: Key masking, dynamic env loading (`process.env.GEMINI_API_KEY`), error sanitization, `.provider_connections.json` zero-secret persistence, and export scrubber are verified secure with 100% test pass rate.
  6. **SSRF Transport Hardening Verified**: Socket-level IP pinning and safe transport verified secure.
- **Status**: **Audit Completed & Verified (Assessment Only)**.

---

## 3. In Progress
- **Item**: Task 1.4A Completed & Verified. Awaiting user review and approval before proceeding.
- **Upcoming Work**:
  - Task 1.4B / Phase 4 — Remediate provider execution policies: implement server-side `costPolicy` enforcement, prevent cross-tier paid failover, gate `testConnection()` on paid models, and return strict 404 on invalid `connectionId`.
  - Task 1.1 — Long-Form Duration Decision Resolution (awaiting product owner decision on DEC-01).

---

## 4. Known Issues & Security Risks

| ID | Issue / Risk | Severity | Evidence / Source | Status |
|---|---|---|---|---|
| **SEC-01** | Historical plaintext credentials in previous git commits | **HIGH** | Keys existed in repository before Task 0.3. | **Project owner must revoke/rotate these credentials in Google AI Studio / GCP Console.** |
| **SEC-02** | Missing `.gitignore` at repository root | **RESOLVED** | `.gitignore` created in Task 0.3. | **Verified Resolved** |
| **SEC-03** | Custom provider SSRF DNS Rebinding & TOCTOU IP Pinning | **RESOLVED** | Task 1.3B: Socket-level IP pinning (`pinnedHttpRequest`), TLS verification preservation, redirect re-validation, 10s timeout, 5 MiB payload cap, and IPv4-compatible IPv6 filtering implemented and verified. | **Verified Resolved (133/133 tests pass)** |
| **SEC-04** | Vite `allowedHosts: true` | **LOW** | `vite.config.ts` allows any host header. | Documented / Needed for container preview |
| **COST-01** | Server-side cost policy engine unintegrated in `executeTextGeneration` | **HIGH** | `executeTextGeneration` does not evaluate `pricingTier` or require user approval for paid models. | **Audited in Task 1.4A; Remediation Planned for Phase 4** |
| **COST-02** | Multi-connection failover does not enforce cost tier parity | **HIGH** | `providerManager.ts` lines 2398-2420 & 2577-2598 check `quotaPool` but omit `pricingTier`. | **Audited in Task 1.4A; Remediation Planned** |
| **COST-03** | Provider `testConnection` dispatches live billable generation | **MEDIUM** | `providerManager.ts` lines 1782-1785 & 2045-2049 send `Say OK` completion to targeted model. | **Audited in Task 1.4A; Remediation Planned** |
| **ROUTE-01**| Unrecognized `connectionId` silently defaults to first enabled connection | **MEDIUM** | `providerManager.ts` lines 2266-2268 silently falls back instead of throwing error. | **Audited in Task 1.4A; Remediation Planned** |

---

## 5. Architectural & Product Decisions

| Decision ID | Description | Status | Rationale |
|---|---|---|---|
| **DEC-01** | Long-Form duration: 3-min vs 4-min minimum | **UNRESOLVED / TBD** | Code allows `[3, 4, 5, 8, 10]` min; brief notes a 4-min spec conflict. Existing code is preserved without guessing until product decision. Verified in Task 1.2 regression suite. |
| **DEC-02** | Deployment Model (Single-user vs Multi-user) | **UNRESOLVED / TBD** | Affects session auth and multi-tenancy requirements. Preserved as local single-user baseline for now. |
| **DEC-03** | SEO Package Gate (Optional vs Mandatory) | **CONFIRMED IN CODE** | SEO is treated as optional in Stage 05, but flagged in Stage 07 completeness checklist. Export is permitted with warning. |

---

## 6. Next Exact Task

- **Task ID**: `TASK-1.4A-HANDOFF`
- **Goal**: Present Task 1.4A Provider Execution & Cost Policy Audit report to user, wait for review and directive before writing code.
- **Scope**: Assessment only; no application logic modified.


