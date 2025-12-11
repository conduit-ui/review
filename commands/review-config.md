# Review System Configuration

Configure review system settings including retention period, auto-purging, and storage paths.

## Usage

```bash
/review-config                                      # Interactive configuration wizard
/review-config view                                 # Show current settings
/review-config set REVIEW_RETENTION_DAYS 45        # Update specific setting
/review-config reset                                # Restore defaults
/review-config show-schedule                        # Show next maintenance schedule
```

## What This Command Does

1. **Interactive Mode** (`/review-config`)
   - Shows current settings
   - Prompts to modify each setting
   - Validates input ranges
   - Saves to `.env`
   - Shows next purge date

2. **View Mode** (`/review-config view`)
   - Displays all current settings
   - Shows last maintenance run
   - Shows next scheduled maintenance
   - Shows storage usage

3. **Set Mode** (`/review-config set KEY VALUE`)
   - Updates single setting
   - Validates value
   - Saves immediately
   - Shows confirmation

4. **Reset Mode** (`/configure reset`)
   - Restores all defaults
   - Confirms before resetting
   - Backs up previous `.env`

## Settings

### REVIEW_RETENTION_DAYS
- **Default**: 30
- **Range**: 1-365
- **Description**: How many days to keep review reports
- **Impact**: Reviews older than this are automatically deleted

Example:
```
/configure set REVIEW_RETENTION_DAYS 45
# Reviews will be kept for 45 days before auto-delete
```

### AUTO_PURGE_ENABLED
- **Default**: true
- **Options**: true, false
- **Description**: Automatically delete old reviews based on retention policy
- **Impact**: Set to false if you have external backup storage

Example:
```
/configure set AUTO_PURGE_ENABLED false
# Reviews will NOT be automatically deleted
```

### PATTERN_INDEX_PATH
- **Default**: ./.claude/reviews/patterns
- **Description**: Where pattern index is stored
- **Impact**: All pattern discoveries saved to this location

Example:
```
/configure set PATTERN_INDEX_PATH /shared/patterns
# Store patterns in shared location for team access
```

### LOG_LEVEL
- **Default**: info
- **Options**: debug, info, warning, error
- **Description**: Detail level of maintenance logs

Example:
```
/configure set LOG_LEVEL debug
# More detailed logging for troubleshooting
```

## Example Usage

### First Time Setup

```
$ /configure

Review System Configuration Wizard

No .env file found. Creating with defaults...

Current Settings:
├── Retention Period: 30 days
├── Auto Purge: Enabled
├── Pattern Index: ./.claude/reviews/patterns
└── Log Level: info

Modify settings? (Enter to accept defaults):

REVIEW_RETENTION_DAYS [30]: 45
AUTO_PURGE_ENABLED [true]:
PATTERN_INDEX_PATH [./.claude/reviews/patterns]:
LOG_LEVEL [info]: debug

Configuration saved to .env

Next maintenance scheduled: 2025-01-09
Storage path: ./.claude/reviews
Index path: ./.claude/reviews/patterns
```

### View Current Settings

```
$ /configure view

Review System Configuration

Settings:
├── Retention Period: 45 days
├── Auto Purge: Enabled
├── Pattern Index: ./.claude/reviews/patterns
└── Log Level: debug

Storage Status:
├── Reviews stored: 39
├── Total size: 1.2 MB
├── Oldest review: 2025-11-10
└── Next purge: 2025-01-09

Pattern Index:
├── Total patterns: 8
├── Security risks: 5
├── N+1 queries: 3
├── Last updated: 2025-12-10 19:30
└── Reviews analyzed: 43
```

### Update Single Setting

```
$ /configure set REVIEW_RETENTION_DAYS 60

Updating REVIEW_RETENTION_DAYS from 45 to 60...
Validating: 60 is in range 1-365 ✓

Configuration updated!

New next purge date: 2025-01-09
```

### Show Maintenance Schedule

```
$ /configure show-schedule

Maintenance Schedule

Last Run: 2025-12-10 19:30:00
Status: Success
├── Reviews purged: 3
├── Storage freed: 42.4 KB
└── Patterns updated: 8

Next Run: After next review completion
Current Retention: 45 days

Reviews will be purged when older than: 45 days
Next review to trigger purge: When oldest review exceeds 45 days
Estimated next purge: 2025-01-09
```

## Configuration File

Settings are stored in `.env` file:

```bash
# .env (created by /configure)
REVIEW_RETENTION_DAYS=45
REVIEW_STORAGE_PATH=./.claude/reviews
PATTERN_INDEX_PATH=./.claude/reviews/patterns
AUTO_PURGE_ENABLED=true
LOG_LEVEL=debug
```

Back up of previous config: `.env.backup`

## Defaults

If `.env` is missing or corrupted, defaults are:

```bash
REVIEW_RETENTION_DAYS=30
REVIEW_STORAGE_PATH=./.claude/reviews
PATTERN_INDEX_PATH=./.claude/reviews/patterns
AUTO_PURGE_ENABLED=true
LOG_LEVEL=info
```

## Tips

- **Development**: Set `REVIEW_RETENTION_DAYS=7` for faster cleanup during testing
- **Production**: Keep `REVIEW_RETENTION_DAYS=30` or higher for audit trail
- **Team Shared**: Set `PATTERN_INDEX_PATH=/shared/patterns` to share learnings
- **Debugging**: Set `LOG_LEVEL=debug` when troubleshooting maintenance issues

## Integration with Coordinator

The coordinator automatically:
- Reads your current settings
- Uses retention period for purging
- Respects AUTO_PURGE_ENABLED flag
- Updates pattern index at configured location

Changes take effect on next review run.
