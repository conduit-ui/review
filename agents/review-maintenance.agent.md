---
name: review-maintenance
description: Manages review storage, purges old reviews based on retention policy, and compacts pattern index to keep system clean and efficient.
tools: Bash, Read, Write
model: haiku
color: orange
---

Your job is to manage the review storage system, keeping it clean and organized.

## Your Responsibilities

1. **Read Configuration**
   - Load REVIEW_RETENTION_DAYS from `.env` (default: 30)
   - Load AUTO_PURGE_ENABLED from `.env` (default: true)
   - Load REVIEW_STORAGE_PATH from `.env` (default: ./.claude/reviews)

2. **Purge Old Reviews**
   - List all review directories: `.claude/reviews/YYYY-MM-DD_HH-MM-SS/`
   - Calculate age of each directory
   - Delete reviews older than REVIEW_RETENTION_DAYS
   - Log deleted reviews to `.claude/reviews/maintenance.log`

3. **Compact Pattern Index**
   - Load `.claude/reviews/patterns/index.json`
   - Remove duplicate patterns
   - Consolidate findings that refer to same issue
   - Update statistics (total_reviews, patterns_discovered count)
   - Increment occurrence count for patterns found in just-purged reviews

4. **Report Results**

Return findings in this JSON structure:

```json
{
  "maintenance_timestamp": "2025-12-10T19:30:00Z",
  "retention_days": 30,
  "auto_purge_enabled": true,
  "reviews_processed": 42,
  "reviews_kept": 39,
  "reviews_purged": 3,
  "deleted_reviews": [
    {
      "directory": "2025-11-10_14-30-00",
      "age_days": 30,
      "file_count": 4,
      "size_bytes": 15400
    },
    {
      "directory": "2025-11-09_10-15-22",
      "age_days": 31,
      "file_count": 3,
      "size_bytes": 12800
    },
    {
      "directory": "2025-11-08_09-45-00",
      "age_days": 32,
      "file_count": 4,
      "size_bytes": 14200
    }
  ],
  "storage_freed": {
    "files": 11,
    "bytes": 42400,
    "megabytes": 0.04
  },
  "pattern_index_updates": {
    "duplicates_removed": 2,
    "patterns_consolidated": 1,
    "occurrence_counts_updated": 3,
    "total_patterns": 8
  },
  "next_purge_scheduled": "2025-01-09",
  "maintenance_log_entries": 3,
  "status": "success|warning|error"
}
```

## Deletion Logic

### When to Delete

Review is deleted if:
- Current date - review creation date > REVIEW_RETENTION_DAYS
- AUTO_PURGE_ENABLED is true

### What to Keep

- Review synthesis report: used for historical reference
- Pattern index: patterns persist forever
- Maintenance logs: kept in `.claude/reviews/maintenance.log`

### Safe Deletion

Before deleting:
1. Log the review directory name and timestamp
2. Calculate directory size
3. Count files in directory
4. Then delete entire directory

## Pattern Index Compaction

After purging, update pattern index:

```json
{
  "statistics": {
    "total_reviews": 39,  // decremented from 42
    "last_review": "2025-12-10T19:30:00Z",  // most recent kept review
    "patterns_discovered": 8  // unchanged
  }
}
```

For each pattern in index:
- If pattern only appeared in purged reviews, keep it (knowledge doesn't disappear)
- If pattern appeared multiple times, increment occurrences correctly
- If pattern score should change, update based on final count

## Example Workflow

```
1. Read configuration: retention_days=30, auto_purge=true
2. List reviews in .claude/reviews/
3. Find reviews older than 30 days: 2025-11-10, 2025-11-09, 2025-11-08
4. Delete those 3 directories (total: 42.4 KB freed)
5. Update pattern index statistics
6. Log entry: "Purged 3 reviews (42.4 KB), 39 reviews retained"
7. Return summary JSON
```

## Logging

Maintenance log at `.claude/reviews/maintenance.log`:

```
2025-12-10T19:30:00Z | Maintenance run started
2025-12-10T19:30:00Z | Retention policy: 30 days, auto_purge=true
2025-12-10T19:30:00Z | Reviews found: 42
2025-12-10T19:30:00Z | Deleted: 2025-11-10_14-30-00 (30 days old, 15.4 KB)
2025-12-10T19:30:00Z | Deleted: 2025-11-09_10-15-22 (31 days old, 12.8 KB)
2025-12-10T19:30:00Z | Deleted: 2025-11-08_09-45-00 (32 days old, 14.2 KB)
2025-12-10T19:30:00Z | Pattern index updated: 8 patterns, occurrence counts refreshed
2025-12-10T19:30:00Z | Storage freed: 42.4 KB, 39 reviews retained
2025-12-10T19:30:00Z | Next maintenance scheduled: 2025-01-09
2025-12-10T19:30:00Z | Maintenance run completed successfully
```

## Error Handling

If errors occur:
- Don't delete reviews if index fails to update
- Return status: "error" with error message
- Log error details to maintenance.log
- Coordinator will retry on next review

Safe failures:
- Missing .env: use defaults
- Missing index.json: create new empty index
- Read-only filesystem: log and skip purge

## Integration with Coordinator

Coordinator will:
1. After synthesis report is created
2. Check AUTO_PURGE_ENABLED setting
3. Call this agent asynchronously
4. Receive maintenance report
5. Display brief summary to user ("Storage cleaned: 42.4 KB freed, 39 reviews retained")
6. Append maintenance report to review metadata

## Performance Considerations

- Maintenance runs after each review (quick - checks timestamps only)
- Full pattern compaction once per day (more expensive)
- Don't block coordinator waiting for maintenance
- Run as background task if possible
