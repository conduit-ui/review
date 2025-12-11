# Review Plugin Development Guide

This guide helps Claude Code developers working on the conduit-ui/review plugin understand the system architecture and workflow.

## Quick Start

### First Review
```bash
cd /path/to/your-laravel-project
/review
```

Results appear in `.claude/reviews/YYYY-MM-DD_HH-MM-SS/`

### Configuration
```bash
/configure                    # Interactive setup
/configure view               # Show settings
/configure set REVIEW_RETENTION_DAYS 45
```

### View Pattern Index
```bash
cat .claude/reviews/patterns/index.json | jq .
```

## Architecture Overview

```
/review (slash command)
  ↓
pr-review-coordinator (main orchestrator)
  ├─ STEP 0: Initialize storage & load config
  ├─ STEP 1: Launch agents in parallel
  │   ├─ pattern-analyzer (background)
  │   ├─ architecture-reviewer (background)
  │   └─ implementation-reviewer (background)
  ├─ STEP 2: Monitor & collect findings
  ├─ STEP 3: Synthesize results
  ├─ STEP 4: Update pattern index
  └─ STEP 4: Trigger maintenance (background)
```

## Key Components

### Agents

1. **pr-review-coordinator** (`agents/pr-review-coordinator.agent.md`)
   - Orchestrates the entire review workflow
   - Launches other agents as background tasks
   - Synthesizes findings into readiness brief
   - Manages pattern index updates and maintenance

2. **pattern-analyzer** (`agents/pattern-analyzer.agent.md`)
   - Analyzes changes against known patterns
   - Identifies security risks, N+1 queries, legacy interactions
   - Discovers new patterns
   - Returns structured pattern findings

3. **architecture-reviewer** (`agents/architecture-reviewer.agent.md`)
   - Deep Laravel architecture analysis
   - SOLID principle compliance
   - Test quality assessment
   - Performance analysis (N+1, eager loading)
   - Production readiness check
   - Pattern context-aware feedback

4. **implementation-reviewer** (provided by system)
   - Code quality and security
   - Type safety, null handling
   - Performance issues
   - CodeRabbit integration (optional)

5. **review-maintenance** (`agents/review-maintenance.agent.md`)
   - Purges old reviews per retention policy
   - Compacts pattern index
   - Logs maintenance actions
   - Runs automatically after reviews

6. **pattern-indexer** (`agents/pattern-indexer.agent.md`)
   - Updates pattern index with discoveries
   - Merges new findings with existing patterns
   - Tracks pattern evolution
   - Maintains learning timeline

### Configuration

- **Config File**: `reviews-config/.env.example`
- **Live Config**: `.claude/reviews/.env` (created per-project)
- **Key Settings**:
  - `REVIEW_RETENTION_DAYS`: How long to keep reviews (default: 30)
  - `AUTO_PURGE_ENABLED`: Auto-delete old reviews (default: true)
  - `PATTERN_INDEX_PATH`: Where patterns stored (default: `./.claude/reviews/patterns`)
  - `LOG_LEVEL`: Logging detail (default: info)

### Storage Structure

```
.claude/reviews/
├── .env                      # Project-specific configuration
├── patterns/
│   ├── index.json            # Pattern index (persists forever)
│   └── maintenance.log       # Maintenance history
├── YYYY-MM-DD_HH-MM-SS/      # Review session (expires)
│   ├── metadata.json
│   ├── synthesis-report.md
│   ├── architecture-findings.md
│   └── implementation-findings.md
├── YYYY-MM-DD_HH-MM-SS/      # Another review...
└── ...
```

## Pattern Index

The pattern index tracks codebase-specific patterns that grow more valuable over time.

### Structure

See `lib/pattern-indexer.md` for detailed structure. Contains:

- **Security Risks**: Vulnerabilities specific to this codebase
- **N+1 Query Patterns**: Known query problems and locations
- **Legacy Interactions**: Deprecated code that still affects new changes
- **Established Patterns**: Conventions that must be followed

### How It Works

1. **First Review**: Discovers initial patterns, adds to index
2. **Subsequent Reviews**: Confirms patterns (increments occurrence count), discovers new ones
3. **Pattern Evolution**: Severity escalates as patterns confirmed multiple times
4. **Learning**: Same pattern found 3x → severity escalates to critical

Example:
- Review 1: "Found timezone handling bug" → added to index
- Review 5: "Found same timezone bug again" → occurrences = 2
- Review 10: "Found another timezone-related issue" → occurrences = 3, severity escalates to HIGH
- Review 15: "Code respects timezone patterns" → review automatically checks timezone scenarios

## Development Workflow

### Working on Plugin Changes

1. **Read Current State**
   ```bash
   git log --oneline -5       # Recent commits
   git status                 # Current changes
   ```

