# Scripting & Automation

## Why Scripting?

- **Automation**: Automate repetitive tasks
- **Efficiency**: Save time and reduce errors
- **Consistency**: Ensure tasks are performed the same way
- **Scalability**: Handle multiple servers/tasks
- **Documentation**: Scripts serve as documentation

## Bash Scripting

### Basic Syntax

```bash
#!/bin/bash
# This is a comment

# Variables
NAME="DevOps"
COUNT=5

# Echo
echo "Hello, $NAME"
echo "Count: $COUNT"

# Command substitution
CURRENT_DATE=$(date)
FILES=$(ls -l)

# Arithmetic
SUM=$((5 + 3))
RESULT=$((COUNT * 2))
```

### Control Structures

```bash
#!/bin/bash

# If-else
if [ "$1" == "start" ]; then
    echo "Starting service..."
elif [ "$1" == "stop" ]; then
    echo "Stopping service..."
else
    echo "Usage: $0 {start|stop}"
fi

# Test conditions
if [ -f "file.txt" ]; then
    echo "File exists"
fi

if [ $COUNT -gt 10 ]; then
    echo "Count is greater than 10"
fi

# For loop
for i in {1..5}; do
    echo "Number: $i"
done

for file in *.txt; do
    echo "Processing: $file"
done

# While loop
counter=0
while [ $counter -lt 5 ]; do
    echo "Counter: $counter"
    counter=$((counter + 1))
done

# Case statement
case "$1" in
    start)
        echo "Starting..."
        ;;
    stop)
        echo "Stopping..."
        ;;
    restart)
        echo "Restarting..."
        ;;
    *)
        echo "Usage: $0 {start|stop|restart}"
        exit 1
        ;;
esac
```

### Functions

```bash
#!/bin/bash

# Function definition
greet() {
    echo "Hello, $1"
}

# Function with return value
add() {
    local result=$(($1 + $2))
    echo $result
}

# Usage
greet "World"
sum=$(add 5 3)
echo "Sum: $sum"
```

### Practical Examples

**Backup Script**
```bash
#!/bin/bash

# Backup script
BACKUP_DIR="/backup"
SOURCE_DIR="/var/www"
DATE=$(date +%Y%m%d_%H%M%S)
BACKUP_FILE="backup_${DATE}.tar.gz"

# Create backup directory if not exists
mkdir -p "$BACKUP_DIR"

# Create backup
echo "Creating backup..."
tar -czf "${BACKUP_DIR}/${BACKUP_FILE}" "$SOURCE_DIR"

# Check if backup was successful
if [ $? -eq 0 ]; then
    echo "Backup created successfully: ${BACKUP_FILE}"
else
    echo "Backup failed!" >&2
    exit 1
fi

# Delete backups older than 7 days
find "$BACKUP_DIR" -name "backup_*.tar.gz" -mtime +7 -delete
echo "Old backups cleaned up"
```

**Log Rotation Script**
```bash
#!/bin/bash

LOG_DIR="/var/log/myapp"
MAX_SIZE=100M
ARCHIVE_DIR="${LOG_DIR}/archive"

mkdir -p "$ARCHIVE_DIR"

for logfile in ${LOG_DIR}/*.log; do
    if [ -f "$logfile" ]; then
        size=$(stat -f%z "$logfile" 2>/dev/null || stat -c%s "$logfile")
        max_bytes=$((100 * 1024 * 1024))  # 100MB
        
        if [ $size -gt $max_bytes ]; then
            timestamp=$(date +%Y%m%d_%H%M%S)
            filename=$(basename "$logfile")
            
            # Compress and move
            gzip -c "$logfile" > "${ARCHIVE_DIR}/${filename%.log}_${timestamp}.log.gz"
            
            # Truncate original
            > "$logfile"
            
            echo "Rotated: $filename"
        fi
    fi
done
```

