# 🌀 AI Orchestration Layer — Detailed Sprint Plan (Single Windsurf Cascade Window)

> **Mode:** Single Cascade Session  
> **Agent Role:** Full-stack Dev + System Integrator  
> **Primary IDE:** Windsurf (headless or GUI)  
> **Integration Target:** GitHub + Docker + CI/CD

---

## 🧭 Sprint 0 — Environment Preparation (Week 0)

**Goal:** Prepare Windsurf, GitHub, and local containers to support headless orchestration.

**Agent Tasks:**
- [ ] Install and configure `windsurfinabox` Docker image.  
- [ ] Verify access to GitHub API (personal access token or App).  
- [ ] Set up base repo: `ai-orchestrator`.  
- [ ] Create `.env` with required variables:

- [ ] Confirm `pytest` and `docker` are callable from within Windsurf.

**Deliverables:**
- Working Windsurf Cascade session.
- GitHub connection test succeeded.
- Docker container can run Windsurf headlessly.

**Acceptance Criteria:**
- Agent can read/write files, commit code, and push to a branch.
- Logs visible in `windsurf-output.txt`.

---

## 🧱 Sprint 1 — Orchestrator Core MVP (Weeks 1–2)

**Goal:** Build minimal orchestration core to manage tasks and agent jobs.

**Agent Instructions:**
1. Create a new Python project `orchestrator_core/`.
2. Implement modules:
 - `task_queue.py` — basic in-memory queue.
 - `job_manager.py` — create, track, and close jobs.
 - `config.py` — loads YAML or ENV configuration.
3. Expose REST endpoints with FastAPI:
 - `POST /job/start`
 - `GET /job/status/{id}`
 - `POST /job/stop`
4. Add basic logging and SQLite persistence (`sqlalchemy`).

**Tests:**
- Unit tests for job lifecycle.
- Mock queue concurrency.

**Acceptance Criteria:**
- One job can be started, tracked, and closed successfully.
- Logs stored under `logs/` directory.

---

## 🌐 Sprint 2 — GitHub Webhook & API Layer (Weeks 3–4)

**Goal:** Integrate GitHub issue events and PR automation.

**Agent Instructions:**
1. Implement `github_listener.py`:
 - Handles `issues` and `pull_request` events.
 - Filters for label `AI-Ready`.
2. Add GitHub REST API client (`requests` or `PyGithub`):
 - Create branch from `main`.
 - Push commits via agent tokens.
3. Store issue data in DB (`Issue` table).

**Testing:**
- Simulate incoming webhook JSON locally.
- Unit tests for branch naming and PR payload.

**Acceptance Criteria:**
- Creating an issue with `AI-Ready` label triggers orchestrator.
- New branch is created automatically.
- Logs contain issue → branch mapping.

---

## ⚙️ Sprint 3 — Agent Execution Pipeline (Weeks 5–6)

**Goal:** Launch Windsurf Cascade tasks automatically from orchestrator.

**Agent Instructions:**
1. Build `agent_runner.py`:
 - Receives job context and repo path.
 - Starts Windsurf in headless mode (`docker run …`).
 - Passes prompt file (`windsurf-instructions.txt`).
2. Implement job status polling loop.
3. Capture `windsurf-output.txt` to DB.

**Testing:**
- Dry run with mock prompt (e.g., “add README.md”).
- Verify log capture and job closure.

**Acceptance Criteria:**
- AI agent executes autonomously within container.
- Output file saved and associated with `AgentJob`.

---

## 🤖 Sprint 4 — Multi-Agent Coordination & Prompts (Weeks 7–8)

**Goal:** Enable multiple agent roles and structured prompt generation.

**Agent Instructions:**
1. Create prompt templates under `prompts/`:
 - `bugfix_template.md`
 - `feature_template.md`
 - `refactor_template.md`
2. Implement `PromptBuilder`:
 - Reads issue type, priority, and files.
 - Generates unified task prompt.
3. Extend orchestrator to handle agent chains:
 - `coder → reviewer → tester`.

**Testing:**
- Simulate sequence on one issue.
- Ensure reviewer receives diff output.

**Acceptance Criteria:**
- Orchestrator can spawn multi-role agent jobs.
- Prompt templates selected correctly.
- All logs linked via job IDs.

---

## 🧩 Sprint 5 — CI/CD Integration & Retry Logic (Weeks 9–10)

**Goal:** Integrate with GitHub Actions and handle failed builds gracefully.

**Agent Instructions:**
1. Implement `ci_listener.py`:
 - Listens to `check_suite` webhook.
 - Updates PR status.
