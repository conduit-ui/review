# Enterprise-Grade Review System with Pattern Indexing

> **For Claude:** REQUIRED SUB-SKILL: Use superpowers:executing-plans to implement this plan task-by-task.

**Goal:** Build a sophisticated PR review system that learns codebase patterns and risks, stores reviews with intelligent purging, and provides enterprise-grade feedback for SaaS teams managing 1000+ applications.

**Architecture:** The system consists of four main components: (1) an enhanced coordinator that reads CLAUDE.md files to understand codebase patterns, (2) a pattern indexing system that learns from each review and persists across sessions, (3) review storage with configurable retention and smart purging, and (4) analysis capabilities that identify security risks, N+1 queries, and legacy code interactions.

**Tech Stack:** Markdown-based agents, JSON pattern indexes, date-based file storage, environment configuration via .env

---

## Task 1: Create Review Storage Architecture

**Files:**
- Create: `.claude/reviews/.env.example`
- Create: `.claude/reviews/config.php` (or equivalent config system)
- Modify: `agents/pr-review-coordinator.agent.md` (add storage initialization)

**Step 1: Write the configuration example file**

Create `.claude/reviews/.env.example`:
```
REVIEW_RETENTION_DAYS=30
REVIEW_STORAGE_PATH=./.claude/reviews
PATTERN_INDEX_PATH=./.claude/reviews/patterns
AUTO_PURGE_ENABLED=true
```

**Step 2: Create storage initialization in coordinator**

Modify `agents/pr-review-coordinator.agent.md` to add a new STEP 0 that:
- Reads or creates `.env` from `.env.example` if it doesn't exist
- Creates `.claude/reviews/patterns/index.json` if it doesn't exist
- Initializes review session directory with timestamp
- Logs retention settings to user

**Step 3: Verify structure**

Check that the following directories can be created:
```
.claude/reviews/
├── .env
├── patterns/
│   └── index.json
├── YYYY-MM-DD_HH-MM-SS/
│   ├── metadata.json
│   ├── architecture-findings.md
│   ├── implementation-findings.md
│   └── synthesis-report.md
```

**Step 4: Commit**

```bash
cd /tmp/conduit-ui-review
git add docs/plans/2025-12-10-enterprise-review-system.md
git add agents/pr-review-coordinator.agent.md
git commit -m "feat: add review storage architecture and configuration system"
```

---

## Task 2: Create Pattern Index System

**Files:**
- Create: `lib/pattern-indexer.md` (agent-compatible documentation)
- Create: `agents/pattern-analyzer.agent.md`

**Step 1: Design the pattern index structure**

Create `lib/pattern-indexer.md` that documents the index format:

```json
{
  "version": "1.0",
  "codebase": {
    "name": "pstrax-monolith",
    "path": "/Users/jordan/Sites/pstrax-laravel/apps/pstrax-monolith"
  },
  "patterns": {
    "security_risks": [
      {
        "id": "timezone-handling-risk",
        "severity": "high",
        "description": "Departments use TimeZone field for UTC offset; legacy code assumes Central time",
        "locations": ["app/Blood/Console/Commands/CheckBloodProductExpirationsCommand.php:95"],
        "impact": "Incorrect timezone conversion could trigger alerts at wrong times",
        "last_seen": "2025-12-10T19:30:00Z"
      }
    ],
    "n_plus_one_risks": [
      {
        "id": "user-watchers-query",
        "location": "app/Alerts/Listeners/SendEntityAlertNotifications.php:50",
        "description": "Loads users by ID without relationship eager loading",
        "status": "mitigated",
        "mitigation": "Uses whereIn() to batch load"
      }
    ],
    "legacy_interactions": [
      {
        "id": "equipment-alerts-renamed",
        "description": "Legacy EquipmentAlerts table renamed to VehicleAlert; old references may exist",
        "status": "monitored",
        "last_audit": "2025-12-10"
      }
    ]
  },
  "statistics": {
    "total_reviews": 42,
    "last_review": "2025-12-10T19:30:00Z",
    "patterns_discovered": 8
  }
}
```