**System Monitoring Script**
```bash
#!/bin/bash

# System monitoring script
LOG_FILE="/var/log/system_monitor.log"
ALERT_EMAIL="admin@example.com"

# CPU usage
CPU_USAGE=$(top -bn1 | grep "Cpu(s)" | awk '{print $2}' | cut -d'%' -f1)
CPU_THRESHOLD=80

# Memory usage
MEMORY_USAGE=$(free | grep Mem | awk '{printf("%.2f", $3/$2 * 100)}')
MEMORY_THRESHOLD=80

# Disk usage
DISK_USAGE=$(df -h / | tail -1 | awk '{print $5}' | cut -d'%' -f1)
DISK_THRESHOLD=80

# Log current status
echo "$(date): CPU=${CPU_USAGE}%, MEM=${MEMORY_USAGE}%, DISK=${DISK_USAGE}%" >> "$LOG_FILE"

# Check thresholds and alert
if (( $(echo "$CPU_USAGE > $CPU_THRESHOLD" | bc -l) )); then
    echo "High CPU usage: ${CPU_USAGE}%" | mail -s "CPU Alert" "$ALERT_EMAIL"
fi

if (( $(echo "$MEMORY_USAGE > $MEMORY_THRESHOLD" | bc -l) )); then
    echo "High memory usage: ${MEMORY_USAGE}%" | mail -s "Memory Alert" "$ALERT_EMAIL"
fi

if [ $DISK_USAGE -gt $DISK_THRESHOLD ]; then
    echo "High disk usage: ${DISK_USAGE}%" | mail -s "Disk Alert" "$ALERT_EMAIL"
fi
```

**Deployment Script**
```bash
#!/bin/bash

set -e  # Exit on error

APP_NAME="myapp"
APP_DIR="/var/www/${APP_NAME}"
REPO_URL="https://github.com/user/myapp.git"
BRANCH="main"

echo "Deploying ${APP_NAME}..."

# Backup current version
if [ -d "$APP_DIR" ]; then
    BACKUP_DIR="/backup/${APP_NAME}_$(date +%Y%m%d_%H%M%S)"
    cp -r "$APP_DIR" "$BACKUP_DIR"
    echo "Backup created: $BACKUP_DIR"
fi

# Clone/pull repository
if [ -d "$APP_DIR/.git" ]; then
    cd "$APP_DIR"
    git pull origin "$BRANCH"
else
    git clone -b "$BRANCH" "$REPO_URL" "$APP_DIR"
    cd "$APP_DIR"
fi

# Install dependencies
if [ -f "package.json" ]; then
    npm ci --production
fi

# Build application
if [ -f "package.json" ] && grep -q "\"build\"" package.json; then
    npm run build
fi

# Restart service
systemctl restart "$APP_NAME"

# Health check
sleep 5
if curl -f http://localhost:3000/health > /dev/null 2>&1; then
    echo "Deployment successful!"
else
    echo "Health check failed! Rolling back..."
    systemctl stop "$APP_NAME"
    rm -rf "$APP_DIR"
    cp -r "$BACKUP_DIR" "$APP_DIR"
    systemctl start "$APP_NAME"
    exit 1
fi
```

## Python Scripting

### Basic Operations

```python
#!/usr/bin/env python3
import os
import sys
import subprocess
import json
import requests
from datetime import datetime

# File operations
def read_file(filename):
    with open(filename, 'r') as f:
        return f.read()

def write_file(filename, content):
    with open(filename, 'w') as f:
        f.write(content)

# Command execution
def run_command(command):
    result = subprocess.run(
        command,
        shell=True,
        capture_output=True,
        text=True
    )
    return result.stdout, result.stderr, result.returncode

# Example usage
stdout, stderr, code = run_command('ls -l')
if code == 0:
    print(stdout)
else:
    print(f"Error: {stderr}", file=sys.stderr)
```

### Practical Examples

**Server Health Check**
```python
#!/usr/bin/env python3
import requests
import smtplib
from email.mime.text import MIMEText
import time

ENDPOINTS = [
    'https://api.example.com/health',
    'https://web.example.com/health',
]

ALERT_EMAIL = 'admin@example.com'
SMTP_SERVER = 'smtp.gmail.com'
SMTP_PORT = 587

def check_endpoint(url):
    try:
        response = requests.get(url, timeout=10)
        return response.status_code == 200
    except Exception as e:
        print(f"Error checking {url}: {e}")
        return False

def send_alert(message):
    msg = MIMEText(message)
    msg['Subject'] = 'Server Health Alert'
    msg['From'] = ALERT_EMAIL
    msg['To'] = ALERT_EMAIL
    
    with smtplib.SMTP(SMTP_SERVER, SMTP_PORT) as server:
        server.starttls()
        server.login(ALERT_EMAIL, 'password')
        server.send_message(msg)

def main():
    for endpoint in ENDPOINTS:
        if not check_endpoint(endpoint):
            message = f"Endpoint {endpoint} is down!"
            print(message)
            send_alert(message)
        else:
            print(f"Endpoint {endpoint} is up")

if __name__ == '__main__':
    main()
```

