# Maintenance Guide

## Table of Contents
- [Regular Maintenance Tasks](#regular-maintenance-tasks)
- [Log Management](#log-management)
- [Session Management](#session-management)
- [Database Maintenance](#database-maintenance)
- [Dependency Updates](#dependency-updates)
- [Troubleshooting](#troubleshooting)
- [Monitoring Alerts](#monitoring-alerts)
- [Backup Procedures](#backup-procedures)

## Regular Maintenance Tasks

### Daily Tasks
```bash
#!/bin/bash
# daily_maintenance.sh

# Check service status
systemctl status eth-btc-bot

# Verify logs for errors
grep -i "error\|exception\|fail" /var/log/eth-btc-bot/bot.log

# Check disk space
df -h /var/log/eth-btc-bot/

# Verify API connectivity
curl -s -o /dev/null -w "%{http_code}" https://api.coingecko.com/api/v3/ping
```

### Weekly Tasks
```bash
#!/bin/bash
# weekly_maintenance.sh

# Rotate logs
logrotate -f /etc/logrotate.d/eth-btc-bot

# Clear browser cache and sessions
rm -rf /home/bot-user/.cache/selenium/
rm -rf /tmp/chrome_profile_*

# Check for dependency updates
pip list --outdated

# Verify backups
ls -la /var/backups/eth-btc-bot/
```

### Monthly Tasks
```bash
#!/bin/bash
# monthly_maintenance.sh

# Update dependencies
pip install -r requirements.txt --upgrade

# Rotate credentials if needed
source scripts/rotate_credentials.sh

# Run full system test
python -m tests.system_test

# Clean up old backups
find /var/backups/eth-btc-bot/ -type f -mtime +90 -delete
```

### Quarterly Tasks
```bash
#!/bin/bash
# quarterly_maintenance.sh

# Security audit
source scripts/security_audit.sh

# Configuration review
python scripts/validate_config.py

# Performance analysis
python scripts/performance_analysis.py

# API usage review
python scripts/api_usage_report.py
```

## Log Management

### Log Rotation Configuration
```
# /etc/logrotate.d/eth-btc-bot
/var/log/eth-btc-bot/*.log {
    daily
    missingok
    rotate 14
    compress
    delaycompress
    notifempty
    create 0640 bot-user bot-user
    sharedscripts
    postrotate
        systemctl reload eth-btc-bot
    endscript
}
```

### Log Analysis
```bash
# Find most common errors
grep -i "error" /var/log/eth-btc-bot/bot.log | sort | uniq -c | sort -nr | head -10

# Check API call frequency
grep "API call" /var/log/eth-btc-bot/bot.log | wc -l

# Analyze tweet posting pattern
grep "Tweet posted" /var/log/eth-btc-bot/bot.log | awk '{print $1, $2}' | sort | uniq -c
```

### Log Cleanup
```bash
# Clean logs older than 90 days
find /var/log/eth-btc-bot/ -name "*.log.*" -type f -mtime +90 -delete

# Compress older logs
find /var/log/eth-btc-bot/ -name "*.log.*" -type f -mtime +7 -exec gzip {} \;
```

## Session Management

### Browser Session Cleanup
```python
def cleanup_browser_sessions():
    """Clear stale browser sessions and profiles"""
    # Find Chrome processes running for more than 2 hours
    old_processes = subprocess.check_output(
        "ps -eo pid,etimes,command | grep chrome | awk '$2 > 7200 {print $1}'",
        shell=True
    ).decode().strip().split('\n')
    
    # Terminate old processes
    for pid in old_processes:
        if pid:
            subprocess.run(['kill', '-9', pid])
    
    # Remove temporary Chrome profiles
    temp_profiles = glob.glob('/tmp/chrome_profile_*')
    for profile in temp_profiles:
        if os.path.getmtime(profile) < (time.time() - 86400):  # Older than 1 day
            shutil.rmtree(profile, ignore_errors=True)
```

### API Session Management
```python
def manage_api_sessions():
    """Reset API sessions regularly to prevent stale connections"""
    global session
    
    # Close existing session
    if session:
        try:
            session.close()
        except:
            pass
    
    # Create new session with updated settings
    session = requests.Session()
    session.timeout = (30, 90)  # (connect, read) timeouts
    
    # Add retry mechanism
    retry = Retry(
        total=5,
        backoff_factor=0.5,
        status_forcelist=[429, 500, 502, 503, 504]
    )
    adapter = HTTPAdapter(max_retries=retry)
    session.mount('http://', adapter)
    session.mount('https://', adapter)
    
    return session
```

## Database Maintenance

### Data Pruning
```python
def prune_database():
    """Remove old data to maintain database performance"""
    with db_connection() as conn:
        cursor = conn.cursor()
        
        # Remove analysis data older than 60 days
        cursor.execute("""
            DELETE FROM analysis_results 
            WHERE timestamp < DATE_SUB(NOW(), INTERVAL 60 DAY)
        """)
        
        # Remove detailed logs older than 30 days
        cursor.execute("""
            DELETE FROM api_call_logs
            WHERE timestamp < DATE_SUB(NOW(), INTERVAL 30 DAY)
        """)
        
        # Optimize tables after large deletions
        cursor.execute("OPTIMIZE TABLE analysis_results")
        cursor.execute("OPTIMIZE TABLE api_call_logs")
        
        conn.commit()
```

### Index Maintenance
```python
def maintain_indexes():
    """Rebuild and analyze database indexes"""
    with db_connection() as conn:
        cursor = conn.cursor()
        
        # Analyze table statistics
        cursor.execute("ANALYZE TABLE analysis_results")
        cursor.execute("ANALYZE TABLE api_call_logs")
        cursor.execute("ANALYZE TABLE tweet_history")
        
        # Check for fragmented indexes
        cursor.execute("""
            SELECT table_name, index_name
            FROM information_schema.statistics
            WHERE table_schema = DATABASE()
        """)
        
        # Rebuild fragmented indexes
        for table, index in cursor.fetchall():
            cursor.execute(f"ALTER TABLE {table} DROP INDEX {index}")
            cursor.execute(f"ALTER TABLE {table} ADD INDEX {index} ({index})")
```

## Dependency Updates

### Checking for Updates
```bash
# Generate list of outdated packages
pip list --outdated > outdated_packages.txt

# Check for security advisories
safety check -r requirements.txt
```

### Update Process
```bash
# Best practice update process
# 1. Create backup of current environment
pip freeze > requirements.backup.txt

# 2. Update packages
pip install -U packagename

# 3. Test functionality after updates
python -m tests.integration_test

# 4. If issues, rollback
# pip uninstall -y packagename
# pip install -r requirements.backup.txt
```

### Critical Dependencies

| Package | Purpose | Update Frequency | Risk Level |
|---------|---------|------------------|------------|
| Selenium | Browser Automation | Monthly | Medium |
| Requests | API Communication | Monthly | Low |
| Python-dotenv | Environment Config | Quarterly | Low |
| Cryptography | Security | Immediately | High |

## Troubleshooting

### Common Issues

#### Twitter Login Failures
```python
def troubleshoot_twitter_login():
    """Diagnose and fix Twitter login issues"""
    # Check for captcha presence
    try:
        captcha = browser.driver.find_element_by_id('recaptcha')
        if captcha.is_displayed():
            logger.error("CAPTCHA detected, manual intervention required")
            send_alert("CAPTCHA detected during Twitter login")
            return False
    except:
        pass
    
    # Check for account lockout
    try:
        account_locked = browser.driver.find_elements_by_xpath(
            "//*[contains(text(), 'locked') or contains(text(), 'unusual activity')]"
        )
        if account_locked:
            logger.error("Account may be locked - security verification needed")
            send_alert("Twitter account locked, verification needed")
            return False
    except:
        pass
    
    # Clear cookies and retry
    browser.driver.delete_all_cookies()
    browser.driver.refresh()
    
    return retry_login()
```

#### API Rate Limiting
```python
def handle_rate_limiting(service_name, retry_after=60):
    """Handle rate limit errors for different services"""
    logger.warning(f"{service_name} rate limit reached, pausing for {retry_after} seconds")
    
    # Record rate limit incident
    metrics.increment(f"{service_name}_rate_limits")
    
    # Update dynamic backoff timer
    global rate_limit_backoff
    rate_limit_backoff[service_name] = min(
        rate_limit_backoff.get(service_name, retry_after) * 1.5,
        3600  # Max 1 hour backoff
    )
    
    # Pause operations
    time.sleep(retry_after)
    
    # Reduce frequency temporarily
    adjust_polling_frequency(service_name, increase_interval=True)
```

#### Browser Crashes
```python
def recover_from_browser_crash():
    """Handle and recover from browser crashes"""
    logger.error("Browser crash detected")
    
    # Kill any zombie processes
    os.system("pkill -f chrome")
    os.system("pkill -f chromedriver")
    
    # Clean temporary files
    for pattern in ["/.org.chromium.Chromium*", "/.com.google.Chrome*"]:
        os.system(f"rm -rf /tmp{pattern}")
    
    # Reinitialize browser with clean profile
    browser.initialize_driver(fresh_profile=True)
    
    # Verify successful recovery
    try:
        browser.driver.get("https://www.google.com")
        if browser.driver.title:
            logger.info("Browser successfully recovered")
            return True
    except:
        logger.critical("Failed to recover browser")
        return False
```

### Diagnostic Procedures
```bash
# System diagnostic script
#!/bin/bash

echo "=== System Resources ==="
free -h
df -h
top -b -n 1 | head -20

echo "=== Network Connectivity ==="
ping -c 3 api.coingecko.com
ping -c 3 twitter.com
ping -c 3 api.anthropic.com

echo "=== Service Status ==="
systemctl status eth-btc-bot

echo "=== Recent Errors ==="
tail -50 /var/log/eth-btc-bot/bot.log | grep -i "error\|exception\|fail"

echo "=== Process List ==="
ps aux | grep -E "python|chrome|selenium"
```

## Monitoring Alerts

### Alert Thresholds
```json
{
  "alerts": {
    "high_error_rate": {
      "threshold": 10,
      "period": "10m",
      "description": "High error rate detected",
      "action": "notify_admin"
    },
    "api_failure": {
      "threshold": 5,
      "period": "5m",
      "description": "Consecutive API failures",
      "action": "restart_service"
    },
    "disk_space": {
      "threshold": 90,
      "period": "1h",
      "description": "Disk space usage above 90%",
      "action": "notify_admin"
    },
    "memory_usage": {
      "threshold": 85,
      "period": "5m",
      "description": "Memory usage above 85%",
      "action": "restart_service"
    }
  }
}
```

### Alert Response Actions
```python
def handle_alert(alert_type, alert_data):
    """Respond to system alerts"""
    logger.warning(f"Alert triggered: {alert_type}")
    
    actions = {
        "notify_admin": send_admin_notification,
        "restart_service": restart_service,
        "clean_disk_space": clean_disk_space,
        "reduce_polling": reduce_polling_frequency
    }
    
    # Get configured action
    action = ALERT_CONFIG["alerts"][alert_type]["action"]
    
    # Execute response
    if action in actions:
        actions[action](alert_type, alert_data)
    else:
        logger.error(f"Unknown alert action: {action}")
```

## Backup Procedures

### Configuration Backup
```bash
#!/bin/bash
# backup_config.sh

# Set backup directory
BACKUP_DIR="/var/backups/eth-btc-bot"
mkdir -p "$BACKUP_DIR"

# Create timestamp
TIMESTAMP=$(date +"%Y%m%d_%H%M%S")

# Backup all configuration files
tar -czf "$BACKUP_DIR/config_$TIMESTAMP.tar.gz" /opt/eth-btc-bot/config/

# Backup environment variables
cp /opt/eth-btc-bot/.env "$BACKUP_DIR/.env_$TIMESTAMP"

# Encrypt sensitive backups
gpg --encrypt --recipient admin@example.com "$BACKUP_DIR/.env_$TIMESTAMP"
rm "$BACKUP_DIR/.env_$TIMESTAMP"

# Remove backups older than 90 days
find "$BACKUP_DIR" -type f -mtime +90 -delete
```

### Data Backup
```python
def backup_database():
    """Backup application database"""
    timestamp = datetime.now().strftime("%Y%m%d_%H%M%S")
    backup_file = f"/var/backups/eth-btc-bot/db_backup_{timestamp}.sql.gz"
    
    try:
        # Create backup directory if it doesn't exist
        os.makedirs(os.path.dirname(backup_file), exist_ok=True)
        
        # Run database dump and compress
        subprocess.run([
            "mysqldump",
            "-u", db_config["user"],
            f"-p{db_config['password']}",
            db_config["database"],
            "--single-transaction",
            "--quick",
            f"--result-file={backup_file}"
        ], check=True)
        
        # Compress the backup
        subprocess.run(["gzip", "-f", backup_file], check=True)
        
        logger.info(f"Database backup created: {backup_file}.gz")
        return f"{backup_file}.gz"
    except Exception as e:
        logger.error(f"Database backup failed: {str(e)}")
        return None
```

### Restore Procedures
```bash
#!/bin/bash
# restore_backup.sh

if [ $# -ne 1 ]; then
    echo "Usage: $0 <backup_file>"
    exit 1
fi

BACKUP_FILE=$1

# Check if file exists
if [ ! -f "$BACKUP_FILE" ]; then
    echo "Backup file not found: $BACKUP_FILE"
    exit 1
fi

# Stop service
systemctl stop eth-btc-bot

# Restore configuration
if [[ "$BACKUP_FILE" == *"config"* ]]; then
    # Backup current config first
    cp -r /opt/eth-btc-bot/config/ /opt/eth-btc-bot/config.bak/
    
    # Extract backup
    tar -xzf "$BACKUP_FILE" -C /
    
    echo "Configuration restored from $BACKUP_FILE"
fi

# Restore database
if [[ "$BACKUP_FILE" == *"db_backup"* ]]; then
    # Uncompress if needed
    if [[ "$BACKUP_FILE" == *.gz ]]; then
        gunzip -c "$BACKUP_FILE" > "${BACKUP_FILE%.gz}"
        BACKUP_FILE="${BACKUP_FILE%.gz}"
    fi
    
    # Import backup
    mysql -u "$DB_USER" -p"$DB_PASSWORD" "$DB_NAME" < "$BACKUP_FILE"
    
    echo "Database restored from $BACKUP_FILE"
fi

# Restart service
systemctl start eth-btc-bot
```