**Step 2: Create pattern-analyzer agent**

Create `agents/pattern-analyzer.agent.md`:

```markdown
---
name: pattern-analyzer
description: Analyzes code changes against codebase pattern index to identify security risks, N+1 queries, and legacy code interactions
tools: Read, Grep, Task
model: haiku
---

Your job is to:
1. Load the pattern index from .claude/reviews/patterns/index.json
2. Analyze the current changes against known patterns
3. Return findings in structured JSON format with severity levels
4. Update the pattern index with new discoveries

## Input
- Current git diff
- Pattern index (if exists)
- CLAUDE.md files from the project

## Output
```json
{
  "security_findings": [],
  "performance_findings": [],
  "legacy_interaction_findings": [],
  "new_patterns_discovered": [],
  "risk_score": 0-10
}
```
```

**Step 3: Integrate pattern analyzer into coordinator**

Modify `agents/pr-review-coordinator.agent.md` to add pattern analysis as part of STEP 1:
- Launch pattern-analyzer as background task
- Wait for it to complete
- Include pattern findings in architecture review context

**Step 4: Commit**

```bash
git add lib/pattern-indexer.md agents/pattern-analyzer.agent.md
git commit -m "feat: create pattern indexing system for codebase risk tracking"
```

---

## Task 3: Implement Smart Review Purging

**Files:**
- Create: `agents/review-maintenance.agent.md`
- Modify: `agents/pr-review-coordinator.agent.md` (add maintenance trigger)

**Step 1: Create maintenance agent**

Create `agents/review-maintenance.agent.md`:

```markdown
---
name: review-maintenance
description: Manages review storage, purges old reviews based on retention policy, and compacts pattern index
tools: Bash, Read, Write
model: haiku
---

Your responsibilities:
1. Read REVIEW_RETENTION_DAYS from .env
2. List all reviews in .claude/reviews/YYYY-MM-DD_HH-MM-SS/
3. Calculate age of each review
4. Delete reviews older than retention period
5. Log purged reviews to maintenance.log
6. Compact pattern index (remove duplicates, consolidate findings)
7. Return summary of actions taken

Output format:
```json
{
  "reviews_purged": 3,
  "reviews_deleted": [
    "2025-11-10_14-30-00",
    "2025-11-09_10-15-22"
  ],
  "patterns_consolidated": 2,
  "storage_freed_mb": 15.4,
  "next_purge_date": "2025-01-09"
}
```
```

**Step 2: Add maintenance trigger to coordinator**

At the end of STEP 3 (after synthesis), add:
```
If AUTO_PURGE_ENABLED is true, trigger review-maintenance agent
Report purging results to user
```

**Step 3: Test the purging logic**

Document expected behavior:
- Reviews older than REVIEW_RETENTION_DAYS are deleted
- Patterns from deleted reviews are preserved in index
- Maintenance happens automatically after each review

**Step 4: Commit**

```bash
git add agents/review-maintenance.agent.md
git commit -m "feat: implement intelligent review storage purging with configurable retention"
```

---

## Task 4: Enhance Architecture Reviewer with Pattern Context

**Files:**
- Modify: `agents/architecture-reviewer.agent.md`

**Step 1: Update architecture reviewer prompt**

Enhance the architecture-reviewer to receive pattern context. Update the coordinator's Task call to include:

```
prompt="You are the architecture reviewer. Before reviewing the PR, consider:

CODEBASE CONTEXT (from pattern index):
- Known security risks: [list from index]
- Known N+1 vulnerabilities: [list from index]
- Legacy interactions to watch: [list from index]

Now review this Laravel PR for:
- Architecture and design patterns (does it work with existing patterns?)
- How it interacts with known risks
- Whether it mitigates or exacerbates security concerns
- Performance implications given codebase history
- Production readiness

Output scores 1-10 and include pattern interaction assessment."
```

