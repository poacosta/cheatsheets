# Rails Development Troubleshooting Cheatsheet

## Server Issues

### PID File Problems
```bash
# Quick fixes
rm tmp/pids/server.pid
kill -9 $(lsof -ti :3000)

# Check running processes
ps aux | grep rails
lsof -i :3000

# Cleanup script
#!/bin/bash
PID_FILE="tmp/pids/server.pid"
if [ -f "$PID_FILE" ]; then
    PID=$(cat "$PID_FILE")
    [ -n "$PID" ] && kill -TERM $PID 2>/dev/null
    rm "$PID_FILE"
fi
```

### Port Conflicts
```bash
# Find process using port
lsof -i :3000
netstat -tulpn | grep :3000

# Kill process on port
kill -9 $(lsof -ti :3000)

# Start on different port
rails s -p 3001
```

### Memory Issues
```bash
# Check Rails memory usage
ps aux | grep rails | awk '{print $6/1024 " MB"}'

# Monitor memory during development
watch -n 1 'ps aux | grep rails'

# Restart with memory profiling
RUBY_GC_MALLOC_LIMIT=90000000 rails s
```

## Database Issues

### Migration Problems
```bash
# Check migration status
rails db:migrate:status

# Rollback specific migration
rails db:migrate:down VERSION=20231201000000

# Reset database (development only)
rails db:drop db:create db:migrate db:seed

# Fix schema conflicts
rails db:schema:load
rails db:migrate:reset
```

### Connection Issues
```bash
# Test database connection
rails db:version
rails runner "puts ActiveRecord::Base.connection.active?"

# Check database configuration
rails runner "puts ActiveRecord::Base.configurations[Rails.env].inspect"

# Manual connection test
psql -h localhost -U username -d database_name
```

### Lock Issues
```bash
# PostgreSQL: Check for locks
SELECT * FROM pg_locks WHERE NOT granted;

# Kill blocking queries
SELECT pg_terminate_backend(pid) FROM pg_stat_activity WHERE state = 'active';

# MySQL: Show process list
SHOW PROCESSLIST;
KILL <process_id>;
```

## Asset Pipeline

### Precompilation Issues
```bash
# Clear assets
rails assets:clobber
rm -rf public/assets tmp/cache/assets

# Precompile with debugging
RAILS_ENV=production rails assets:precompile --trace

# Check asset paths
rails runner "puts Rails.application.config.assets.paths"
```

### Webpack/Webpacker Issues
```bash
# Clear webpack cache
rm -rf node_modules/.cache
yarn install

# Rebuild webpack
rails webpacker:compile
rails webpacker:clobber

# Debug webpack
./bin/webpack-dev-server --debug
```

## Gem Issues

### Bundle Problems
```bash
# Clean bundle
bundle clean --force
rm -rf .bundle vendor/cache
bundle install

# Check gem conflicts
bundle check
bundle outdated

# Platform specific
bundle lock --add-platform x86_64-linux
bundle install --local
```

### Version Conflicts
```bash
# Check Ruby version
ruby -v
cat .ruby-version

# RVM/rbenv version management
rvm use 3.1.0
rbenv local 3.1.0

# Gemfile.lock issues
rm Gemfile.lock
bundle install
```

## Performance Debugging

### Slow Queries
```ruby
# In development.rb
config.active_record.verbose_query_logs = true

# Query analysis
ActiveRecord::Base.logger = Logger.new(STDOUT)

# Bullet gem configuration
config.after_initialize do
  Bullet.enable = true
  Bullet.bullet_logger = true
  Bullet.console = true
end
```

### Memory Profiling
```ruby
# Add to Gemfile
gem 'memory_profiler'
gem 'derailed_benchmarks'

# Usage
MemoryProfiler.report do
  # Your code here
end.pretty_print
```

### Request Profiling
```bash
# Install rack-mini-profiler
gem 'rack-mini-profiler'

# Usage: Add ?pp=help to any URL
# Or programmatically
Rack::MiniProfiler.authorize_request
```

## Testing Issues

### RSpec Problems
```bash
# Clear test cache
rm -rf tmp/cache/

# Database cleanup
RAILS_ENV=test rails db:drop db:create db:migrate

# Run specific tests
rspec spec/models/user_spec.rb:25
rspec --tag focus
rspec --fail-fast
```

