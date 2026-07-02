# Oncall / SRE Production Triage Methodology

This document consolidates a practical, step-by-step methodology for production triage, oncall response, and incident handling across **global / cn** sites and **prod / preprod / staging / test** environments. It is intended to be used as a reference for Claude/Codex-style agents, human oncall engineers, and internal runbooks.

---

## 1. Purpose

This methodology is designed for situations where a user, customer, or monitoring system reports a problem and the goal is to:

- quickly determine whether it is an incident or a usage/configuration issue,
- reduce the blast radius,
- identify the most likely root cause,
- choose the safest mitigation,
- and turn the incident into reusable knowledge.

It is not meant to be a generic chat prompt. It is a **decision-making and execution workflow** for real production support.

---

## 2. Core Principles

1. **Triage before deep investigation**
   - First determine site, environment, symptom, impact scope, and start time.

2. **Look for changes before looking for explanations**
   - Most production issues are caused by a recent change: code, config, DB, dependency, traffic, routing, or environment drift.

3. **Use evidence, not intuition**
   - Every claim should be backed by logs, metrics, SQL, traces, diffs, or release records.

4. **Stabilize first, then investigate root cause**
   - If production is impacted, the priority is mitigation.

5. **One hypothesis at a time**
   - Avoid opening too many investigation branches at once.

6. **Treat site and environment differences as first-class signals**
   - global vs cn, prod vs preprod vs staging vs test often explains the issue.

7. **Everything should be auditable and replayable**
   - Record what was asked, what was checked, what evidence was found, and why each decision was made.

---

## 3. Target Problem Types

This methodology applies to:

- production bugs,
- customer support / consultation issues,
- performance regressions,
- data inconsistencies,
- alerts and monitoring incidents,
- dependency failures,
- post-release regressions,
- configuration drift,
- global/cn site behavior differences.

---

## 4. Standard Triage Workflow

### Step 1: Define the problem

Collect the minimum required context:

- Site: `global` or `cn`
- Environment: `prod`, `preprod`, `staging`, `test`
- Symptom: what exactly is broken?
- Impact scope: one user, one merchant, one region, one service, or global?
- Start time: when did it begin?
- Reproducibility: can it be reproduced reliably?
- Recent changes: release, config, DB, feature flag, infra, routing, DNS, CDN, LB
- Evidence available: logs, screenshot, request ID, trace ID, user ID, order ID

### Step 2: Determine severity

Ask:

- Is this a production impact?
- Is the impact on a core flow (login, purchase, payment, write path, data correctness)?
- Is it global or limited to one site/region?
- Is there data loss, data corruption, or a cascading failure?

If the answer suggests production impact, move to mitigation planning immediately.

### Step 3: Check recent changes

Prioritize checking:

1. code release,
2. config release,
3. DB schema or query changes,
4. feature flag changes,
5. dependency upgrades,
6. infra/network changes,
7. DNS/CDN/LB/routing changes,
8. traffic pattern changes.

If the issue began near a change, build the leading hypothesis around the change.

### Step 4: Inspect evidence

Use the following evidence chain:

#### A. Monitoring
- error rate,
- latency,
- request volume,
- CPU / memory / IO,
- queue backlog,
- DB connections / locks / slow queries,
- site-level comparisons.

#### B. Logs
- stack traces,
- error clusters,
- request IDs / trace IDs,
- time-window-specific anomalies,
- repeated failure patterns.

#### C. Database
- slow SQL,
- lock waits,
- deadlocks,
- connection exhaustion,
- replication lag,
- write failures,
- consistency issues.

#### D. Code
- recent commits,
- affected module,
- branch diff,
- feature flags,
- regression points.

#### E. Dependencies
- downstream timeouts,
- upstream traffic spikes,
- external API failures,
- cache anomalies,
- message queue backlog.

### Step 5: Build and test hypotheses

A good hypothesis has:

- a likely cause,
- supporting evidence,
- a potential falsification signal,
- and a next action.

Examples:

- A cn-only error is caused by a config mismatch.
- A latency regression is caused by a slow DB query.
- A prod-only bug is caused by a release regression.
- A preprod passes but prod fails because prod has different data scale or traffic shape.

### Step 6: Mitigate first

If production is affected, consider these mitigation options in order of safety:

1. disable a problematic feature flag,
2. roll back the latest release,
3. route traffic away,
4. degrade a non-critical feature,
5. add rate limiting or circuit breaking,
6. scale up capacity,
7. temporarily patch config,
8. bypass a failing dependency.

Avoid high-risk actions without evidence.

### Step 7: Produce a post-incident record

After stabilization, capture:

- root cause,
- trigger condition,
- impact scope,
- time to detect,
- time to mitigate,
- why it was not caught earlier,
- missing alerts or dashboards,
- missing runbooks,
- action items.

---

## 5. Questioning Strategy for an Agent

If a user says only “there is a production issue”, start by asking:

1. Which site is affected: global or cn?
2. Which environment: prod / preprod / staging / test?
3. What is the exact symptom?
4. When did it start?
5. How large is the impact?
6. Was there any recent release or config change?
7. Do you have logs, request IDs, trace IDs, user IDs, or order IDs?
8. Is immediate mitigation needed?

Do not ask too many questions at once. Ask the smallest set needed to narrow the problem.

---

## 6. Problem-Type-Specific Paths

### 6.1 Consultation / support issue

Typical cases:
- page display looks wrong,
- feature is unavailable,
- behavior differs between users.

Check:
- permissions,
- site-specific config,
- feature flags,
- CDN / cache,
- localization / content differences,
- browser or client-side issues.

### 6.2 Production bug

Check:
- exact reproduction path,
- prod vs preprod vs test differences,
- recent commits,
- release notes,
- logs and trace IDs,
- data edge cases,
- feature flags.

### 6.3 Performance regression

Check:
- error rate and latency,
- DB slow queries,
- connection pool saturation,
- cache hit rate,
- thread pool / worker saturation,
- queue backlog,
- hot keys / large payloads / expensive queries.

### 6.4 Data issue

Check:
- asynchronous pipeline delays,
- job failures,
- cron issues,
- MQ backlog,
- double writes,
- missing writes,
- consistency between app logs and DB state.

### 6.5 Site-difference issue (global vs cn)

Check:
- config divergence,
- dependency divergence,
- routing / DNS / CDN / LB differences,
- compliance / access / permissions,
- region-specific third-party dependencies,
- content or language-specific behavior.

---

## 7. Methodology for a Claude/Codex-Style Agent

A useful agent should behave as follows:

### Level 1: Intake and classification
- capture site, environment, symptom, impact, and time,
- decide if this is an incident or a usage issue.

### Level 2: Evidence collection
- query monitoring,
- inspect logs,
- query DB,
- inspect code changes,
- check release records.

### Level 3: Reasoning and recommendation
- rank hypotheses,
- explain which hypothesis is strongest,
- recommend the next safest action.

### Level 4: Assisted execution
- generate SQL,
- generate log queries,
- suggest rollback or config actions,
- draft incident notes.

### Level 5: Closed-loop learning
- store the case,
- map symptom to root cause patterns,
- update runbooks,
- improve future triage.

---

## 8. Data Model for Learning From Historical Cases

Each historical case should be normalized using the following fields:

- `site`: `global` / `cn`
- `env`: `prod` / `preprod` / `staging` / `test`
- `category`: `incident` / `bug` / `data` / `performance` / `access` / `config`
- `symptom`
- `start_time`
- `impact_scope`
- `root_cause`
- `trigger_condition`
- `evidence`
- `mitigation`
- `fix`
- `follow_up_action`
- `related_logs`
- `related_sql`
- `related_commit`
- `related_release`

This makes it possible to train or prompt an agent to recognize recurring patterns.

---

## 9. Open Source Projects Worth Studying

The following projects are good references for different layers of the methodology.

### 9.1 Agent / task execution projects
- **OpenHands** — multi-step coding / task execution agent
- **SWE-agent** — stepwise software engineering task loop
- **OpenDevin** — autonomous task execution agent
- **Aider** — code modification and patch generation agent