**Step 2: Update architecture reviewer to capture new patterns**

The agent should:
- Identify new security risks discovered during review
- Flag new N+1 query patterns
- Note new legacy code interactions
- Return these as `new_patterns_discovered` for index update

**Step 3: Modify synthesizer to merge findings**

In the synthesizer (coordinator STEP 3), merge:
- Known patterns from index
- New patterns from this review
- Current PR findings
- Into a unified risk assessment

**Step 4: Commit**

```bash
git add agents/architecture-reviewer.agent.md agents/pr-review-coordinator.agent.md
git commit -m "feat: enhance architecture reviewer with codebase pattern context"
```

---

## Task 5: Create Pattern Update Mechanism

**Files:**
- Create: `agents/pattern-indexer.agent.md`
- Modify: `agents/pr-review-coordinator.agent.md` (add pattern update step)

**Step 1: Create pattern indexer agent**

Create `agents/pattern-indexer.agent.md`:

```markdown
---
name: pattern-indexer
description: Updates the codebase pattern index with new discoveries from the current review
tools: Read, Write
model: haiku
---

Your job:
1. Read current pattern index from .claude/reviews/patterns/index.json
2. Receive new patterns discovered from architecture and implementation reviewers
3. Merge new patterns with existing ones (avoid duplicates)
4. Update last_seen timestamps
5. Increment review statistics
6. Write updated index back to disk

Input: current_index.json + new_patterns[]
Output: updated_index.json

Only add patterns where:
- Severity is high or medium
- Pattern appears in 2+ reviews OR
- Is a critical security risk
```

**Step 2: Add pattern update to coordinator workflow**

After STEP 3 synthesis, add new STEP 4:
- Call pattern-indexer agent
- Pass synthesis results + new patterns
- Wait for index update to complete
- Report what patterns were learned

**Step 3: Document pattern discovery workflow**

Show how patterns compound over time:
- Review 1: Discover timezone risk, add to index
- Review 2: See timezone risk again, increase confidence
- Review 3: Find related migration risk, link together
- Index becomes more useful as reviews accumulate

**Step 4: Commit**

```bash
git add agents/pattern-indexer.agent.md agents/pr-review-coordinator.agent.md
git commit -m "feat: implement automatic pattern index updates from review findings"
```

---

## Task 6: Add Configuration Management Slash Command

**Files:**
- Create: `commands/configure.md`
- Modify: `README.md` (document the command)

**Step 1: Create configure command**

Create `commands/configure.md`:

```markdown
# Configure Review System

This command lets you view and modify review system settings.

## Usage

`/configure` - Interactive configuration wizard
`/configure view` - Show current settings
`/configure set REVIEW_RETENTION_DAYS 45` - Update specific setting
`/configure reset` - Restore defaults

## What it does:
1. Shows current .env settings
2. Allows user to modify REVIEW_RETENTION_DAYS, AUTO_PURGE_ENABLED, etc.
3. Validates settings (retention must be 1-365 days)
4. Saves to .env
5. Shows next purge date based on new settings

## Example output:
```
Current Review System Configuration:
├── Retention Period: 30 days
├── Storage Path: ./.claude/reviews
├── Auto Purge: Enabled
├── Next Purge: 2025-01-09
└── Pattern Index: 8 patterns tracked

Modify any setting (or press Enter to skip):
Review Retention Days [30]: 45
Auto Purge [true]:
```
```

**Step 2: Update README with configuration section**

Add to README.md:

```markdown
## Configuration

Customize review retention and storage:

```bash
/configure                    # Interactive setup
/configure view               # Show current settings
/configure set REVIEW_RETENTION_DAYS 45
```

### Settings

- `REVIEW_RETENTION_DAYS` (1-365): How long to keep reviews
- `AUTO_PURGE_ENABLED` (true/false): Automatically delete old reviews
- `PATTERN_INDEX_PATH`: Where to store pattern discoveries

Default: 30-day retention with auto-purge enabled.
```

