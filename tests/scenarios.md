# Enterprise Review System - Test Scenarios

These scenarios represent realistic use cases for the review plugin on enterprise SaaS applications.

## Scenario 1: Security Risk Discovery & Pattern Learning

### Setup
- **App**: Multi-tenant SaaS with payment processing
- **Team**: 5 developers
- **Reviews**: First week of using plugin

### Flow

**Review 1 - Initial Discovery**
```
Changes: Update password hashing to bcrypt with cost factor 12
File: app/Auth/Actions/HashPassword.php:34

Pattern Analyzer finds:
- No crypto vulnerabilities (bcrypt correct)
- But marks password handling as "security_critical"

Architecture Reviewer finds:
- Clean implementation following Laravel conventions
- Recommends logging all password operations for audit

Pattern Index updated:
- New pattern: "crypto-sensitive-operation"
- Severity: HIGH
- Locations: [app/Auth/Actions/HashPassword.php:34]
```

**Review 3 - Pattern Confirmed**
```
Changes: Add password reset email template
File: app/Auth/Notifications/PasswordReset.php:12

Pattern Analyzer:
- Recognizes this touches crypto-sensitive-operation pattern
- Verifies password is properly hashed before notification
- Escalates attention to HIGH

Architecture Reviewer:
- Checks: "Is this respecting password handling patterns?"
- Confirms proper flow: hash → store → notify with link

Pattern Index updated:
- crypto-sensitive-operation: occurrences = 2
- Severity remains HIGH
- New location added: app/Auth/Notifications/PasswordReset.php:12
```

**Review 8 - Pattern Escalation**
```
Changes: Add 2FA token validation
File: app/Auth/Services/TwoFactorService.php:67

Pattern Analyzer:
- Detects this is 3rd interaction with crypto-sensitive-operation
- Escalates severity to CRITICAL
- Flags for mandatory security review

Architecture Reviewer:
- Additional scrutiny on crypto operations
- Checks: token generation entropy, storage, comparison timing
- Requires comprehensive test coverage

Synthesis Report:
- 🔴 RED LIGHT: CRITICAL - Crypto operations touched
- 🟡 YELLOW LIGHT: Requires security team sign-off
- Readiness Score: 7.5/10 (good, but needs security review)

Pattern Index:
- crypto-sensitive-operation: occurrences = 3, severity = CRITICAL
- Now appears in every review's checklist
```

**Impact**: By Review 8, team has learned that crypto is sensitive. New crypto-related PRs automatically get extra scrutiny without needing human documentation.

---

## Scenario 2: N+1 Query Detection & Mitigation

### Setup
- **App**: Analytics dashboard with 1000+ customer accounts
- **Issue**: Reports sometimes timeout with lots of accounts

### Flow

**Review 1 - Query Problem Found**
```
Changes: Add customer revenue report
File: app/Analytics/Services/RevenueReportService.php:45

Code:
```php
foreach ($customers as $customer) {
    $revenue = $customer->invoices()->sum('total');  // N+1!
    $reportLines[] = [..., 'revenue' => $revenue];
}
```

Pattern Analyzer:
- Finds query in loop
- Reports: "N+1 query risk - customer->invoices() called per customer"
- Severity: MEDIUM

Architecture Reviewer:
- Confirms N+1 problem
- Score: 6/10 (functionality works, performance problem)

Synthesis Report:
- 🔴 RED LIGHT: N+1 query detected
- Must be fixed before merge

Pattern Index created:
```json
{
  "id": "revenue-report-n-plus-one",
  "severity": "medium",
  "location": "app/Analytics/Services/RevenueReportService.php:45",
  "description": "Customer invoices summed in loop without batching",
  "status": "active",
  "occurrences": 1
}
```
```

**Developer Fixes & Review 2 - Fix Verification**
```
Changes: Use eager loading to fix N+1

Code:
```php
$customers = Customer::with('invoices')->get();
foreach ($customers as $customer) {
    $revenue = $customer->invoices->sum('total');  // Works with already-loaded data
    $reportLines[] = [..., 'revenue' => $revenue];
}
```

Pattern Analyzer:
- Scans changes
- Finds: N+1 pattern location was modified
- Status: MITIGATED
- Confirms fix: eager loading + in-memory sum

Pattern Index updated:
```json
{
  "id": "revenue-report-n-plus-one",
  "severity": "medium",
  "status": "mitigated",
  "fix_applied": "Changed to eager loading with with('invoices')",
  "occurrences": 1
}
```

Synthesis Report:
- ✅ GREEN LIGHT: N+1 issue resolved
- Score: 9/10
- Recommendation: Use this pattern for similar reports
```