**Log Analyzer**
```python
#!/usr/bin/env python3
import re
from collections import Counter, defaultdict
from datetime import datetime

def analyze_nginx_log(log_file):
    ip_counter = Counter()
    status_counter = Counter()
    url_counter = Counter()
    errors = []
    
    # Nginx log pattern
    pattern = r'(\S+) - - \[(.*?)\] "(.*?)" (\d+) (\d+) "(.*?)" "(.*?)"'
    
    with open(log_file, 'r') as f:
        for line in f:
            match = re.match(pattern, line)
            if match:
                ip, timestamp, request, status, size, referer, user_agent = match.groups()
                
                ip_counter[ip] += 1
                status_counter[status] += 1
                
                # Extract URL from request
                if request:
                    parts = request.split()
                    if len(parts) >= 2:
                        url_counter[parts[1]] += 1
                
                # Collect errors (4xx, 5xx)
                if status.startswith('4') or status.startswith('5'):
                    errors.append((timestamp, ip, request, status))
    
    # Print results
    print("\n=== Top 10 IP Addresses ===")
    for ip, count in ip_counter.most_common(10):
        print(f"{ip}: {count}")
    
    print("\n=== Status Code Distribution ===")
    for status, count in status_counter.most_common():
        print(f"{status}: {count}")
    
    print("\n=== Top 10 URLs ===")
    for url, count in url_counter.most_common(10):
        print(f"{url}: {count}")
    
    print("\n=== Recent Errors ===")
    for timestamp, ip, request, status in errors[-10:]:
        print(f"[{timestamp}] {ip} - {request} - {status}")

if __name__ == '__main__':
    analyze_nginx_log('/var/log/nginx/access.log')
```

**AWS Resource Manager**
```python
#!/usr/bin/env python3
import boto3
from datetime import datetime, timedelta

def list_unused_volumes():
    """Find unattached EBS volumes"""
    ec2 = boto3.client('ec2')
    volumes = ec2.describe_volumes(
        Filters=[{'Name': 'status', 'Values': ['available']}]
    )
    
    print("Unattached EBS Volumes:")
    for volume in volumes['Volumes']:
        size = volume['Size']
        vol_id = volume['VolumeId']
        created = volume['CreateTime']
        print(f"- {vol_id} ({size}GB) - Created: {created}")

def list_old_snapshots(days=30):
    """Find snapshots older than specified days"""
    ec2 = boto3.client('ec2')
    cutoff_date = datetime.now() - timedelta(days=days)
    
    snapshots = ec2.describe_snapshots(OwnerIds=['self'])
    
    print(f"\nSnapshots older than {days} days:")
    for snapshot in snapshots['Snapshots']:
        if snapshot['StartTime'].replace(tzinfo=None) < cutoff_date:
            snap_id = snapshot['SnapshotId']
            started = snapshot['StartTime']
            print(f"- {snap_id} - Created: {started}")

def list_stopped_instances():
    """Find stopped EC2 instances"""
    ec2 = boto3.client('ec2')
    instances = ec2.describe_instances(
        Filters=[{'Name': 'instance-state-name', 'Values': ['stopped']}]
    )
    
    print("\nStopped EC2 Instances:")
    for reservation in instances['Reservations']:
        for instance in reservation['Instances']:
            instance_id = instance['InstanceId']
            instance_type = instance['InstanceType']
            name = next(
                (tag['Value'] for tag in instance.get('Tags', []) if tag['Key'] == 'Name'),
                'N/A'
            )
            print(f"- {instance_id} ({instance_type}) - Name: {name}")

if __name__ == '__main__':
    list_unused_volumes()
    list_old_snapshots(30)
    list_stopped_instances()
```