### Factory Issues
```ruby
# Debug factories
FactoryBot.lint
FactoryBot.build(:user).valid?

# Check factory definitions
FactoryBot.factories.map(&:name)

# Sequences debugging
FactoryBot.sequences.each { |s| puts "#{s.name}: #{s.value}" }
```

## Development Environment

### Rails Console Issues
```bash
# Console with specific environment
rails c production
rails c -e staging

# Debug console startup
rails c --verbose

# Sandbox mode (rollback changes)
rails c --sandbox
```

### Log Analysis
```bash
# Tail logs
tail -f log/development.log
tail -f log/production.log | grep ERROR

# Log rotation
logrotate -f /etc/logrotate.d/rails

# Search logs
grep -n "ERROR" log/development.log
grep -A 5 -B 5 "specific_error" log/production.log
```

### Cache Issues
```bash
# Clear all Rails caches
rails tmp:clear
rails dev:cache  # Toggle development caching

# Specific cache clearing
Rails.cache.clear
ActionController::Base.expire_page("/path")

# Redis cache (if using)
redis-cli FLUSHALL
```

## Production Deployment

### Precompilation
```bash
# Assets
RAILS_ENV=production rails assets:precompile

# Database
RAILS_ENV=production rails db:migrate

# Check production console
RAILS_ENV=production rails c
```

### Environment Variables
```bash
# Check environment
rails runner "puts Rails.env"
rails runner "puts ENV['DATABASE_URL']"

# Dotenv debugging
rails runner "puts Dotenv.load"
```

### SSL/Security Issues
```bash
# Force SSL in production
config.force_ssl = true

# Check security headers
curl -I https://yourapp.com
```

## Docker Specific

### Container Issues
```bash
# Build with no cache
docker build --no-cache -t your-app .

# Debug container
docker run -it your-app /bin/bash
docker exec -it container_id /bin/bash

# Check logs
docker logs container_id
docker-compose logs -f web
```

### Volume Issues
```bash
# Clear volumes
docker-compose down -v
docker volume prune

# Check mounted volumes
docker inspect container_id | grep Mounts -A 20
```

## Common Error Patterns

### LoadError / NameError
```ruby
# Autoloading issues in development
config.eager_load = false
config.cache_classes = false

# Check autoload paths
puts ActiveSupport::Dependencies.autoload_paths

# Zeitwerk debugging
Rails.autoloaders.log!
```

### Database Connection Pool
```ruby
# Pool configuration
# config/database.yml
pool: <%= ENV.fetch("RAILS_MAX_THREADS") { 5 } %>
timeout: 5000

# Monitor connections
ActiveRecord::Base.connection_pool.stat
```

### Background Jobs
```bash
# Sidekiq
sidekiq -e development -v

# Check Redis
redis-cli
KEYS sidekiq:*

# Delayed Job
rails jobs:work
rails runner "Delayed::Job.delete_all"
```

## Quick Diagnostics

### Health Check Script
```bash
#!/bin/bash
echo "=== Rails Health Check ==="
echo "Ruby: $(ruby -v)"
echo "Rails: $(rails -v)"
echo "Bundle: $(bundle check && echo 'OK' || echo 'FAILED')"
echo "DB: $(rails runner 'puts ActiveRecord::Base.connection.active?' 2>/dev/null || echo 'FAILED')"
echo "Assets: $(ls public/assets/ 2>/dev/null | wc -l) files"
echo "PID: $([ -f tmp/pids/server.pid ] && echo 'EXISTS' || echo 'NONE')"
```

### System Information
```ruby
# Rails console diagnostics
puts "Rails: #{Rails.version}"
puts "Ruby: #{RUBY_VERSION}"
puts "Environment: #{Rails.env}"
puts "Database: #{ActiveRecord::Base.connection.adapter_name}"
puts "Cache: #{Rails.cache.class}"
puts "Jobs: #{Rails.application.config.active_job.queue_adapter}"
```

---

## Emergency Commands

```bash
# Nuclear option (development only)
git clean -fdx
rm -rf node_modules .bundle
bundle install
yarn install
rails db:drop db:create db:migrate db:seed
rails assets:precompile
```

**Remember**: Always backup your database before running destructive commands in production!