**Reviews 5, 12, 19 - Pattern Reconfirmed**
```
New PRs that add similar reports:
- Review 5: Add profit margin report
- Review 12: Add customer segment analysis
- Review 19: Add historical revenue trends

Each review:
- Pattern Analyzer automatically checks for N+1
- Flags if similar problem appears
- Reviews that follow pattern get GREEN light
- Team learns to use eager loading consistently

Pattern Index after Review 19:
- revenue-report-n-plus-one: occurrences = 4
- established_patterns: "eager-loading-requirement"
- All developers now know: "Use eager loading for collection calculations"
```

**Impact**: From one N+1 problem, team learns the pattern. All subsequent reports follow best practice. No more query timeout issues.

---

## Scenario 3: Legacy Code Interaction Detection

### Setup
- **App**: 10-year-old Laravel application being modernized
- **Challenge**: Old code patterns still in use
- **Goal**: Safely migrate away from legacy approaches

### Flow

**Review 1 - Legacy Code Identified**
```
Changes: Add new user profile API endpoint
File: app/Users/Http/Controllers/ProfileController.php:23

New code checks user permission via:
```php
if (Auth::user()->hasPermission('view_profile')) {  // New way
    ...
}
```

But other code still uses:
```php
if ($GLOBALS['USER_PERMS'][$id]['view_profile']) {  // Old way
}
```

Pattern Analyzer:
- Finds reference to legacy pattern
- Flags as legacy interaction
- New code uses proper auth, old code uses globals
- Status: MIGRATING (not deprecated yet, still in use)

Pattern Index created:
```json
{
  "id": "legacy-global-auth",
  "severity": "high",
  "title": "Global Auth Still In Use",
  "description": "Old code uses $GLOBALS['USER_PERMS']; new code uses Auth::user()",
  "old_locations": [
    "routes/web.php:15",
    "app/Reports/ReportGenerator.php:78",
    "app/Billing/InvoiceService.php:45"
  ],
  "new_preferred_pattern": "Auth::user()->hasPermission()",
  "status": "migrating",
  "risk": "Mixing auth methods could cause permission inconsistencies",
  "occurrences": 1
}
```

Synthesis Report:
- ⚠️ YELLOW LIGHT: Interacts with legacy auth system
- Question: "Are permissions consistent between old and new auth?"
- Recommendation: Verify this endpoint works with both old & new code
```

**Reviews 3, 7, 15 - Legacy Pattern Confirmed**
```
As team adds features:
- Review 3: Add admin dashboard (uses Auth::user())
- Review 7: Add invoice generation (still uses $GLOBALS somehow)
- Review 15: Add notification service (uses Auth::user())

Pattern Index evolves:
- Review 3: "Pattern confirmed in admin feature"
- Review 7: "Found another $GLOBALS reference - must be in Reports"
- Review 15: Recognizes migration pattern, asks for deprecation timeline

Synthesis Report for Review 7:
- 🔴 RED LIGHT: Uses deprecated global auth
- Must migrate before merge
- Help: See migration guide in docs
```

**Review 30 - Migration Complete**
```
Changes: Remove last use of global auth
File: app/Reports/ReportGenerator.php (refactored)

Pattern Analyzer:
- Scans for old references
- Finds none remaining
- Marks pattern as "DEPRECATED"

Architecture Reviewer:
- Confirms all auth now uses Laravel Auth
- No permission inconsistencies
- Score: 10/10

Pattern Index:
```json
{
  "id": "legacy-global-auth",
  "status": "deprecated",
  "migrated_date": "2025-12-10",
  "final_occurrences": 15,
  "migration_completed": true
}
```

Synthesis Report:
- ✅ GREEN LIGHT: Final legacy auth removal complete
- Celebration: "Legacy auth migration finished"
- New team members: Won't see this pattern in future reviews
```

**Impact**: Plugin tracked legacy code across 30 reviews. Team safely migrated without auth bugs.

---

## Scenario 4: Timezone Risk Escalation

### Setup
- **App**: Multi-location SaaS with timezone-sensitive operations
- **Problem**: Alerts trigger at wrong times for different locations
- **Knowledge**: Timezone handling is critical and error-prone

### Pattern Learning Timeline

