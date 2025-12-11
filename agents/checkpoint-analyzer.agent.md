# commit-checkpoint-analyzer Agent

Specialized agent for analyzing staged git changes and generating checkpoint verification reports.

## Purpose

Analyzes code changes for a single commit using CodeRabbit, parses findings, and generates structured checkpoint verification data. This agent bridges CodeRabbit CLI output and the commit-checkpoint command's interactive flow.

## When to Use

Called automatically by the `/commit-checkpoint` command to verify staged changes before committing.

## Inputs

```json
{
  "commit_message": "Add blood product expiration check logic",
  "branch_name": "feature/PSTRAX-2039-blood-product-expiration-alerts",
  "staged_files": ["app/Blood/Console/Commands/CheckBloodProductExpirationsCommand.php", ...],
  "skip_review": false,
  "timeout_seconds": 60
}
```

## Processing Flow

1. **Verify staged changes exist**
   - Check that there are actual file changes staged
   - Extract diff using `git diff --cached`

2. **Run CodeRabbit analysis**
   - Execute: `coderabbit review --base development --prompt-only --cached`
   - Capture output with timeout
   - Handle timeout gracefully (warn, don't block)

3. **Parse CodeRabbit output**
   - Extract findings from CodeRabbit text output
   - Categorize by severity: CRITICAL, IMPORTANT, MINOR
   - Build structured findings array

4. **Calculate quality score**
   - Base: 10.0
   - CRITICAL: -3 per issue
   - IMPORTANT: -1 per issue
   - MINOR: -0.1 per issue
   - Min: 0.0, Max: 10.0

5. **Analyze checkpoint scope**
   - Determine what area of code this commit affects
   - Categorize: "CommandLogic", "NotificationSystem", "Testing", etc.
   - Assess if changes represent working checkpoint

6. **Generate checkpoint analysis**
   - Can this commit stand alone?
   - Are tests present and passing?
   - Are critical dependencies satisfied?
   - Is it architecturally sound?

## Outputs

```json
{
  "success": true,
  "commit_message": "Add blood product expiration check logic",
  "scope": "CheckBloodProductExpirationsCommand",
  "review": {
    "tool": "CodeRabbit",
    "timestamp": "2025-12-10T22:15:00Z",
    "quality_score": 8.2,
    "critical_issues": 0,
    "important_issues": 1,
    "minor_issues": 0,
    "timeout": false,
    "findings": [
      {
        "severity": "IMPORTANT",
        "file": "app/Blood/Console/Commands/CheckBloodProductExpirationsCommand.php",
        "line": 125,
        "title": "Missing auth context",
        "description": "Command creates alerts without setting created_by_user_id",
        "suggestion": "Set explicit system user or document null handling"
      }
    ]
  },
  "checkpoint": {
    "is_working_checkpoint": true,
    "checkpoint_type": "CommandLogic",
    "has_tests": true,
    "tests_count": 5,
    "dependencies_satisfied": true,
    "architectural_soundness": "GOOD",
    "recommendation": "APPROVED"
  },
  "warnings": [],
  "timing": {
    "coderabbit_duration_seconds": 8.3,
    "analysis_duration_seconds": 0.5
  }
}
```

## Error Handling

### CodeRabbit Timeout (>60 seconds)
- Set `review.timeout: true`
- Return partial results if available
- Warning message: "CodeRabbit analysis timed out"
- Recommendation: Allow commit but flag checkpoint as "TIMEOUT_REVIEW"

### CodeRabbit Failure
- Attempt retry once
- If still fails: return `success: false` with error details
- Do NOT block commit - let caller decide

### No staged changes
- Return error: "No staged changes detected"
- Return exit code for command to handle

### Parse errors
- Log parsing failures
- Continue with available findings
- Mark checkpoint as "PARTIAL_REVIEW"

## Special Behaviors

### When skip_review=true
- Skip CodeRabbit entirely
- Use heuristics only (tests exist, no obvious issues)
- Return checkpoint analysis based on code inspection
- Quality score: null (not applicable)
- Recommendation: "SKIPPED_REVIEW"

### When CodeRabbit warns on non-existent lines
- Filter out spurious findings
- Focus on actual file changes
- Log filtered findings for debugging

### Integration with git hook
- When called from git hook, output is more concise
- Include exit code recommendation (0 = OK, 1 = CRITICAL)
- Suppress interactive UI elements

## Configuration

Can be customized via `.claude/config.yaml`:

```yaml
checkpoint:
  enabled: true
  coderabbit_timeout_seconds: 60
  skip_review_for_hotfixes: true
  min_quality_score: 7.0
  block_on_critical: true
  warn_on_important: true
  allow_minor_warnings: true
```

## Return Value

Returns JSON object that `/commit-checkpoint` command parses and displays to user.

Exit codes:
- 0: Analysis successful
- 1: Analysis failed (CodeRabbit error, parse error)
- 2: No staged changes

## Implementation Notes

- Uses ripgrep internally for finding-extraction
- Handles diff context for file:line mapping
- Assumes CodeRabbit CLI installed and in PATH
- Caches git diff to avoid re-running