**Step 3: Commit**

```bash
git add commands/configure.md README.md
git commit -m "feat: add /configure command for review system settings"
```

---

## Task 7: Create Comprehensive Agent Test Scenarios

**Files:**
- Create: `tests/scenarios.md`

**Step 1: Document test scenarios**

Create `tests/scenarios.md` with realistic SaaS scenarios:

```markdown
# Enterprise Review System - Test Scenarios

## Scenario 1: Security Risk Discovery
App: Multi-tenant SaaS with payment processing
Change: Update password hashing function
Expected: Pattern analyzer detects crypto library change, flags all password flows for audit

## Scenario 2: N+1 Query Detection
App: Analytics dashboard with 1000+ apps
Change: Add new metrics calculation loop
Expected: Pattern analyzer spots potential N+1 in loop, references previous N+1 patterns discovered

## Scenario 3: Legacy Code Interaction
App: 10-year-old Rails monolith being modernized
Change: Refactor user authentication
Expected: Agent warns about 3 legacy auth systems still in use, provides safe migration path

## Scenario 4: Review Storage and Purging
App: Active development project
Action: Run 35 reviews over 30 days
Expected: After 31st day, oldest reviews auto-purge while patterns remain in index

## Scenario 5: Pattern Learning Over Time
App: Startup SaaS
Reviews: 1st review finds timezone bug, 10th review finds related payment timing issue
Expected: Pattern index links these together, 15th review surfaces both proactively
```

**Step 2: Create CLAUDE.md for review system**

Create `CLAUDE.md` in root:

```markdown
# Review Plugin Development Guide

## Quick Start
- Review changes: `git diff`
- Test agents: Run `/review` command
- Check patterns: `.claude/reviews/patterns/index.json`
- View retention: `.claude/reviews/.env`

## Key Files
- `agents/pr-review-coordinator.agent.md` - Main orchestrator
- `agents/pattern-analyzer.agent.md` - Risk detection
- `agents/architecture-reviewer.agent.md` - Design patterns
- `agents/review-maintenance.agent.md` - Storage management

## Pattern Index
Reviews accumulate knowledge in `.claude/reviews/patterns/index.json`. Over time, the system learns:
- Security risks specific to each app
- N+1 query patterns and locations
- Legacy code interactions
- Performance bottlenecks

This index is per-project, so each SaaS app builds its own knowledge base.

## Testing New Changes
1. Make agent changes
2. Run `/review` on a real project (e.g., PSTrax)
3. Check synthesis report in `.claude/reviews/YYYY-MM-DD_HH-MM-SS/`
4. Verify patterns were discovered/updated
5. Commit with detailed message
```

**Step 3: Commit**

```bash
git add tests/scenarios.md CLAUDE.md
git commit -m "docs: add test scenarios and development guide"
```

---

## Task 8: Update Main Coordinator with Complete Workflow

**Files:**
- Modify: `agents/pr-review-coordinator.agent.md` (major rewrite)

**Step 1: Rewrite coordinator with all components**

Replace coordinator with enhanced version that includes:
1. Initialize storage and config
2. Load pattern index
3. Launch pattern analyzer (background)
4. Launch architecture reviewer (with pattern context)
5. Launch implementation reviewer (background)
6. Monitor all agents
7. Synthesize findings (incorporating patterns)
8. Update pattern index
9. Trigger maintenance (if enabled)
10. Generate final report with pattern learning summary

**Step 2: Update synthesis report format**

Enhance synthesis report to include:
```markdown
## 📚 Pattern Learning Summary

### Patterns Discovered
- 2 new security risks identified
- 1 legacy code interaction flagged
- 0 N+1 queries found

### Patterns Confirmed
- Timezone handling risk (seen in 3 previous reviews)
- Department permission model (seen in 8 previous reviews)

### Pattern Impact on This Review
- ⚠️ YELLOW: Changes interact with known timezone risk - verify test coverage
- ✅ GREEN: Follows established permission pattern - no concerns

### Next Review Actions
- Monitor: Changes to password hashing will trigger crypto audit
- Track: Payment timing changes against existing timing risks
```

