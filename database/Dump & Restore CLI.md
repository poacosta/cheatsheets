# PostgreSQL Dump & Restore CLI Cheatsheet

## Quick Commands

### Basic Dump Operations

```bash
# Full database dump
pg_dump dbname > backup.sql
pg_dump -U username -h hostname dbname > backup.sql

# Compressed dump (custom format)
pg_dump -Fc dbname > backup.dump
pg_dump -Fc -U username -h hostname dbname > backup.dump

# Directory format (parallel processing)
pg_dump -Fd dbname -f backup_dir

# Gzip compressed SQL
pg_dump dbname | gzip > backup.sql.gz
```

### Basic Restore Operations

```bash
# From SQL dump
psql dbname < backup.sql
psql -U username -h hostname dbname < backup.sql

# From custom format
pg_restore -d dbname backup.dump
pg_restore -U username -h hostname -d dbname backup.dump

# From gzipped SQL
gunzip -c backup.sql.gz | psql dbname
```

## Advanced Dump Patterns

### Schema & Data Control

```bash
# Schema only (structure, no data)
pg_dump --schema-only dbname > schema.sql

# Data only (no structure)
pg_dump --data-only dbname > data.sql

# Exclude specific tables
pg_dump --exclude-table=logs --exclude-table=temp_* dbname > backup.sql

# Include only specific tables
pg_dump --table=users --table=orders dbname > partial.sql

# Exclude data from specific tables (structure only for those)
pg_dump --exclude-table-data=logs --exclude-table-data=audit_* dbname > backup.sql
```

### Network & Security

```bash
# With SSL
pg_dump "postgresql://user:password@host:5432/dbname?sslmode=require" > backup.sql

# With connection string
pg_dump "host=localhost port=5432 dbname=mydb user=myuser password=mypass" > backup.sql

# Prompt for password
pg_dump -W -U username dbname > backup.sql

# Using .pgpass file (recommended for automation)
# Create ~/.pgpass with: hostname:port:database:username:password
chmod 600 ~/.pgpass
pg_dump -U username -h hostname dbname > backup.sql
```

## Advanced Restore Patterns

### Performance & Control

```bash
# Parallel restore (custom format only)
pg_restore -j 4 -d dbname backup.dump

# Verbose output
pg_restore -v -d dbname backup.dump

# Clean database before restore (drop existing objects)
pg_restore --clean -d dbname backup.dump

# Create database and restore
pg_restore --create -d postgres backup.dump

# Ignore errors and continue
pg_restore --if-exists --clean -d dbname backup.dump
```

### Selective Restore

```bash
# List contents of dump file
pg_restore --list backup.dump

# Restore only specific tables
pg_restore -t users -t orders -d dbname backup.dump

# Restore only data (no schema)
pg_restore --data-only -d dbname backup.dump

# Restore only schema (no data)
pg_restore --schema-only -d dbname backup.dump

# Exclude specific schemas
pg_restore --exclude-schema=temp_schema -d dbname backup.dump
```

## Production Patterns I Actually Use

### Zero-Downtime Backup Strategy

```bash
#!/bin/bash
# Production backup script
DATE=$(date +%Y%m%d_%H%M%S)
DB_NAME="production_db"
BACKUP_DIR="/backups/postgres"

# Custom format with compression
pg_dump -Fc -v \
  --exclude-table-data=logs \
  --exclude-table-data=analytics_raw \
  -h production-host \
  -U backup_user \
  $DB_NAME > "$BACKUP_DIR/${DB_NAME}_${DATE}.dump"

# Verify backup integrity
pg_restore --list "$BACKUP_DIR/${DB_NAME}_${DATE}.dump" > /dev/null
if [ $? -eq 0 ]; then
    echo "Backup successful: ${DB_NAME}_${DATE}.dump"
    # Clean old backups (keep last 7 days)
    find $BACKUP_DIR -name "${DB_NAME}_*.dump" -mtime +7 -delete
else
    echo "Backup verification failed!"
    exit 1
fi
```

### Staging Environment Refresh

```bash
#!/bin/bash
# Refresh staging with production data
PROD_DUMP="production_$(date +%Y%m%d).dump"
STAGING_DB="staging_db"

# Create fresh dump from production
pg_dump -Fc production_db > $PROD_DUMP

# Drop and recreate staging database
dropdb $STAGING_DB
createdb $STAGING_DB

# Restore with parallel processing
pg_restore -j 4 -v -d $STAGING_DB $PROD_DUMP

# Run staging-specific transformations
psql $STAGING_DB -c "
  UPDATE users SET email = concat('test+', id, '@company.com');
  TRUNCATE TABLE audit_logs;
  UPDATE config SET value = 'staging' WHERE key = 'environment';
"

echo "Staging refresh complete"
```

## Personal Gotchas & Solutions

### Connection Issues

```bash
# When pg_dump hangs (common with large DBs)
# Use statement timeout
pg_dump --statement-timeout=300000 dbname > backup.sql

# For unreliable networks
pg_dump --lock-wait-timeout=10000 dbname > backup.sql
```

### Large Database Handling

```bash
# For databases > 1GB, always use custom format
pg_dump -Fc --compress=9 large_db > backup.dump

# For massive databases, use directory format for parallel processing
pg_dump -Fd -j 4 massive_db -f backup_directory/
```

### Character Encoding Issues

```bash
# Specify encoding explicitly
pg_dump --encoding=UTF8 dbname > backup.sql

# For Windows compatibility
pg_dump --encoding=WIN1252 dbname > backup.sql
```

## Environment Variables

```bash
# Set these to avoid repetitive flags
export PGHOST=localhost
export PGPORT=5432
export PGUSER=myuser
export PGDATABASE=mydb
export PGPASSWORD=mypassword  # Not recommended, use .pgpass instead

# Then simply run
pg_dump > backup.sql
pg_restore -d dbname backup.dump
```

## Format Comparison

| Format    | Extension | Pros                                    | Cons                            | Use Case                    |
| --------- | --------- | --------------------------------------- | ------------------------------- | --------------------------- |
| SQL       | `.sql`    | Human readable, universal               | Large size, no parallel restore | Development, simple backups |
| Custom    | `.dump`   | Compressed, selective restore, parallel | Binary format                   | Production backups          |
| Directory | `dir/`    | Parallel dump/restore, flexible         | Complex structure               | Large databases             |
| TAR       | `.tar`    | Portable, compressed                    | No parallel restore             | Archive storage             |

## Performance Notes

### When Size Matters

- Custom format with compression (`-Fc`) typically 60-80% smaller than SQL
- Directory format allows parallel processing but uses more disk space during operation
- Exclude large log/analytics tables from regular backups

### When Speed Matters

- Parallel operations (`-j`) scale well up to CPU core count
- Network latency affects dump speed more than restore speed
- Local dumps are 3-5x faster than remote dumps

## Security Checklist

```bash
# Secure backup practices
chmod 600 backup.dump                    # Restrict file permissions
pg_dump --no-password ...                # Never pass password in command line
export PGPASSFILE=/secure/path/.pgpass    # Use password file
pg_dump --exclude-table=sensitive_data   # Exclude sensitive tables
```

## Quick Troubleshooting

```bash
# Check dump file integrity
pg_restore --list backup.dump | head -5

# Estimate restore time
pg_restore --list backup.dump | wc -l    # Rough object count

# Test restore without executing
pg_restore --schema-only --dry-run backup.dump

# Monitor long-running restore
pg_restore -v backup.dump 2>&1 | tee restore.log
```
