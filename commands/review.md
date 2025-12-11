Use the pr-review-coordinator agent to conduct a comprehensive parallel review of the current PR.

The agent will:
1. Launch Jordan agent for architecture and production readiness assessment
2. Launch CodeRabbit for implementation details and security scanning
3. Synthesize results into actionable brief for human reviewer
4. Generate trust/verify/fix buckets with readiness score
5. Calculate confidence level for merge readiness
6. Track review metadata for learning patterns

After the review completes, you'll get a structured brief with categorized feedback:
- ✅ GREEN LIGHTS: Architecture validated, no security issues
- ⚠️ YELLOW LIGHTS: Business logic verification needed
- 🔴 RED LIGHTS: Issues that must be fixed

The agent will then present categorized feedback by topic and ask:
"I have feedback in these categories: [Architecture, Security, Implementation, Testing].
Would you like to work through anything specific first?"

Options to follow up:
- Review a specific category in detail
- Show RED LIGHT (must-fix) issues
- Walk through YELLOW LIGHT (verification) topics
- Request human review with summary
- Handle manually with full findings

Context: $ARGUMENTS