2. Create retry logic:
 - If `CIResult = failed`, relaunch agent with “fix failed tests” prompt.
 - Limit to 3 attempts.
3. Store `CIResult` entity in DB.

**Testing:**
- Simulate failed and passed builds.
- Validate retry counter.

**Acceptance Criteria:**
- CI feedback loop operational.
- Failed PR triggers automated retry once.

---

## 🔒 Sprint 6 — Compliance, Security & Governance (Weeks 11–12)

**Goal:** Add audit logging and SOC 2-ready evidence tracking.

**Agent Instructions:**
1. Extend database with `ComplianceRecord`.
2. Log every orchestrator event (hash, timestamp, agent ID).
3. Store all prompts and model metadata in `/artifacts/`.
4. Implement “read-only verification” API endpoint:
 - `/compliance/{job_id}` returns evidence bundle.

**Testing:**
- Simulate evidence verification.
- SHA256 hash check on artifacts.

**Acceptance Criteria:**
- Every PR has linked compliance record.
- Evidence is verifiable and immutable.

---

## 📊 Sprint 7 — Observability & Scaling (Weeks 13–14)

**Goal:** Add performance metrics and support multiple concurrent agents.

**Agent Instructions:**
1. Integrate Prometheus client for metrics:
 - `active_jobs_total`
 - `avg_job_duration_seconds`
 - `ci_pass_rate`
2. Add Kubernetes `deployment.yaml` + autoscaler.
3. Log resource usage per container.

**Testing:**
- Spin up 3 concurrent agent jobs.
- Ensure queueing and scaling behave correctly.

**Acceptance Criteria:**
- Metrics available via `/metrics` endpoint.
- Autoscaler responds to queue length.

---

## 🧠 Sprint 8 — Production Integration & Team Enablement (Weeks 15–16)

**Goal:** Deploy orchestrator into a production engineering environment with humans in the loop.

**Agent Instructions:**
1. Implement environment variable `PRODUCTION_MODE=true`.
2. Limit to repos approved in config.
3. Add human review requirement before merge.
4. Create documentation:
 - `README.md`
 - `AI-Ready Issue Guide`
 - `Reviewer Checklist`

**Testing:**
- Pilot run on real issue in staging repo.
- Measure PR generation and review time.

**Acceptance Criteria:**
- 1+ real PRs generated, reviewed, and merged via hybrid workflow.
- Logs and metrics validated.
- Documentation complete.

---

## 🧾 Overall Success Metrics

| **Metric** | **Target** | **Measured By** |
|-------------|-------------|-----------------|
| Time-to-PR | < 60 min average | Job logs |
| Test Pass Rate | ≥ 95% on first run | CIResults |
| Automation Rate | ≥ 85% of AI-Ready issues | Issue table |
| Review Time | < 15 min avg | GitHub review timestamps |
| Compliance Traceability | 100% | ComplianceRecord count |

---

## 🧰 Developer / Agent Tools Setup (Preloaded in Windsurf)

| Tool | Purpose |
|------|----------|
| **FastAPI** | REST orchestration service |
| **SQLAlchemy + SQLite** | Lightweight database |
| **PyGithub** | GitHub API wrapper |
| **Docker SDK for Python** | Manage agent containers |
| **Prometheus Client** | Metrics |
| **pytest + coverage** | Testing framework |
| **cryptography** | Evidence hashing |
| **requests** | Webhook communication |

---

## 🧭 Sprint Execution Loop (Inside Windsurf)

During each sprint, run the following cycle in the Cascade window:

1. **Observe:** Read issue from `AI-Ready` backlog.  
2. **Plan:** Build/update `PromptBuilder` or orchestrator logic.  
3. **Act:** Execute Cascade tasks autonomously (use `!run` or CLI commands).  
4. **Test:** Run local unit/CI tests.  
5. **Commit:** Save to `ai-{sprint-number}` branch.  
6. **Push:** Trigger GitHub Actions pipeline.  
7. **Log:** Save execution trace to `windsurf-output.txt`.

This ensures a continuous build–test–review loop entirely managed inside one Cascade session.

---

## 🏁 Completion Definition

At the end of Sprint 8:

- ✅ Orchestrator service runs fully autonomous for `AI-Ready` issues.  
- ✅ Windsurf Cascade executes coding tasks headlessly.  
- ✅ PRs generated automatically and reviewed via human oversight.  
- ✅ Compliance and observability dashboards operational.  
- ✅ Documentation delivered to engineering and audit teams.

---

**End of Sprint Plan**
