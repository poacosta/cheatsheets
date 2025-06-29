# Linux System Administration Essentials

## Quick Commands

### File & Directory Operations

```bash
# Grant all permissions to current user
sudo chown $USER:$USER -R .

# Rename directory
mv /home/user/oldname /home/user/newname

# Remove files and directories safely
rm -rf directory/  # Use with caution
find . -name "*.tmp" -delete  # Selective removal
```

### Permission Management

```bash
# Common permission patterns
chmod 755 script.sh     # Executable script
chmod 644 config.file   # Read-write for owner, read for others
chmod 600 private.key   # Private file, owner only

# Recursive permission fix
find . -type d -exec chmod 755 {} \;  # Directories
find . -type f -exec chmod 644 {} \;  # Files
```

### Environment Variables

```bash
# Set temporary variable
export API_KEY="your-key-here"

# Persistent environment (add to ~/.bashrc or ~/.zshrc)
echo 'export PATH="$PATH:/opt/custom/bin"' >> ~/.bashrc

# List all environment variables
env | sort

# Check specific variable
echo $PATH
printenv HOME
```

## Common Patterns

### Safe File Operations

```bash
# Always backup before major changes
cp important-file.conf important-file.conf.backup.$(date +%Y%m%d)

# Use rsync for directory synchronization
rsync -avz --progress source/ destination/

# Find and replace across files
grep -r "old-string" . --include="*.txt"
find . -name "*.txt" -exec sed -i 's/old-string/new-string/g' {} \;
```

### Process Management

```bash
# Find processes by name
ps aux | grep process-name
pgrep -f process-name

# Kill processes gracefully
pkill -TERM process-name
# Force kill if needed
pkill -KILL process-name

# Monitor system resources
htop
iotop  # I/O monitoring
nethogs  # Network usage by process
```

### Directory Navigation & Search

```bash
# Advanced find operations
find . -type f -name "*.log" -mtime +7 -delete  # Remove old logs
find . -type f -size +100M  # Find large files
find . -type f -perm 777  # Find world-writable files

# Disk usage analysis
du -sh */  # Size of directories
df -h      # Filesystem usage
ncdu       # Interactive disk usage analyzer
```

## Personal Gotchas

### Ownership & Permissions

- Use `$USER:$USER` for current user ownership
- Always test permission changes on non-critical files first
- Remember: directories need execute permission to be traversed

### Path Management

```bash
# Check if command exists
command -v docker >/dev/null 2>&1 || echo "Docker not installed"

# Add to PATH safely
[[ ":$PATH:" != *":/opt/custom/bin:"* ]] && export PATH="/opt/custom/bin:$PATH"
```

### File Removal Safety

- Use `ls` first to verify what will be affected
- Consider `trash-cli` instead of `rm` for important files
- Always double-check wildcards before executing

## Performance Notes

### Efficient Commands

```bash
# Use find instead of ls for complex searches
find . -name "*.log" -type f  # Better than ls -la | grep .log

# Parallel operations
find . -name "*.txt" -print0 | xargs -0 -P 4 grep "pattern"  # 4 parallel processes

# Memory-efficient file processing
while IFS= read -r line; do
  # process line
done < large-file.txt  # Better than loading entire file into memory
```

### System Monitoring

```bash
# Check system load
uptime
w

# Memory usage
free -h
cat /proc/meminfo

# CPU information
lscpu
cat /proc/cpuinfo
```

## Architecture Considerations

### Security Best Practices

```bash
# Secure file permissions for sensitive data
chmod 600 ~/.ssh/id_rsa
chmod 700 ~/.ssh

# Check for world-writable files (security risk)
find / -type f -perm -002 2>/dev/null

# Monitor system integrity
sudo find /etc -name "*.conf" -type f -exec ls -la {} \;
```

### System Maintenance

```bash
# Log rotation and cleanup
sudo logrotate -f /etc/logrotate.conf

# Package management (Ubuntu/Debian)
sudo apt update && sudo apt upgrade
sudo apt autoremove
sudo apt autoclean

# Service management
sudo systemctl status service-name
sudo systemctl enable service-name
sudo systemctl disable service-name
```

## Advanced Patterns

### Scripting Helpers

```bash
# Error handling in scripts
set -euo pipefail  # Exit on error, undefined vars, pipe failures

# Logging with timestamps
exec > >(while read line; do echo "$(date '+%Y-%m-%d %H:%M:%S') $line"; done)

# Lock files to prevent concurrent execution
(
  flock -n 9 || exit 1
  # Critical section here
) 9>/var/lock/script.lock
```

## Context Links

- [Linux File Permissions Guide](https://linuxize.com/post/understanding-linux-file-permissions/)
- [Environment Variables in Linux](https://linuxize.com/post/how-to-set-and-list-environment-variables-in-linux/)
- [File Removal Best Practices](https://linuxize.com/post/how-to-remove-files-and-directories-using-linux-command-line/)
