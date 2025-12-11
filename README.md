# Code Review Agents

Comprehensive parallel PR review system with AI-powered architecture and implementation analysis. Orchestrates specialized reviewer agents to assess code quality, security, and production readiness—then synthesizes findings into actionable briefs with confidence scoring.

## Features

✨ **Parallel Reviews** - Simultaneous architecture and implementation analysis (50% faster)
📊 **Readiness Scoring** - Quantified PR readiness (0-10 scale)
🎯 **Categorized Feedback** - GREEN/YELLOW/RED light classification
🔍 **Security & Quality** - CodeRabbit integration + architecture validation
📈 **Learning Loop** - Improves recommendations over time

## Quick Start

### 1. Install Prerequisites
```bash
# CodeRabbit CLI
brew install coderabbit

# Git (usually pre-installed)
git --version
```

### 2. Install Package
```bash
# Via direct git installation
/plugin install review@github:conduit-ui/review
```

### 3. Run Review
```bash
# In your project with a PR
/review

# Results appear in .claude/reviews/[timestamp]/
```

## How It Works

```
Your PR
   ↓
Parallel Review:
├─ Architecture Assessment (design, patterns, test coverage)
└─ Implementation Review (code quality, security, issues)
   ↓
Synthesis Phase:
├─ Categorize findings (RED/YELLOW/GREEN)
├─ Calculate readiness score
└─ Generate actionable brief
   ↓
Output:
└─ .claude/reviews/[timestamp]/synthesis-report.md
```

## Scoring

| Score | Verdict | Meaning |
|-------|---------|---------|
| 9.0-10.0 | ✅ Ready with confidence | Minimal review needed |
| 8.0-8.9 | ⚠️ Ready for review | Author should be present for Q&A |
| 7.0-7.9 | ⚠️ Ready but expect questions | May need iteration |
| <7.0 | 🔴 Hold | Needs more work before merge |

## Agents

- **pr-review-coordinator** - Orchestrates parallel reviews
- **architecture-reviewer** - Design patterns & production readiness
- **implementation-reviewer** - Code quality, security, issues
- **commit-reviewer** - Pre-commit validation
- **checkpoint-analyzer** - Staged change verification

## Documentation

- [QUICKSTART.md](./docs/QUICKSTART.md) - 5-minute setup
- [ARCHITECTURE.md](./docs/ARCHITECTURE.md) - How it works
- [CUSTOMIZATION.md](./docs/CUSTOMIZATION.md) - Configure for your project
- [SCORING-MODEL.md](./docs/SCORING-MODEL.md) - Score calculation
- [EXAMPLES.md](./docs/EXAMPLES.md) - Language-specific examples

## Requirements

- **CodeRabbit CLI** (required) - Install from https://coderabbit.ai/cli
- **Git** (required)
- **GitHub CLI** (optional) - For better PR integration

## Configuration

Create `.claude/review-config.json` to customize:

```json
{
  "scoring": {
    "architecture_weight": 0.6,
    "implementation_weight": 0.4
  },
  "review": {
    "enableLearning": true,
    "reportDirectory": ".claude/reviews"
  }
}
```

## Integration

### GitHub Actions
```yaml
name: Code Review
on: [pull_request]
jobs:
  review:
    runs-on: ubuntu-latest
    steps:
      - uses: anthropics/claude-code@v1
        with:
          command: /review
```

### Pre-commit Hook
```bash
#!/bin/bash
/review
if grep -q "RED LIGHTS" .claude/reviews/*/synthesis-report.md; then
  exit 1
fi
```

## Requirements

- **CodeRabbit CLI** - [Install](https://coderabbit.ai/cli)
- **Git** - Usually pre-installed
- **GitHub CLI** (optional) - [Install](https://cli.github.com)

## Support

- 📖 Documentation: https://github.com/conduit-ui/review
- 🐛 Issues: https://github.com/conduit-ui/review/issues
- 💬 Discussions: https://github.com/conduit-ui/review/discussions

## License

MIT - See LICENSE file

Built with ❤️ using Claude Code