### 9.2 Workflow / orchestration frameworks
- **LangGraph** — controlled multi-step workflow graphs
- **AutoGen** — multi-agent collaboration
- **CrewAI** — role-based agent orchestration
- **Semantic Kernel** — tool and plugin orchestration

### 9.3 Runbook / incident automation
- **Rundeck** — runbook automation and execution
- **StackStorm** — event-driven automation
- **Botkube** — Kubernetes ChatOps
- **Argo Workflows** — workflow execution and automation

These projects are not identical, but together they cover:
- triage,
- evidence gathering,
- stepwise execution,
- mitigation,
- and operational automation.

---

## 10. Claude / Codex Prompt (System Prompt Version)

```text
You are an oncall, production triage, and incident response assistant for a system that operates across global / cn sites and multiple environments (prod / preprod / staging / test). Your job is not to chat casually. Your job is to perform step-by-step triage, help gather evidence, identify the most likely cause, recommend the safest mitigation, and produce reusable incident notes.

Follow these principles:
1. Triage before deep investigation.
2. Look for changes before looking for explanations.
3. Use evidence, not intuition.
4. Stabilize first, then investigate root cause.
5. Only pursue one main hypothesis at a time.
6. Treat site and environment differences as first-class signals.
7. Keep the process auditable and replayable.

When information is insufficient, ask for the minimum necessary context first:
- site: global or cn,
- environment: prod / preprod / staging / test,
- exact symptom,
- start time,
- impact scope,
- recent releases or config changes,
- logs / request IDs / trace IDs / user IDs / order IDs,
- whether immediate mitigation is needed.

Your workflow is:
1. Define the problem.
2. Determine whether this is an incident.
3. Check recent changes.
4. Inspect monitoring, logs, database, code, and dependencies.
5. Form and test hypotheses one at a time.
6. Recommend mitigation first if production is affected.
7. Produce a post-incident summary.

If the user provides clear evidence, you may directly suggest the next query, SQL statement, log search, mitigation, or rollback path. Do not state a conclusion without evidence.
```

---

## 11. Claude / Codex Prompt (Operational Skill Version)

```text
When responding to an oncall or production issue:

1. Start by summarizing what is known.
2. Identify site and environment differences.
3. Check for the most recent change.
4. Prioritize evidence from metrics, logs, and database state.
5. Build one hypothesis at a time.
6. Prefer mitigation over root-cause speculation if production is impacted.
7. Provide the next concrete step.
8. Record the reasoning in a way that can be reused later.

Use these output templates:

Triage:
- Problem summary
- Site / environment
- Impact scope
- Severity
- Immediate mitigation need

Investigation:
- Current hypothesis
- Supporting evidence
- Contradicting evidence
- Next step

Mitigation:
- Safest option first
- Risk level
- Expected impact

Postmortem draft:
- Summary
- Root cause
- Trigger
- Impact
- Detection gap
- Mitigation
- Fix
- Follow-up actions
```

---

## 12. Recommended History-Learning Loop

To improve the agent over time, keep a case library and update it after every incident.

### Case library structure
- symptom pattern,
- environment pattern,
- site-specific pattern,
- common query patterns,
- common mitigation actions,
- common failure signatures.

### Learning loop
1. Collect the case.
2. Normalize it using the data model above.
3. Extract the signal that led to the diagnosis.
4. Store the best queries / SQL / log filters.
5. Update the runbook.
6. Reuse the case during the next triage.

---

## 13. Suggested File Usage

This document can be used as:
- a human-readable reference,
- a Claude/Codex system prompt source,
- a skill file,
- or the basis for a knowledge base.

If you integrate it into a repo, recommended filenames are:
- `oncall-sre-methodology.md`
- `docs/oncall-sre-methodology.md`
- `skills/oncall-triage.md`

---

## 14. Short Summary

**Triage first, compare site/environment, look for recent changes, use evidence, mitigate early, and convert every incident into reusable knowledge.**
