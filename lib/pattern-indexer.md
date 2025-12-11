# Pattern Index System

The pattern index tracks codebase-specific patterns discovered over time, enabling smarter reviews as the system learns about your project.

## Pattern Index Structure

The pattern index is stored as JSON in `.claude/reviews/patterns/index.json`:

```json
{
  "version": "1.0",
  "codebase": {
    "name": "pstrax-monolith",
    "path": "/Users/jordan/Sites/pstrax-laravel/apps/pstrax-monolith",
    "last_updated": "2025-12-10T19:30:00Z"
  },
  "patterns": {
    "security_risks": [
      {
        "id": "timezone-handling-risk",
        "severity": "high",
        "title": "Timezone Offset Mapping Risk",
        "description": "Departments use TimeZone field for UTC offset; legacy code assumes Central time (-6)",
        "locations": [
          "app/Blood/Console/Commands/CheckBloodProductExpirationsCommand.php:95",
          "app/Blood/Models/BloodProduct.php:142"
        ],
        "impact": "Incorrect timezone conversion could trigger alerts at wrong times for departments outside Central timezone",
        "mitigation": "Always use calculateTargetOffset() method; verify test coverage for multiple timezones",
        "last_seen": "2025-12-10T19:30:00Z",
        "occurrences": 3,
        "status": "active"
      },
      {
        "id": "payment-timing-risk",
        "severity": "high",
        "title": "Payment Processing Timing Assumptions",
        "description": "Payment queue jobs assume UTC processing; department-local times not considered",
        "locations": [
          "app/Billing/Jobs/ProcessPayment.php:67"
        ],
        "impact": "Could process payments for wrong billing period in multi-timezone scenarios",
        "mitigation": "Use Queue::onConnection with explicit timezone handling",
        "last_seen": "2025-12-09T14:22:00Z",
        "occurrences": 1,
        "status": "active"
      }
    ],
    "n_plus_one_risks": [
      {
        "id": "user-watchers-query",
        "severity": "medium",
        "title": "User Watchers N+1 Query",
        "location": "app/Alerts/Listeners/SendEntityAlertNotifications.php:50",
        "description": "Initial implementation loaded users in loop; now uses whereIn() for batch loading",
        "fix_applied": "Changed User::find($id) to User::whereIn('UserID', $ids)->get()",
        "status": "mitigated",
        "last_reviewed": "2025-12-10T19:30:00Z",
        "related_pattern": "batch-loading-pattern"
      },
      {
        "id": "department-permissions-risk",
        "severity": "medium",
        "title": "Department Permission Check in Loop",
        "location": "app/Inventory/Services/ContainerService.php:156",
        "description": "Permission check happens per-container instead of bulk checking",
        "fix_recommendation": "Use ActionHasEntityTypePermissions trait with batch validation",
        "status": "active",
        "last_reviewed": "2025-12-08T10:15:00Z"
      }
    ],
    "legacy_interactions": [
      {
        "id": "equipment-alerts-renamed",
        "severity": "medium",
        "title": "EquipmentAlerts Table Renamed",
        "description": "Legacy EquipmentAlerts table renamed to VehicleAlert in Fleet module refactor",
        "old_reference": "EquipmentAlerts",
        "new_reference": "VehicleAlert",
        "legacy_code_locations": [
          "database/migrations/2023_05_15_create_equipment_alerts.php"
        ],
        "status": "monitored",
        "last_audit": "2025-12-10T19:30:00Z",
        "risk": "Code that references old table name directly will break"
      },
      {
        "id": "dual-signature-requirement",
        "severity": "high",
        "title": "DEA Dual Signature Requirement",
        "description": "Controlled substance operations require dual signatures for audit compliance",
        "affects_modules": ["ControlledSubstance"],
        "enforcement": "App/ControlledSubstance/Actions/RequiresDualSignature trait",
        "status": "active",
        "last_audit": "2025-12-09T14:00:00Z",
        "risk": "Bypassing dual signature violates DEA compliance"
      }
    ],
    "established_patterns": [
      {
        "id": "service-action-data-architecture",
        "title": "Service-Action-Data Pattern",
        "description": "Standard architecture for all modules: Services delegate to Actions, Actions use Data DTOs",
        "example": "App/Blood/Services/BloodService.php → App/Blood/Actions/CreateBloodAlert.php → App/Blood/Data/BloodAlertData.php",
        "enforcement": "Code generation via php artisan generate:entity",
        "status": "established",
        "occurrences_in_codebase": 14
      },
      {
        "id": "eager-loading-requirement",
        "title": "Eager Loading Best Practice",
        "description": "All queries that access relationships must use eager loading to prevent N+1",
        "standard": "Use with(), load(), or QueryBuilder::for() for relationships",
        "violations_found": 2,
        "status": "enforced"
      }
    ]
  },
  "statistics": {
    "total_reviews": 42,
    "last_review": "2025-12-10T19:30:00Z",
    "patterns_discovered": 8,
    "security_risks_found": 5,
    "n_plus_one_risks_found": 3,
    "legacy_interactions_tracked": 2,
    "established_patterns": 6
  },
  "learning_timeline": [
    {
      "date": "2025-11-15",
      "review_count": 10,
      "patterns_discovered": 3,
      "summary": "Initial scan discovered timezone handling risk, payment timing issue, user loading pattern"
    },
    {
      "date": "2025-12-01",
      "review_count": 25,
      "patterns_discovered": 5,
      "summary": "Added department permission checks, equipment alerts renamed, DEA signature requirement"
    },
    {
      "date": "2025-12-10",
      "review_count": 42,
      "patterns_discovered": 8,
      "summary": "Confirmed established patterns, escalated timezone risk to HIGH severity"
    }
  ]
}
```

## Pattern Categories

### Security Risks
- **Severity**: high, medium, low
- **Tracking**: locations, impact, mitigation
- **Status**: active, mitigated, deprecated
- **Learning**: occurrences increase with each review

### N+1 Query Risks
- **Tracking**: exact location, description, status
- **Status**: active, mitigated, monitored
- **Pattern Recognition**: repeated patterns get higher priority

### Legacy Interactions
- **Tracking**: what changed, where old references exist
- **Risk**: potential breaking points
- **Audit**: when last reviewed

### Established Patterns
- **Tracking**: what the pattern is, examples
- **Enforcement**: how violations are caught
- **Count**: how many times used correctly

## How Patterns Are Updated

1. **During Review**: Architecture reviewer identifies new patterns
2. **Pattern Analyzer**: Checks code against known patterns
3. **Index Update**: New patterns added, existing patterns updated
4. **Learning**: Same pattern found multiple times → priority increases
5. **Cleanup**: Deprecated patterns marked but kept for history

## Pattern Impact on Reviews

When reviewing code, the system:

1. Loads current pattern index
2. Checks changes against all patterns
3. Flags if change involves a known risk
4. Suggests mitigations if pattern has been mitigated before
5. Escalates severity if pattern seen multiple times

Example: If timezone risk appears 3 times, severity moves from medium to high, and new reviews automatically test for timezone issues.

## Accessing Pattern Data

In agents or prompts, reference patterns like:

```
Known security risks in this codebase:
- Timezone handling (HIGH) - seen 3 times
- Payment timing (HIGH) - seen 1 time
- Permission checks (MEDIUM) - seen 2 times

Review this change for timezone impact.
```

This context helps new reviewers understand what matters in this specific codebase.
