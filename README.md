# Laravel Code Review Agents

Expert Laravel PR review system with specialized agents for architecture assessment, test coverage analysis, and production-readiness validation. Emphasizes expressive, maintainable code and deep pattern recognition.

## Features

🏗️ **Architecture Review** - Design patterns, SOLID principles, Laravel conventions
🧪 **Test-Driven Assessment** - Test quality, coverage analysis, edge cases
🎯 **Production Readiness** - Error handling, logging, performance implications
📊 **Readiness Scoring** - Quantified PR readiness (0-10 scale)
🔍 **Pattern Recognition** - Identifies anti-patterns and architectural concerns

## Quick Start

### 1. Add Marketplace
```bash
/plugin marketplace add conduit-ui/review
```

### 2. Install Plugin
```bash
/plugin install review@conduit-ui-marketplace
```

### 3. Run Review
```bash
# In your Laravel project
/review

# Results in .claude/reviews/[timestamp]/synthesis-report.md
```

## How It Works

```
Your Laravel PR
   ↓
Architecture Review:
├─ Design patterns & SOLID compliance
├─ Laravel convention adherence
├─ Test quality and coverage
├─ Production readiness
└─ Performance implications
   ↓
Synthesis:
├─ Categorize findings
├─ Calculate readiness score
└─ Generate actionable brief
```

## Agents

### pr-review-coordinator
Orchestrates reviews and synthesizes findings into actionable briefs with readiness scoring.

### architecture-reviewer (OttBot)
Deep Laravel architecture expert who analyzes:
- Design patterns (Repository, Service, Action, Factory patterns)
- SOLID principle compliance
- Test quality and test-driven design
- Production readiness (error handling, logging, monitoring)
- Performance implications (N+1 queries, eager loading, query optimization)
- Database migrations and rollback safety
- Code expressiveness and maintainability

Focuses on:
- **Clarity over cleverness** - code that's easy to understand
- **Early returns** - reducing nesting for readability
- **Expressive names** - descriptive variables and methods
- **Test quality** - comprehensive edge case coverage
- **Pattern correctness** - following established Laravel patterns

### implementation-reviewer (Optional)
Runs CodeRabbit analysis if installed. Focuses on:
- Code quality and consistency
- Security vulnerabilities
- Implementation details

### commit-reviewer
Validates commits meet Laravel standards:
- Test validation before commit
- Message formatting
- Conventional commit style

### checkpoint-analyzer
Verifies staged changes form working checkpoint before commit.

## Scoring

| Score | Verdict | Meaning |
|-------|---------|---------|
| 9.0-10.0 | ✅ Ready with confidence | Minimal review needed |
| 8.0-8.9 | ⚠️ Ready for review | Author should be present for Q&A |
| 7.0-7.9 | ⚠️ Ready but expect questions | May need iteration |
| <7.0 | 🔴 Hold | Needs more work before merge |

## Requirements

- **Git** (required)
- **CodeRabbit CLI** (optional) - for implementation review
  - Install: `brew install coderabbit`
  - Or skip for architecture-only review

## Laravel-Specific Features

### Understands Laravel Patterns
- Service-Action-Data architecture
- Query Builder and Eloquent patterns
- Test conventions and assertions
- Module-based organization
- Permission and authorization patterns
- Form Request validation
- Pipeline and middleware patterns

### Detects Common Issues
- N+1 query problems
- Missing eager loading
- Untested edge cases
- Database migration safety
- Test fixture misuse
- Silent failure patterns
- Type safety gaps

### Values Expressive Code
- Clean, readable code over clever code
- Early returns for readability
- Descriptive variable/method names
- Test-driven development
- Explicit over implicit
- Small, focused functions

## Configuration

Create `.claude/review-config.json`:

```json
{
  "scoring": {
    "architecture_weight": 0.8,
    "implementation_weight": 0.2
  },
  "review": {
    "enableLearning": true,
    "reportDirectory": ".claude/reviews",
    "coderabbitEnabled": false
  }
}
```

## Workflow Integration

### Pre-commit Hook
```bash
#!/bin/bash
/review
if grep -q "RED LIGHTS" .claude/reviews/*/synthesis-report.md; then
  exit 1
fi
```

### GitHub Actions
```yaml
name: Architecture Review
on: [pull_request]
jobs:
  review:
    runs-on: ubuntu-latest
    steps:
      - uses: anthropics/claude-code@v1
        with:
          command: /review
```

## What You Get

When you run `/review`, you receive:

**synthesis-report.md** with:
- Readiness score (0-10)
- ✅ GREEN LIGHTS (trust these)
- ⚠️ YELLOW LIGHTS (verify with author)
- 🔴 RED LIGHTS (must fix)
- Questions for author
- Detailed findings by category

## Support & Contributing

- 📖 Source: https://github.com/conduit-ui/review
- 🐛 Issues: https://github.com/conduit-ui/review/issues
- 💬 Discussions: https://github.com/conduit-ui/review/discussions

## License

MIT - See LICENSE file

Built with ❤️ for Laravel developers using Claude Code