**Step 3: Commit**

```bash
git add agents/pr-review-coordinator.agent.md
git commit -m "feat: integrate pattern indexing and learning into main review workflow"
```

---

## Task 9: Create PR and Documentation

**Files:**
- Create: `ENTERPRISE_FEATURES.md`
- Update: `README.md`

**Step 1: Create enterprise features guide**

Create `ENTERPRISE_FEATURES.md`:

```markdown
# Enterprise-Grade Features

## Pattern-Aware Reviews

Each codebase learns its own patterns. Reviews become smarter over time:

- **Day 1**: Discover 5 security risks, N+1 queries, legacy interactions
- **Day 30**: Same patterns now automatically checked in every review
- **Day 90**: System has learned what matters for this specific app

## Intelligent Storage Management

Reviews are kept for 30 days (configurable). Pattern knowledge is permanent:

```
.claude/reviews/
├── 2025-12-10_19-30-00/  ← Will be deleted after 30 days
│   ├── synthesis-report.md
│   └── architecture-findings.md
├── patterns/
│   └── index.json  ← Grows smarter, never deleted
```

## Security Risk Tracking

The system tracks:
- **High-risk patterns**: Timezone handling, payment flows, authentication
- **Query performance**: Identified N+1 queries and their locations
- **Legacy interactions**: Code that affects multiple systems
- **Dependencies**: How changes interact with known risks

## For SaaS Teams

Managing 1000+ apps means each has unique patterns:

- App 1 (Healthcare): Learns HIPAA compliance patterns
- App 2 (Fintech): Learns payment processing patterns
- App 3 (Analytics): Learns performance patterns

Each app builds its own knowledge base while the review system is identical.

## Configuration

```bash
# View current settings
/configure view

# Adjust retention period
/configure set REVIEW_RETENTION_DAYS 45

# Disable automatic purging (if you have external storage)
/configure set AUTO_PURGE_ENABLED false
```
```

**Step 2: Update README with enterprise section**

Add to README.md after quick-start:

```markdown
## Enterprise Features

### Pattern Learning
The review system learns your codebase's patterns. Each review adds to a pattern index:
- Security risks discovered in your codebase
- N+1 queries and their locations
- Legacy code interactions
- Performance characteristics

Future reviews automatically check against these patterns.

### Configurable Storage
Reviews are retained for 30 days (configurable). Pattern knowledge persists:

```bash
/configure set REVIEW_RETENTION_DAYS 45
/configure view
```

### Per-Project Knowledge
Each codebase builds its own pattern index. Perfect for SaaS teams:
- App 1 learns its own patterns
- App 2 learns its own patterns
- Reviews get smarter over time for each project
```

**Step 3: Commit and prepare PR**

```bash
git add ENTERPRISE_FEATURES.md README.md docs/plans/2025-12-10-enterprise-review-system.md
git commit -m "docs: add enterprise features documentation"
git push origin main
```

---

## Acceptance Criteria

✅ Pattern index created and updated after each review
✅ Reviews automatically purged after retention period
✅ New patterns discovered and stored
✅ Coordinator workflow: config → analyze → review → synthesize → learn → purge
✅ `/configure` command allows users to modify settings
✅ CLAUDE.md documents development workflow
✅ Synthesis reports include pattern learning summary
✅ Documentation explains per-project pattern indexing
✅ Test scenarios document realistic enterprise workflows
✅ All code is committed with clear messages

---

## Summary

This plan transforms the review system into an enterprise-grade tool that learns from each codebase. Instead of generic reviews, the system becomes smarter for each project it works on—discovering security risks, tracking legacy code, and flagging performance patterns that matter for that specific application.

For a SaaS company managing 1000+ apps, each application gets personalized review intelligence while using the same plugin.