**Database Backup Script**
```python
#!/usr/bin/env python3
import subprocess
import os
from datetime import datetime
import boto3

DB_HOST = 'localhost'
DB_USER = 'root'
DB_PASSWORD = 'password'
DB_NAME = 'myapp'
BACKUP_DIR = '/backup/mysql'
S3_BUCKET = 'my-backups'

def create_backup():
    """Create MySQL backup"""
    timestamp = datetime.now().strftime('%Y%m%d_%H%M%S')
    backup_file = f"{BACKUP_DIR}/{DB_NAME}_{timestamp}.sql.gz"
    
    # Ensure backup directory exists
    os.makedirs(BACKUP_DIR, exist_ok=True)
    
    # Create backup
    cmd = f"mysqldump -h {DB_HOST} -u {DB_USER} -p{DB_PASSWORD} {DB_NAME} | gzip > {backup_file}"
    result = subprocess.run(cmd, shell=True, capture_output=True)
    
    if result.returncode == 0:
        print(f"Backup created: {backup_file}")
        return backup_file
    else:
        print(f"Backup failed: {result.stderr.decode()}")
        return None

def upload_to_s3(file_path):
    """Upload backup to S3"""
    s3 = boto3.client('s3')
    file_name = os.path.basename(file_path)
    
    try:
        s3.upload_file(file_path, S3_BUCKET, f"mysql/{file_name}")
        print(f"Uploaded to S3: s3://{S3_BUCKET}/mysql/{file_name}")
        return True
    except Exception as e:
        print(f"S3 upload failed: {e}")
        return False

def cleanup_old_backups(days=7):
    """Remove local backups older than specified days"""
    cutoff = datetime.now().timestamp() - (days * 86400)
    
    for filename in os.listdir(BACKUP_DIR):
        file_path = os.path.join(BACKUP_DIR, filename)
        if os.path.isfile(file_path) and filename.endswith('.sql.gz'):
            if os.path.getmtime(file_path) < cutoff:
                os.remove(file_path)
                print(f"Removed old backup: {filename}")

if __name__ == '__main__':
    backup_file = create_backup()
    if backup_file:
        upload_to_s3(backup_file)
        cleanup_old_backups(7)
```

## Best Practices

### 1. Error Handling
```bash
# Bash
set -e  # Exit on error
set -u  # Exit on undefined variable
set -o pipefail  # Exit on pipe failure

# Python
try:
    # risky operation
    result = operation()
except Exception as e:
    print(f"Error: {e}", file=sys.stderr)
    sys.exit(1)
```

### 2. Logging
```python
import logging

logging.basicConfig(
    level=logging.INFO,
    format='%(asctime)s - %(name)s - %(levelname)s - %(message)s',
    handlers=[
        logging.FileHandler('script.log'),
        logging.StreamHandler()
    ]
)

logger = logging.getLogger(__name__)
logger.info('Script started')
logger.error('An error occurred')
```

### 3. Configuration Files
```python
import configparser

# Read config
config = configparser.ConfigParser()
config.read('config.ini')

db_host = config['database']['host']
db_user = config['database']['user']
```

### 4. Idempotency
```bash
# Check before creating
if [ ! -d "/var/app" ]; then
    mkdir /var/app
fi

# Python
if not os.path.exists(directory):
    os.makedirs(directory)
```

## Interview Questions

1. What is the difference between `#!/bin/bash` and `#!/bin/sh`?
2. How do you handle errors in bash scripts?
3. Explain the difference between `$@` and `$*` in bash
4. How do you make a script executable?
5. What is the purpose of `set -e` in bash?
6. How do you pass arguments to a Python script?
7. What is the difference between `subprocess.run()` and `os.system()`?
8. How do you schedule scripts to run automatically?
9. How do you make scripts portable across different systems?
10. What are some security considerations when writing scripts?

## Practical Exercises

1. Write a backup automation script
2. Create a log rotation script
3. Build a system monitoring script
4. Write a deployment automation script
5. Create a database backup script
6. Build a cleanup script for old files
7. Write a script to manage cloud resources
8. Create a script to parse and analyze logs

## Resources

- [Bash Scripting Tutorial](https://www.shellscript.sh/)
- [Advanced Bash-Scripting Guide](https://tldp.org/LDP/abs/html/)
- [Python Documentation](https://docs.python.org/3/)
- [Python for DevOps](https://www.oreilly.com/library/view/python-for-devops/9781492057680/)
- [Shell Check](https://www.shellcheck.net/) - Shell script analyzer