```
Day 1-2 (Reviews 1-5):
- Pattern discovered: "Timezone offset mapping risk"
- Severity: MEDIUM
- Known issue in alert scheduling

Day 3-5 (Reviews 6-15):
- Pattern confirmed 2x in payment processing
- "Payment timing could be wrong" - found
- Severity escalates to HIGH
- Occurrence count: 3

Day 6-7 (Reviews 16-25):
- Pattern confirmed again in alert scheduling
- New interaction found with location configuration
- Severity escalates to CRITICAL
- Occurrence count: 5

Day 8+ (Reviews 26+):
- Every review of timezone-related code gets extra scrutiny
- Tests for multiple timezones required
- Architecture reviewer has timezone context
- Team knows: "Timezone changes need careful attention"
```

**Key Insight**: Pattern learning makes the system more intelligent about this app's specific risks. By Day 8, the plugin knows timezone is critical for this codebase.

---

## Scenario 5: Review Storage & Purging

### Setup
- **Team**: Active development, 1-2 reviews per day
- **Retention**: 30 days (default)
- **Review Period**: 60 days

### Timeline

```
Day 1-30:
- Reviews accumulate: Days 1-30 present in storage
- Storage grows steadily
- No purging (nothing older than 30 days)
- Example: 30-45 reviews, ~1.2 MB

Day 31:
- Review on Day 31 triggers maintenance
- Day 1 review (31 days old) is purged
- Storage freed: ~50 KB
- Pattern index stays (knowledge persists)

Day 60:
- Reviews 1-30 all purged
- Only Days 31-60 remain
- Storage stable at 1.2 MB
- Pattern index has learned from all 60 reviews

Day 90:
- 90 days of learnings in pattern index
- Only 30 days of reviews stored
- Pattern index: 42 reviews analyzed, 12 patterns discovered
- Next review: can reference all 90 days of learning
```

**Storage Breakdown at Day 60**:
```
.claude/reviews/
├── .env                              (config)
├── patterns/
│   ├── index.json                    (42 reviews analyzed!)
│   └── maintenance.log               (purge history)
├── 2025-11-31_14-30-00/              (Day 31)
├── 2025-12-01_09-15-00/              (Day 32)
├── ... (30 days of reviews)
└── 2025-12-30_17-45-00/              (Day 60)

Total storage: 1.2 MB
Pattern intelligence: 90 days
Knowledge retention: Forever
```

---

## Scenario 6: Enterprise Team Workflow

### Setup
- **Team**: 8 engineers
- **Apps**: 5 internal SaaS products
- **Goal**: Consistent code quality across team

### Workflow

```
Engineer A: Reviews on App 1
- /review on App 1
- App 1 learns its patterns
- Pattern index grows with App 1's knowledge

Engineer B: Reviews on App 2
- /review on App 2
- App 2 learns its patterns
- Both apps have their own knowledge bases

Engineer C: Transfers from App 1 to App 2
- Works on App 2
- /review runs
- App 2's pattern index gives context
- Learns App 2's specific risks
- No knowledge of App 1 (different codebase)

Engineer A: Reviews code for App 1
- /review
- Gets context from App 1's 30 reviews
- Knows: timezone is critical, N+1 queries common, etc.
- Review is more intelligent than it was on Day 1

3 Months Later:
- App 1: 90 reviews, 12 patterns discovered, highly optimized
- App 2: 85 reviews, 15 patterns discovered, patterns different from App 1
- App 3: 45 reviews, 8 patterns discovered
- App 4: 72 reviews, 13 patterns discovered
- App 5: 38 reviews, 7 patterns discovered

Each app is smarter about its own risks. Team knowledge grows.
```

---

## Running These Scenarios

To test these scenarios:

1. **Pick a test Laravel project** (any multi-team SaaS)
2. **Run initial reviews** - `/review` multiple times
3. **Check pattern learning** - `cat .claude/reviews/patterns/index.json | jq`
4. **Verify purging** - Set `REVIEW_RETENTION_DAYS=7`, run reviews for 10 days
5. **Test escalation** - Make same-type change multiple times, watch severity increase

## Success Metrics

✅ Pattern index grows with each review
✅ Patterns escalate as they're confirmed multiple times
✅ Reviews reference pattern context (synthesis reports mention patterns)
✅ Old reviews purged, pattern knowledge persists
✅ Team learns specific risks for each codebase
✅ Reviews become more intelligent over time
