# bmm

Pipeline skills for BMAD BMM workflows. Parallelizes story creation, development, code review, and research with Agent Teams, and provides context-isolated review for unbiased adversarial analysis.

## Skills

### [omb-bmm-create-story-parallel](skills/omb-bmm-create-story-parallel/SKILL.md)

Parallelized execution of `bmad-bmm-create-story`. Overrides research steps (2–4) with 3 concurrent analysis agents.

Three research agents run simultaneously:

| Analysis | Agent | Model | Focus |
|----------|-------|-------|-------|
| Artifact Analysis | artifact-analyst | Opus | PRD, epics, and existing artifacts |
| Architecture Analysis | architecture-analyst | Sonnet | Technical architecture review |
| Tech Research | tech-researcher | Sonnet | Technology investigation |

Agents stay alive after initial research for follow-up questions during story authoring (step 5+).

### [omb-bmm-dev-story-parallel](skills/omb-bmm-dev-story-parallel/SKILL.md)

Parallelized execution of `bmad-bmm-dev-story`. Overrides implementation steps (5–8) with a concurrent TDD pipeline.

Two pipeline modes based on context:

- **Implementation Pipeline** — Full 4-stage TDD per task, with dependency graph-based parallelism:

| Stage | Agent | Model | Focus |
|-------|-------|-------|-------|
| Red | tester-{N} | Sonnet | Write failing tests with stub signatures |
| Green | implementer-{N} | Sonnet | Minimal implementation to pass all tests |
| Refactor | refactorer-{N} | Sonnet | Improve code quality, keep tests green |
| Verify | validator-{N} | Sonnet | Full validation gates and acceptance criteria |

- **Review Pipeline** — Lightweight 2-stage fix per review item:

| Stage | Agent | Model | Focus |
|-------|-------|-------|-------|
| Fix | refactorer-{R} | Sonnet | Address review finding, keep tests green |
| Verify | validator-{R} | Sonnet | Validate fix and check for regressions |

Independent tasks/items run concurrently; dependent ones wait for upstream verification to pass.

### [omb-bmm-code-review-parallel](skills/omb-bmm-code-review-parallel/SKILL.md)

Parallelized execution of `bmad-bmm-code-review`. Overrides the review step (3) with 4 concurrent review dimensions.

Four independent reviewers run simultaneously:

| Dimension | Agent | Model | Focus |
|-----------|-------|-------|-------|
| Git Audit | git-auditor | Sonnet | Cross-reference git changes vs story file list |
| AC Validation | ac-validator | Opus | Verify all Acceptance Criteria are met |
| Task Audit | task-auditor | Opus | Verify task completion claims |
| Code Quality | code-reviewer | Opus | Deep code quality review |

Findings are aggregated, deduplicated, and categorized by severity after all reviewers complete.

### [omb-bmm-technical-research-parallel](skills/omb-bmm-technical-research-parallel/SKILL.md)

Parallelized execution of `bmad-bmm-technical-research`. Overrides research steps (2–5) with 4 concurrent specialist agents.

Four research agents run simultaneously:

| Area | Agent | Model | Focus |
|------|-------|-------|-------|
| Technology Stack | tech-stack-analyst | Sonnet | Languages, frameworks, tools, platforms |
| Integration Patterns | integration-analyst | Sonnet | APIs, protocols, interoperability |
| Architectural Patterns | architecture-analyst | Sonnet | System design, scalability, security |
| Implementation | implementation-researcher | Sonnet | Adoption strategies, workflows, operations |

Agents report key findings summaries; team lead coordinates user [C] Continue approval. Upon approval, agents write directly to the document before shutdown.

### [omb-bmm-market-research-parallel](skills/omb-bmm-market-research-parallel/SKILL.md)

Parallelized execution of `bmad-bmm-market-research`. Overrides research steps (2–5) with 4 concurrent specialist agents.

Four research agents run simultaneously:

| Area | Agent | Model | Focus |
|------|-------|-------|-------|
| Customer Behavior | customer-behavior-analyst | Sonnet | Behavior patterns, segments, demographics |
| Pain Points | pain-points-analyst | Sonnet | Challenges, frustrations, unmet needs |
| Decision Journey | decision-journey-analyst | Sonnet | Decision criteria, touchpoints, journey mapping |
| Competitive Analysis | competitive-analyst | Sonnet | Market positioning, competitive landscape |

Agents report key findings summaries; team lead coordinates user [C] Continue approval. Upon approval, agents write directly to the document before shutdown.

### [omb-bmm-domain-research-parallel](skills/omb-bmm-domain-research-parallel/SKILL.md)

Parallelized execution of `bmad-bmm-domain-research`. Overrides research steps (2–5) with 4 concurrent specialist agents.

Four research agents run simultaneously:

| Area | Agent | Model | Focus |
|------|-------|-------|-------|
| Industry Analysis | industry-analyst | Sonnet | Market size, growth, industry dynamics |
| Competitive Landscape | competitive-analyst | Sonnet | Key players, market share, competitive dynamics |
| Regulatory Focus | regulatory-analyst | Sonnet | Regulations, compliance, standards |
| Technical Trends | technical-trends-analyst | Sonnet | Emerging technologies, innovation patterns |

Agents report key findings summaries; team lead coordinates user [C] Continue approval. Upon approval, agents write directly to the document before shutdown.

### [omb-bmm-quick-dev-isolated-review](skills/omb-bmm-quick-dev-isolated-review/SKILL.md)

Context-isolated adversarial review for `bmad-bmm-quick-dev`. Overrides step 5 by spawning a separate agent with no implementation context.

The main agent runs steps 1–4 normally, then constructs a diff and hands it to an isolated reviewer:

| Role | Agent | Model | Focus |
|------|-------|-------|-------|
| Adversarial Reviewer | (foreground sub-agent) | Opus | Diff-only review with zero implementation context |

The reviewer receives only the diff and `review-adversarial-general.xml` — no plans, decisions, or self-check results. This eliminates confirmation bias from the implementing agent reviewing its own work. Findings are processed back by the main agent before resuming step 6.