2. **Understand Patterns**
   - Look at existing agents in `agents/`
   - Check `lib/pattern-indexer.md` for pattern structure
   - Review `ENTERPRISE_FEATURES.md` for design goals

3. **Make Changes**
   - Modify agent prompts in `agents/*.md`
   - Update configuration in `reviews-config/*.example`
   - Add new slash commands in `commands/*.md`

4. **Test Changes**
   - Install locally: `/plugin install review@conduit-ui`
   - Run review on real project: `/review`
   - Check `.claude/reviews/` output
   - Verify pattern index updated

5. **Commit**
   ```bash
   git add agents/ commands/ lib/
   git commit -m "feat: [description of change]"
   git push origin main
   ```

### Testing Pattern Learning

Create test scenario:

```bash
# Review 1
/review
# Check: .claude/reviews/patterns/index.json has patterns

# Review 2
/review
# Check: Same patterns have occurrences incremented

# Review 3
/review
# Check: Patterns with 3+ occurrences escalated to HIGH severity
```

## Key Files by Purpose

| Purpose | Files |
|---------|-------|
| Main workflow | `agents/pr-review-coordinator.agent.md` |
| Pattern analysis | `agents/pattern-analyzer.agent.md`, `lib/pattern-indexer.md` |
| Code review | `agents/architecture-reviewer.agent.md`, `agents/implementation-reviewer.agent.md` |
| Storage & cleanup | `agents/review-maintenance.agent.md`, `reviews-config/.env.example` |
| User interface | `commands/configure.md`, `commands/review.md` |
| Documentation | `README.md`, `ENTERPRISE_FEATURES.md` |

## Common Tasks

### Add New Review Category

1. Create new agent in `agents/new-category.agent.md`
2. Update coordinator to launch it in parallel
3. Add synthesis logic to combine findings
4. Update README with new capabilities

### Change Pattern Structure

1. Update `lib/pattern-indexer.md` with new fields
2. Update `agents/pattern-analyzer.agent.md` to discover new pattern type
3. Update `agents/pattern-indexer.agent.md` to handle merging
4. Backward-compatible: old patterns still work

### Add New Configuration Option

1. Add to `reviews-config/.env.example`
2. Update `agents/pr-review-coordinator.agent.md` STEP 0 to read it
3. Update agent that uses setting
4. Document in `/configure` command

## Testing with PSTrax

The plugin is tested against PSTrax Laravel monolith:

```bash
cd /Users/jordan/Sites/pstrax-laravel
/review
```

PSTrax provides real-world testing with:
- Multi-tenant architecture
- Complex patterns (timezone, payment processing, DEA compliance)
- Large codebase with legacy code
- Active development with PRs

## Pattern Index Growth

Track how the index grows over time:

```bash
# Day 1
cat .claude/reviews/patterns/index.json | jq '.statistics'
# {
#   "total_reviews": 1,
#   "patterns_discovered": 3
# }

# Day 7
cat .claude/reviews/patterns/index.json | jq '.statistics'
# {
#   "total_reviews": 7,
#   "patterns_discovered": 8,
#   "security_risks": 5,
#   "n_plus_one_risks": 2,
#   "legacy_interactions": 1
# }

# Day 30
cat .claude/reviews/patterns/index.json | jq '.statistics'
# {
#   "total_reviews": 42,
#   "patterns_discovered": 12,
#   "security_risks": 8,
#   "n_plus_one_risks": 3,
#   "legacy_interactions": 2,
#   "established_patterns": 6
# }
```

## Debugging

### Review Not Starting
```bash
# Check coordinator is launching agents
cat .claude/reviews/YYYY-MM-DD_HH-MM-SS/synthesis-report.md

# Check agent failures
/configure view  # Current settings
cat .claude/reviews/patterns/index.json | jq '.patterns'
```

### Patterns Not Updating
```bash
# Check pattern-indexer was called
cat .claude/reviews/patterns/maintenance.log

# Check index structure
cat .claude/reviews/patterns/index.json | jq '.learning_timeline'
```

### Storage Growing Too Large
```bash
# Check retention settings
/configure view

# See what will be purged
ls -lah .claude/reviews/
```

## Resources

- `README.md` - User-facing documentation
- `ENTERPRISE_FEATURES.md` - Feature overview for teams
- `lib/pattern-indexer.md` - Pattern index structure reference
- `docs/plans/2025-12-10-enterprise-review-system.md` - Implementation plan

## Contributing

This is an enterprise-grade plugin. When contributing:

1. **Maintain compatibility** - Changes should work with existing pattern indexes
2. **Document patterns** - New pattern types need documentation
3. **Test thoroughly** - Test against real Laravel projects
4. **Follow conventions** - Match existing agent structure and style
5. **Commit clearly** - Use Chris Beam commit message style

See `docs/plans/2025-12-10-enterprise-review-system.md` for detailed implementation reference.
