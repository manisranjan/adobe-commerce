# Adobe Commerce Local Setup - Mac

A comprehensive guide to setting up Adobe Commerce 2.4.8 development environment on macOS with all required services.

---

## Quick Start Checklist

**Core Installation:**
- [ ] Install Homebrew
- [ ] Install Nginx 1.28
- [ ] Configure SSL certificate
- [ ] Install PHP 8.3 (or 8.4)
- [ ] Install MariaDB 11.4
- [ ] Install OpenSearch 3
- [ ] Install Redis/Valkey 8
- [ ] Install RabbitMQ 4.1
- [ ] Install Composer 2.8
- [ ] Install Adobe Commerce 2.4.8-p3
- [ ] Configure Nginx for Magento
- [ ] Start all services
- [ ] Access frontend and admin

**Development Tools (Optional but Recommended):**
- [ ] Install Xdebug (for debugging)
- [ ] Install MailHog (for email testing)
- [ ] Configure IDE for debugging

**Estimated setup time:** 2-4 hours (core) + 30 minutes (dev tools)

---

## Table of Contents
1. [Prerequisites](#prerequisites)
2. [System Requirements](#system-requirements)
3. [Installing Homebrew](#installing-homebrew)
4. [Installing Nginx](#installing-nginx)
5. [Installing SSL Certificate](#installing-ssl-certificate)
6. [Installing PHP](#installing-php)
7. [Installing MariaDB](#installing-mariadb)
8. [Installing OpenSearch](#installing-opensearch)
9. [Installing Valkey (Redis Alternative)](#installing-valkey-redis-alternative)
10. [Installing RabbitMQ](#installing-rabbitmq)
11. [Installing Composer](#installing-composer)
12. [Adobe Commerce Installation](#adobe-commerce-installation)
13. [Nginx Configuration Guide](#configuration)
    - [Configure Virtual Host](#configure-nginx-virtual-host-for-adobe-commerce)
    - [Multiple Sites Setup](#multiple-sites-configuration)
    - [Troubleshooting Nginx](#nginx-configuration-troubleshooting)
14. [Development Tools](#development-tools)
    - [Installing Xdebug](#installing-xdebug)
    - [Installing MailHog](#installing-mailhog)
    - [IDE Extensions](#recommended-ide-extensions)
15. [Troubleshooting](#troubleshooting)
16. [Useful Commands](#useful-commands)
17. [Performance Optimization](#performance-optimization)
18. [Access Your Site](#access-your-site)
19. [Additional Resources](#additional-resources)

---

## Prerequisites

Before starting, ensure you have:
- macOS (10.14 or later recommended, macOS 11+ for Apple Silicon)
- Terminal access
- Administrator privileges
- At least 8GB RAM (16GB recommended)
- 30GB free disk space minimum
- Internet connection for downloading packages

---

## System Requirements

Adobe Commerce 2.4.8 requires specific software versions. This guide is optimized for the latest release.

### Version Compatibility Matrix

| Software Dependencies | 2.4.8-p3 | 2.4.8-p2 | 2.4.8-p1 | 2.4.8 |
|-----------------------|----------|----------|----------|-------|
| **Composer**          | 2.8      |   2.8    | 2.8      | 2.8   |
| **OpenSearch**        | 3        | 3        | 2        | 2     |
| **MariaDB**           | 11.4     | 11.4     | 11.4     | 11.4  |
| **New Relic**         | 11.5.0.18+, 10.15.0.4+ | 11.5.0.18+, 10.15.0.4+ |
| **PHP**               | 8.4,8.3  | 8.4,8.3  | 8.4,8.3  | 8.4,8.3 |
| **RabbitMQ**          | 4.1      | 4.1      | 4.1      | 4.1   |
| **ActiveMQ Artemis**  | 2        | --       | --       | --    |
| **Valkey**            | 8        | 8        | 8        | 8     |
| **nginx**             | 1.28     | 1.28     | 1.26     | 1.26  |

### What We'll Install in This Guide

This guide covers installation for **Adobe Commerce 2.4.8-p3** (latest):

- ✅ **Nginx 1.28** - Web server
- ✅ **PHP 8.3** - Application runtime (8.4 also supported)
- ✅ **MariaDB 11.4** - Database server
- ✅ **OpenSearch 3** - Search engine (replaces Elasticsearch)
- ✅ **Valkey 8** - Caching layer (Redis fork)
- ✅ **RabbitMQ 4.1** - Message queue system
- ✅ **Composer 2.8** - Dependency manager
- ✅ **SSL Certificate** - HTTPS support

### Optional Components

- **New Relic** - Application monitoring (optional)
- **ActiveMQ Artemis 2** - Alternative message broker (optional)

---

## Installing Homebrew

Homebrew is a package manager for macOS that simplifies the installation of software and services.

### Install Homebrew

Open your terminal and run the following command:

```bash
/bin/bash -c "$(curl -fsSL https://raw.githubusercontent.com/Homebrew/install/HEAD/install.sh)"
```

**What this does:**
- Downloads and installs Homebrew package manager
- Sets up necessary directories and permissions
- Adds Homebrew to your PATH

### Verify Installation

```bash
brew --version
```

You should see output like: `Homebrew 4.x.x`

### Additional Resources

For more details, visit the official Homebrew documentation: [https://brew.sh/](https://brew.sh/)

### Update Homebrew (If Already Installed)

If Homebrew is already installed, update the repository index:

```bash
brew update
```

**Why update?**
- Ensures you get the latest package versions
- Updates formula definitions
- Fixes potential security issues

---

## Installing Nginx

Nginx is a high-performance web server that will serve your Adobe Commerce application. Adobe Commerce 2.4.8-p3 requires Nginx 1.28.

### Install Nginx via Homebrew

```bash
brew install nginx
```

**What gets installed:**
- Nginx web server (version 1.28 or compatible)
- Configuration files in `/opt/homebrew/etc/nginx/`
- HTML root directory in `/opt/homebrew/var/www/`
- Default port: 8080 (we'll configure for 80/443)

### Check Nginx Version

Once installed, verify the installation and ensure it meets requirements:

```bash
nginx -v
```

Expected output: `nginx version: nginx/1.28.x` (or newer)

**Note:** If you get an older version, update Homebrew:
```bash
brew update
brew upgrade nginx
```

### Start Nginx Server

```bash
nginx
```

**Alternatively, use Homebrew services:**

```bash
# Start Nginx
brew services start nginx

# Stop Nginx
brew services stop nginx

# Restart Nginx
brew services restart nginx
```

### Test Nginx Installation

Open your browser and visit: [http://127.0.0.1:8080](http://127.0.0.1:8080)

You should see the default Nginx welcome page.

### Stop Nginx Server

To stop the Nginx server, use the stop signal:

```bash
nginx -s stop
```

**Other Nginx commands:**
```bash
nginx -s reload   # Reload configuration
nginx -s reopen   # Reopen log files
nginx -s quit     # Graceful shutdown
nginx -t          # Test configuration
```

### Nginx Configuration Locations

- **Config file:** `/opt/homebrew/etc/nginx/nginx.conf`
- **Sites config:** `/opt/homebrew/etc/nginx/servers/`
- **Logs:** `/opt/homebrew/var/log/nginx/`
- **Web root:** `/opt/homebrew/var/www/`

---

## Installing SSL Certificate

SSL certificates enable HTTPS connections for secure local development. We'll create a self-signed certificate for development purposes.

### Create SSL Directory

Navigate to the Homebrew SSL directory:

```bash
mkdir -p /opt/homebrew/etc/ssl
cd /opt/homebrew/etc/ssl
```

### Generate Self-Signed SSL Certificate

Run the following command to create a self-signed certificate valid for 1 year:

```bash
sudo openssl req \
  -x509 \
  -nodes \
  -days 365 \
  -newkey rsa:2048 \
  -keyout /opt/homebrew/etc/ssl/self-signed.key \
  -out /opt/homebrew/etc/ssl/self-signed.crt
```

**Command breakdown:**
- `req` - Certificate request utility
- `-x509` - Output self-signed certificate
- `-nodes` - Don't encrypt private key
- `-days 365` - Certificate valid for 1 year
- `-newkey rsa:2048` - Generate new 2048-bit RSA key
- `-keyout` - Private key output location
- `-out` - Certificate output location

### Certificate Information Prompts

You'll be prompted to enter certificate details. You can use these example values:

```
Country Name (2 letter code) [AU]: US
State or Province Name (full name) [Some-State]: California
Locality Name (eg, city) []: San Francisco
Organization Name (eg, company) [Internet Widgits Pty Ltd]: Local Development
Organizational Unit Name (eg, section) []: Development
Common Name (e.g. server FQDN or YOUR name) []: local.magento.test
Email Address []: dev@local.test
```

**Important:** The `Common Name` should match your local domain (e.g., `local.magento.test`)

### Verify Certificate Files

Check that both files were created:

```bash
ls -la /opt/homebrew/etc/ssl/
```

You should see:
- `self-signed.crt` - SSL certificate
- `self-signed.key` - Private key

### Configure Nginx for SSL

Edit your Nginx site configuration:

```bash
nano /opt/homebrew/etc/nginx/servers/magento.conf
```

Add SSL configuration:

```nginx
server {
    listen 443 ssl;
    server_name local.magento.test;

    ssl_certificate /opt/homebrew/etc/ssl/self-signed.crt;
    ssl_certificate_key /opt/homebrew/etc/ssl/self-signed.key;

    # SSL Settings
    ssl_protocols TLSv1.2 TLSv1.3;
    ssl_ciphers HIGH:!aNULL:!MD5;
    ssl_prefer_server_ciphers on;

    root /Users/yourusername/Sites/local.magento.test;
    index index.php;

    # ... rest of your Nginx configuration
}
```

### Trust the Certificate (macOS)

To avoid browser warnings, add the certificate to macOS Keychain:

```bash
sudo security add-trusted-cert -d -r trustRoot \
  -k /Library/Keychains/System.keychain \
  /opt/homebrew/etc/ssl/self-signed.crt
```

### Test SSL Configuration

```bash
# Test Nginx configuration
nginx -t

# Reload Nginx
nginx -s reload
```

Visit `https://local.magento.test` in your browser.

---

## Installing PHP

Adobe Commerce 2.4.8 requires PHP 8.3 or 8.4 with specific extensions.

### Install PHP 8.3 (Recommended) or 8.4

```bash
# PHP 8.3 (Recommended - most stable for Adobe Commerce 2.4.8)
brew install php@8.3

# Or PHP 8.4 (Latest, fully supported)
brew install php@8.4
```

**Why PHP 8.3?**
- ✅ Proven stability with Adobe Commerce 2.4.8
- ✅ Better extension compatibility
- ✅ Recommended by Adobe for production

**Why PHP 8.4?**
- ✅ Latest PHP features and performance improvements
- ✅ Fully supported by Adobe Commerce 2.4.8
- ✅ Better for new projects

### Required PHP Extensions

Most extensions are included with Homebrew's PHP installation. Verify required extensions:

```bash
php -m | grep -E 'bcmath|ctype|curl|dom|gd|hash|iconv|intl|mbstring|openssl|pdo_mysql|simplexml|soap|sockets|sodium|xsl|zip|zlib'
```

**If any extensions are missing, install them:**

```bash
# For PHP 8.3
brew install php@8.3-intl
brew install php@8.3-bcmath
brew install php@8.3-gd
brew install php@8.3-soap
brew install php@8.3-xsl

# For PHP 8.4
brew install php@8.4-intl
brew install php@8.4-bcmath
brew install php@8.4-gd
brew install php@8.4-soap
brew install php@8.4-xsl
```

### Required PHP Extensions for Adobe Commerce 2.4.8

- `bcmath` - Precision mathematics
- `ctype` - Character type checking
- `curl` - HTTP requests
- `dom` - XML DOM manipulation
- `gd` - Image processing
- `hash` - Hashing functions
- `iconv` - Character encoding conversion
- `intl` - Internationalization
- `mbstring` - Multibyte string handling
- `openssl` - Cryptography
- `pdo_mysql` - MySQL database driver
- `simplexml` - XML parsing
- `soap` - SOAP protocol
- `sockets` - Network sockets
- `sodium` - Modern cryptography
- `xsl` - XSLT transformations
- `zip` - ZIP archive handling
- `zlib` - Compression

### Verify PHP Installation

```bash
php -v
php -m  # List all installed extensions
```

Expected output:
```
PHP 8.3.x (cli) (built: ...)
```

### Configure PHP for Adobe Commerce

Edit PHP configuration:

```bash
# For PHP 8.3
nano /opt/homebrew/etc/php/8.3/php.ini

# For PHP 8.4
nano /opt/homebrew/etc/php/8.4/php.ini
```

**Required settings for Adobe Commerce 2.4.8:**

```ini
; Memory and execution
memory_limit = 4G
max_execution_time = 1800
max_input_time = 1800

; Post and upload limits
post_max_size = 64M
upload_max_filesize = 64M

; Opcache (critical for performance)
opcache.enable = 1
opcache.enable_cli = 1
opcache.memory_consumption = 512
opcache.interned_strings_buffer = 16
opcache.max_accelerated_files = 130000
opcache.validate_timestamps = 1
opcache.revalidate_freq = 2
opcache.save_comments = 1

; Realpath cache
realpath_cache_size = 10M
realpath_cache_ttl = 7200

; Session
session.save_handler = redis
session.save_path = "tcp://127.0.0.1:6379?database=0"

; Compression
zlib.output_compression = On

; Error reporting (development)
display_errors = On
display_startup_errors = On
error_reporting = E_ALL

; Timezone
date.timezone = America/Los_Angeles
```

### Link PHP Version

```bash
# Unlink current PHP
brew unlink php

# Link PHP 8.3 (recommended)
brew link php@8.3 --force --overwrite

# Or link PHP 8.4
brew link php@8.4 --force --overwrite
```

### Start PHP-FPM

```bash
# For PHP 8.3
brew services start php@8.3

# For PHP 8.4
brew services start php@8.4

# Verify PHP-FPM is running
brew services list | grep php
```

### Test PHP-FPM

```bash
# Check PHP-FPM socket
ls -la /opt/homebrew/var/run/php-fpm.sock

# Or check PHP-FPM status
php-fpm -t
```

---

## Installing MariaDB

Adobe Commerce 2.4.8 requires MariaDB 11.4. MariaDB is a MySQL-compatible database server with enhanced performance.

### Why MariaDB over MySQL?

- ✅ **Better performance** for Adobe Commerce workloads
- ✅ **Enhanced replication** features
- ✅ **Fully compatible** with MySQL applications
- ✅ **Official requirement** for Adobe Commerce 2.4.8
- ✅ **Open-source** with active development

### Install MariaDB 11.4

```bash
brew install mariadb@11.4
```

**What gets installed:**
- MariaDB server 11.4.x
- MySQL-compatible command-line tools
- Configuration files in `/opt/homebrew/etc/my.cnf`
- Data directory in `/opt/homebrew/var/mysql/`

### Start MariaDB Service

```bash
brew services start mariadb@11.4
```

**Verify service is running:**
```bash
brew services list | grep mariadb
```

Expected output: `mariadb@11.4  started`

### Secure MariaDB Installation

Run the security script to improve database security:

```bash
mysql_secure_installation
```

**Follow the prompts:**

1. **Enter current password for root:** (press Enter if no password set)
2. **Switch to unix_socket authentication:** No
3. **Change root password:** Yes (set a strong password)
4. **Remove anonymous users:** Yes
5. **Disallow root login remotely:** Yes
6. **Remove test database:** Yes
7. **Reload privilege tables:** Yes

### Verify MariaDB Version

```bash
mysql --version
```

Expected output: `mysql  Ver 15.1 Distrib 11.4.x-MariaDB`

### Connect to MariaDB

```bash
# Login as root
mysql -u root -p
```

### Create Database and User for Adobe Commerce

Run these commands in the MySQL/MariaDB prompt:

```sql
-- Create database
CREATE DATABASE magento CHARACTER SET utf8mb4 COLLATE utf8mb4_unicode_ci;

-- Create user with strong password
CREATE USER 'magento'@'localhost' IDENTIFIED BY 'your_secure_password_here';

-- Grant privileges
GRANT ALL PRIVILEGES ON magento.* TO 'magento'@'localhost';

-- Apply privileges
FLUSH PRIVILEGES;

-- Verify database created
SHOW DATABASES;

-- Exit
EXIT;
```

### Configure MariaDB for Adobe Commerce

Edit MariaDB configuration:

```bash
nano /opt/homebrew/etc/my.cnf
```

**Add these optimizations for Adobe Commerce:**

```ini
[mysqld]
# Basic settings
default_storage_engine = innodb
innodb_file_per_table = 1

# Performance tuning
innodb_buffer_pool_size = 2G
innodb_log_file_size = 512M
innodb_flush_log_at_trx_commit = 2
innodb_flush_method = O_DIRECT

# Connection settings
max_connections = 500
max_allowed_packet = 64M

# Query cache (disabled for better performance with InnoDB)
query_cache_size = 0
query_cache_type = 0

# Temporary tables
tmp_table_size = 256M
max_heap_table_size = 256M

# Binary logging (optional, for replication/backups)
# log_bin = mysql-bin
# expire_logs_days = 7

# Character set
character_set_server = utf8mb4
collation_server = utf8mb4_unicode_ci

# InnoDB settings
innodb_lock_wait_timeout = 50
innodb_io_capacity = 2000
innodb_io_capacity_max = 4000

[client]
default-character-set = utf8mb4

[mysql]
default-character-set = utf8mb4
```

**Configuration explanation:**
- `innodb_buffer_pool_size`: Main memory cache (set to 50-70% of available RAM)
- `innodb_log_file_size`: Redo log size for crash recovery
- `max_connections`: Maximum concurrent connections
- `utf8mb4`: Full Unicode support (required for emojis, etc.)

### Restart MariaDB

Apply configuration changes:

```bash
brew services restart mariadb@11.4
```

### Verify Database Connection

Test connection with the magento user:

```bash
mysql -u magento -p magento
```

**Run a test query:**
```sql
SELECT VERSION();
SHOW VARIABLES LIKE 'character_set%';
EXIT;
```

### Database Backup and Restore (Optional)

**Create backup:**
```bash
mysqldump -u magento -p magento > magento_backup_$(date +%Y%m%d).sql
```

**Restore backup:**
```bash
mysql -u magento -p magento < magento_backup_20231127.sql
```

### Performance Monitoring

**Check MariaDB status:**
```bash
mysql -u root -p -e "SHOW ENGINE INNODB STATUS\G"
```

**Check database size:**
```bash
mysql -u root -p -e "SELECT table_schema AS 'Database', ROUND(SUM(data_length + index_length) / 1024 / 1024, 2) AS 'Size (MB)' FROM information_schema.tables GROUP BY table_schema;"
```

---

## Installing OpenSearch

Adobe Commerce 2.4.8 requires OpenSearch 3 (replaces Elasticsearch). OpenSearch is an open-source search and analytics engine.

### Why OpenSearch?

- ✅ **Open-source** alternative to Elasticsearch
- ✅ **Enhanced security** features
- ✅ **Better performance** for catalog search
- ✅ **Official requirement** for Adobe Commerce 2.4.8
- ✅ **Community-driven** development

### Install OpenSearch 3

```bash
brew install opensearch
```

**What gets installed:**
- OpenSearch 3.x server
- OpenSearch Dashboards (management UI)
- Configuration files in `/opt/homebrew/etc/opensearch/`
- Data directory in `/opt/homebrew/var/lib/opensearch/`

### Configure OpenSearch for Adobe Commerce

Before starting OpenSearch, configure it for single-node development:

```bash
nano /opt/homebrew/etc/opensearch/opensearch.yml
```

**Add/modify these settings:**

```yaml
# Cluster name
cluster.name: magento-opensearch

# Node name
node.name: magento-node-1

# Network settings
network.host: 127.0.0.1
http.port: 9200

# Discovery settings (single node)
discovery.type: single-node

# Disable security for local development (optional)
plugins.security.disabled: true

# Memory settings
bootstrap.memory_lock: false

# Path settings
path.data: /opt/homebrew/var/lib/opensearch
path.logs: /opt/homebrew/var/log/opensearch
```

### Set OpenSearch Environment Variables

Create or edit OpenSearch environment file:

```bash
nano /opt/homebrew/etc/opensearch/jvm.options.d/jvm.options
```

**Add memory settings (adjust based on your RAM):**

```
# Heap size (set to 50% of available RAM, max 32GB)
-Xms2g
-Xmx2g

# Garbage collection
-XX:+UseG1GC
-XX:MaxGCPauseMillis=200
```

**For machines with limited RAM (8GB or less):**
```
-Xms1g
-Xmx1g
```

### Start OpenSearch

```bash
brew services start opensearch
```

**Wait 30-60 seconds for OpenSearch to start**, then verify.

### Verify OpenSearch Installation

Check if OpenSearch is running:

```bash
curl http://localhost:9200
```

**Expected output:**
```json
{
  "name" : "magento-node-1",
  "cluster_name" : "magento-opensearch",
  "cluster_uuid" : "...",
  "version" : {
    "distribution" : "opensearch",
    "number" : "3.x.x",
    "build_type" : "tar",
    "build_hash" : "...",
    "build_date" : "...",
    "build_snapshot" : false,
    "lucene_version" : "9.x.x",
    "minimum_wire_compatibility_version" : "2.17.0",
    "minimum_index_compatibility_version" : "2.0.0"
  },
  "tagline" : "The OpenSearch Project: https://opensearch.org/"
}
```

### Check OpenSearch Cluster Health

```bash
curl http://localhost:9200/_cluster/health?pretty
```

**Expected output:**
```json
{
  "cluster_name" : "magento-opensearch",
  "status" : "green",
  "timed_out" : false,
  "number_of_nodes" : 1,
  "number_of_data_nodes" : 1
}
```

**Status meanings:**
- 🟢 **green** - All shards allocated (healthy)
- 🟡 **yellow** - Primary shards allocated, replicas not (acceptable for single-node)
- 🔴 **red** - Some primary shards not allocated (issue)

### Install OpenSearch Plugins (Optional but Recommended)

Adobe Commerce benefits from these OpenSearch plugins:

```bash
# Navigate to OpenSearch plugins directory
cd /opt/homebrew/opt/opensearch/bin

# Install analysis plugins for better search
./opensearch-plugin install analysis-icu
./opensearch-plugin install analysis-phonetic

# Restart OpenSearch to load plugins
brew services restart opensearch
```

### OpenSearch Dashboards (Optional)

For visual management and monitoring:

```bash
brew install opensearch-dashboards

# Start dashboards
brew services start opensearch-dashboards

# Access at: http://localhost:5601
```

### Test OpenSearch with Sample Data

Create a test index:

```bash
# Create index
curl -X PUT "localhost:9200/test_index" -H 'Content-Type: application/json' -d'
{
  "settings": {
    "number_of_shards": 1,
    "number_of_replicas": 0
  }
}'

# Add document
curl -X POST "localhost:9200/test_index/_doc/1" -H 'Content-Type: application/json' -d'
{
  "title": "Test Product",
  "price": 29.99
}'

# Search
curl -X GET "localhost:9200/test_index/_search?pretty"

# Delete test index
curl -X DELETE "localhost:9200/test_index"
```

### OpenSearch Memory and Performance

**Monitor OpenSearch memory:**
```bash
curl http://localhost:9200/_nodes/stats/jvm?pretty
```

**Check OpenSearch performance:**
```bash
curl http://localhost:9200/_cat/nodes?v
curl http://localhost:9200/_cat/indices?v
```

### Troubleshooting OpenSearch

**OpenSearch won't start:**
```bash
# Check logs
tail -f /opt/homebrew/var/log/opensearch/magento-opensearch.log

# Check if port 9200 is in use
lsof -i :9200

# Restart with verbose output
opensearch
```

**Out of memory errors:**
```bash
# Reduce heap size in jvm.options
nano /opt/homebrew/etc/opensearch/jvm.options.d/jvm.options

# Set to smaller values
-Xms512m
-Xmx512m
```

**Clear OpenSearch data (reset):**
```bash
# Stop OpenSearch
brew services stop opensearch

# Remove data
rm -rf /opt/homebrew/var/lib/opensearch/*

# Start OpenSearch
brew services start opensearch
```

### OpenSearch Configuration for Production

For production environments, consider:

```yaml
# Enable security
plugins.security.disabled: false

# Enable SSL
plugins.security.ssl.http.enabled: true

# Set cluster configuration
cluster.routing.allocation.disk.threshold_enabled: true
cluster.routing.allocation.disk.watermark.low: 85%
cluster.routing.allocation.disk.watermark.high: 90%

# Performance tuning
indices.memory.index_buffer_size: 30%
indices.queries.cache.size: 10%
```

---

## Installing Valkey (Redis Alternative)

Adobe Commerce 2.4.8 requires Valkey 8 for caching and session storage. Valkey is a high-performance data structure store and Redis fork.

### Why Valkey?

- ✅ **Redis-compatible** - Drop-in replacement for Redis
- ✅ **Enhanced performance** and reliability
- ✅ **Open-source** with active development
- ✅ **Official requirement** for Adobe Commerce 2.4.8
- ✅ **Better memory management** and persistence

### Install Valkey 8

Since Valkey is relatively new, you may need to install via source or use Redis as a compatible alternative:

**Option 1: Install Redis (Valkey-compatible)**

```bash
brew install redis
```

**Option 2: Install Valkey from source (if available)**

```bash
# Check if Valkey is available in Homebrew
brew search valkey

# If available
brew install valkey
```

For this guide, we'll use **Redis as it's Valkey-compatible** and widely available.

### Start Redis/Valkey

```bash
brew services start redis
```

### Verify Redis Installation

```bash
redis-cli ping
```

**Expected output:** `PONG`

**Check version:**
```bash
redis-cli --version
redis-server --version
```

### Configure Redis for Adobe Commerce

Edit Redis configuration:

```bash
nano /opt/homebrew/etc/redis.conf
```

**Add/modify these settings:**

```conf
# Bind to localhost only (security)
bind 127.0.0.1

# Port
port 6379

# Maximum memory
maxmemory 512mb
maxmemory-policy allkeys-lru

# Persistence (for session data)
save 900 1
save 300 10
save 60 10000

# AOF persistence (more durable)
appendonly yes
appendfsync everysec

# Disable protected mode for local development
protected-mode no

# Log level
loglevel notice

# Database count (Adobe Commerce uses multiple databases)
databases 16
```

**Configuration explanation:**
- `maxmemory`: Limit Redis memory usage
- `maxmemory-policy`: Evict least recently used keys when memory is full
- `save`: Snapshot to disk periodically
- `appendonly`: Enable AOF (Append Only File) for durability

### Restart Redis

Apply configuration changes:

```bash
brew services restart redis
```

### Test Redis

```bash
# Connect to Redis
redis-cli

# Test commands
127.0.0.1:6379> SET test "Hello from Redis"
127.0.0.1:6379> GET test
127.0.0.1:6379> DEL test
127.0.0.1:6379> PING
127.0.0.1:6379> INFO
127.0.0.1:6379> EXIT
```

### Configure Multiple Redis Instances (Recommended)

Adobe Commerce benefits from separate Redis instances for different purposes:

1. **Default cache** (port 6379)
2. **Page cache** (port 6380)
3. **Session storage** (port 6381)

**Create additional Redis instances:**

```bash
# Copy default config
cp /opt/homebrew/etc/redis.conf /opt/homebrew/etc/redis-page.conf
cp /opt/homebrew/etc/redis.conf /opt/homebrew/etc/redis-session.conf

# Edit page cache instance
nano /opt/homebrew/etc/redis-page.conf
```

**Modify port and settings for page cache:**
```conf
port 6380
maxmemory 1gb
maxmemory-policy volatile-lru
```

**Edit session instance:**
```bash
nano /opt/homebrew/etc/redis-session.conf
```

**Modify port for session:**
```conf
port 6381
maxmemory 256mb
maxmemory-policy noeviction
```

**Start additional instances:**
```bash
redis-server /opt/homebrew/etc/redis-page.conf --daemonize yes
redis-server /opt/homebrew/etc/redis-session.conf --daemonize yes
```

### Monitor Redis

**Check Redis status:**
```bash
redis-cli INFO
redis-cli INFO memory
redis-cli INFO stats
```

**Monitor in real-time:**
```bash
redis-cli --stat
redis-cli MONITOR
```

**Check all databases:**
```bash
redis-cli
127.0.0.1:6379> INFO keyspace
```

### Redis Performance Tuning

**For better performance:**
```bash
# Disable transparent huge pages (Linux)
echo never > /sys/kernel/mm/transparent_hugepage/enabled

# Set vm.overcommit_memory (add to /etc/sysctl.conf)
vm.overcommit_memory = 1
```

---

## Installing RabbitMQ

Adobe Commerce 2.4.8 requires RabbitMQ 4.1 for message queue processing and asynchronous operations.

### Why RabbitMQ?

- ✅ **Message queue system** for asynchronous processing
- ✅ **Improves performance** by offloading tasks
- ✅ **Reliable delivery** of messages
- ✅ **Scales well** with high traffic
- ✅ **Required** for Adobe Commerce B2B features

### Install RabbitMQ 4.1

```bash
brew install rabbitmq
```

**What gets installed:**
- RabbitMQ server 4.1.x (or latest)
- Management plugin for web UI
- Configuration files in `/opt/homebrew/etc/rabbitmq/`
- Default port: 5672 (AMQP), 15672 (Management UI)

### Start RabbitMQ

```bash
brew services start rabbitmq
```

**Add RabbitMQ to PATH:**
```bash
echo 'export PATH="/opt/homebrew/opt/rabbitmq/sbin:$PATH"' >> ~/.zshrc
source ~/.zshrc
```

### Enable RabbitMQ Management Plugin

The management plugin provides a web UI for monitoring:

```bash
rabbitmq-plugins enable rabbitmq_management
```

### Verify RabbitMQ Installation

**Check version:**
```bash
rabbitmqctl version
```

**Expected output:** `4.1.x` or newer

**Check status:**
```bash
rabbitmqctl status
```

### Access RabbitMQ Management UI

Open your browser and visit: [http://localhost:15672](http://localhost:15672)

**Default credentials:**
- Username: `guest`
- Password: `guest`

**Note:** Guest user only works from localhost.

### Create RabbitMQ User for Adobe Commerce

For security, create a dedicated user:

```bash
# Create user
rabbitmqctl add_user magento magento_password

# Set permissions
rabbitmqctl set_permissions -p / magento ".*" ".*" ".*"

# Set user tags (optional, for management access)
rabbitmqctl set_user_tags magento administrator
```

### Configure RabbitMQ Virtual Host

Create a dedicated virtual host for Adobe Commerce:

```bash
# Create virtual host
rabbitmqctl add_vhost magento_vhost

# Set permissions for magento user on virtual host
rabbitmqctl set_permissions -p magento_vhost magento ".*" ".*" ".*"

# List virtual hosts
rabbitmqctl list_vhosts
```

### Test RabbitMQ Connection

```bash
# List users
rabbitmqctl list_users

# List queues
rabbitmqctl list_queues

# Check cluster status
rabbitmqctl cluster_status
```

### RabbitMQ Configuration

Edit RabbitMQ configuration (create if doesn't exist):

```bash
nano /opt/homebrew/etc/rabbitmq/rabbitmq.conf
```

**Recommended settings for Adobe Commerce:**

```conf
# Listeners
listeners.tcp.default = 5672

# Management plugin
management.tcp.port = 15672

# Memory and disk limits
vm_memory_high_watermark.relative = 0.4
disk_free_limit.absolute = 2GB

# Heartbeat
heartbeat = 60

# Connection settings
channel_max = 2047
```

### Restart RabbitMQ

Apply configuration changes:

```bash
brew services restart rabbitmq
```

### Monitor RabbitMQ

**Via CLI:**
```bash
# List queues with details
rabbitmqctl list_queues name messages consumers

# List exchanges
rabbitmqctl list_exchanges

# List bindings
rabbitmqctl list_bindings

# Check memory usage
rabbitmqctl status | grep memory
```

**Via Management UI:**
- Visit [http://localhost:15672](http://localhost:15672)
- Login with user credentials
- View queues, exchanges, connections

### RabbitMQ Plugins

**List available plugins:**
```bash
rabbitmq-plugins list
```

**Enable useful plugins:**
```bash
# Already enabled
rabbitmq-plugins enable rabbitmq_management

# Optional: Enable additional plugins
rabbitmq-plugins enable rabbitmq_shovel
rabbitmq-plugins enable rabbitmq_shovel_management
```

### Troubleshooting RabbitMQ

**RabbitMQ won't start:**
```bash
# Check logs
tail -f /opt/homebrew/var/log/rabbitmq/rabbit@localhost.log

# Check if port is in use
lsof -i :5672
lsof -i :15672

# Reset RabbitMQ (WARNING: deletes all data)
rabbitmqctl stop_app
rabbitmqctl reset
rabbitmqctl start_app
```

**Can't access management UI:**
```bash
# Ensure plugin is enabled
rabbitmq-plugins enable rabbitmq_management

# Restart RabbitMQ
brew services restart rabbitmq

# Check firewall settings
```

**Memory warnings:**
```bash
# Check current memory usage
rabbitmqctl status | grep memory

# Adjust memory limit in rabbitmq.conf
vm_memory_high_watermark.relative = 0.6
```

---

## Installing Composer

Composer is a dependency manager for PHP. Adobe Commerce 2.4.8 requires Composer 2.8.

### Install Composer 2.8

```bash
brew install composer
```

### Verify Composer Version

```bash
composer --version
```

**Expected output:** `Composer version 2.8.x`

**If older version is installed:**
```bash
composer self-update
composer --version
```

### Configure Composer for Adobe Commerce

Composer needs authentication to access Adobe Commerce repository.

**Set authentication credentials:**

```bash
composer global config http-basic.repo.magento.com <public-key> <private-key>
```

**Get your Adobe Commerce access keys:**

1. Visit [Adobe Commerce Marketplace](https://commercemarketplace.adobe.com/)
2. Login to your account
3. Navigate to **My Profile** → **Access Keys**
4. Click **Create A New Access Key**
5. Give it a name (e.g., "Local Development")
6. Copy the **Public Key** (username) and **Private Key** (password)

### Configure Composer Settings

**Increase memory limit:**
```bash
composer config --global process-timeout 2000
composer config --global memory-limit -1
```

**Disable IPv6 (if connection issues):**
```bash
composer config --global disable-tls false
composer config --global secure-http true
```

### Composer Configuration File

View your global Composer configuration:

```bash
cat ~/.composer/config.json
```

**Example configuration:**
```json
{
    "http-basic": {
        "repo.magento.com": {
            "username": "your-public-key",
            "password": "your-private-key"
        }
    },
    "github-oauth": {
        "github.com": "your-github-token"
    },
    "process-timeout": 2000,
    "memory-limit": -1
}
```

### Test Composer Authentication

```bash
# Test Adobe repo access
composer create-project --repository-url=https://repo.magento.com/ magento/project-community-edition --dry-run

# Should show available versions without errors
```

### Composer Performance Optimization

**Enable parallel downloads:**
```bash
composer global require hirak/prestissimo
```

**Or use Composer 2's built-in parallel downloads:**
```bash
# Already enabled in Composer 2.x by default
composer config --global repo.packagist.org composer https://packagist.org
```

### Clear Composer Cache

If you encounter issues:

```bash
composer clear-cache
composer diagnose
```

---

## Adobe Commerce Installation

### Create Project Directory

```bash
mkdir -p ~/Sites/local.magento.test
cd ~/Sites/local.magento.test
```

### Install Adobe Commerce 2.4.8 via Composer

**For Magento Open Source (Community Edition):**
```bash
composer create-project --repository-url=https://repo.magento.com/ \
  magento/project-community-edition=2.4.8-p3 .
```

**For Adobe Commerce (Enterprise Edition):**
```bash
composer create-project --repository-url=https://repo.magento.com/ \
  magento/project-enterprise-edition=2.4.8-p3 .
```

**For B2B (if needed):**
```bash
composer require magento/extension-b2b:1.4.2
```

**Installation time:** This may take 10-30 minutes depending on your internet connection.

### Set File Permissions

```bash
cd ~/Sites/local.magento.test

# Set correct permissions for directories and files
find var generated vendor pub/static pub/media app/etc -type f -exec chmod g+w {} +
find var generated vendor pub/static pub/media app/etc -type d -exec chmod g+ws {} +

# Make Magento CLI executable
chmod u+x bin/magento

# Set ownership (optional, for web server access)
# chown -R :www-data .
```

### Configure env.php for Redis/Valkey

Before installation, optionally pre-configure Redis for better performance:

```bash
# We'll do this after installation in the Configuration section
```

### Install Adobe Commerce

Run the installation command with all required parameters:

```bash
bin/magento setup:install \
  --base-url=https://local.magento.test \
  --base-url-secure=https://local.magento.test \
  --use-secure=1 \
  --use-secure-admin=1 \
  --db-host=localhost \
  --db-name=magento \
  --db-user=magento \
  --db-password=your_secure_password_here \
  --admin-firstname=Admin \
  --admin-lastname=User \
  --admin-email=admin@local.test \
  --admin-user=admin \
  --admin-password=Admin@123! \
  --language=en_US \
  --currency=USD \
  --timezone=America/Los_Angeles \
  --use-rewrites=1 \
  --search-engine=opensearch \
  --opensearch-host=localhost \
  --opensearch-port=9200 \
  --opensearch-index-prefix=magento2 \
  --opensearch-timeout=15 \
  --opensearch-enable-auth=0 \
  --session-save=redis \
  --session-save-redis-host=127.0.0.1 \
  --session-save-redis-port=6379 \
  --session-save-redis-db=2 \
  --cache-backend=redis \
  --cache-backend-redis-server=127.0.0.1 \
  --cache-backend-redis-port=6379 \
  --cache-backend-redis-db=0 \
  --page-cache=redis \
  --page-cache-redis-server=127.0.0.1 \
  --page-cache-redis-port=6379 \
  --page-cache-redis-db=1
```

**Installation parameters explained:**

| Parameter | Description | Value |
|-----------|-------------|-------|
| `--base-url` | Store URL | https://local.magento.test |
| `--use-secure` | Use HTTPS | 1 (enabled) |
| `--db-host` | Database host | localhost |
| `--db-name` | Database name | magento |
| `--search-engine` | Search engine | opensearch (not elasticsearch) |
| `--opensearch-host` | OpenSearch host | localhost |
| `--opensearch-port` | OpenSearch port | 9200 |
| `--session-save` | Session storage | redis |
| `--cache-backend` | Cache backend | redis |

**Installation time:** 5-15 minutes

### Post-Installation Steps

After successful installation, run these commands:

```bash
# Set developer mode
bin/magento deploy:mode:set developer

# Disable two-factor authentication (for local development)
bin/magento module:disable Magento_AdminAdobeImsTwoFactorAuth
bin/magento module:disable Magento_TwoFactorAuth

# Run setup upgrade
bin/magento setup:upgrade

# Compile dependency injection
bin/magento setup:di:compile

# Deploy static content
bin/magento setup:static-content:deploy -f en_US

# Reindex all
bin/magento indexer:reindex

# Flush cache
bin/magento cache:flush
```

### Configure RabbitMQ

Enable and configure RabbitMQ for message queues:

```bash
bin/magento setup:config:set \
  --amqp-host=localhost \
  --amqp-port=5672 \
  --amqp-user=magento \
  --amqp-password=magento_password \
  --amqp-virtualhost=magento_vhost
```

### Verify Installation

**Check installation status:**
```bash
bin/magento --version
```

**Expected output:** `Magento CLI version 2.4.8-p3`

**Check module status:**
```bash
bin/magento module:status
```

**Check configuration:**
```bash
# Check OpenSearch connection
curl http://localhost:9200/_cat/indices?v | grep magento

# Check Redis connection
redis-cli KEYS "magento*"

# Check database
mysql -u magento -p magento -e "SHOW TABLES;"
```

### Create Admin User (if needed)

If you need additional admin users:

```bash
bin/magento admin:user:create \
  --admin-user=developer \
  --admin-password=Developer@123! \
  --admin-email=dev@local.test \
  --admin-firstname=Developer \
  --admin-lastname=User
```

### Install Sample Data (Optional)

For development and testing:

```bash
# Deploy sample data
bin/magento sampledata:deploy

# Run setup upgrade
bin/magento setup:upgrade

# Reindex and cache flush
bin/magento indexer:reindex
bin/magento cache:flush
```

**Note:** Sample data installation requires authentication and may take 30-60 minutes.

---

## Configuration

### Configure Nginx Virtual Host for Adobe Commerce

Adobe Commerce requires specific Nginx configuration to handle URL rewrites, static files, and PHP processing correctly.

#### Step 1: Navigate to Nginx Servers Directory

```bash
cd /opt/homebrew/etc/nginx/servers
```

**Note:** If the `servers` directory doesn't exist, create it:
```bash
mkdir -p /opt/homebrew/etc/nginx/servers
```

#### Step 2: Create Magento Configuration File

Create a new configuration file for your Adobe Commerce site:

```bash
nano magento.conf
```

#### Step 3: Choose Your Configuration Approach

Adobe Commerce provides two approaches for Nginx configuration:

**Option A: Using Adobe's Sample Configuration (Recommended)**

This approach uses Adobe Commerce's built-in `nginx.conf.sample` file which is updated with each release.

```nginx
# Define the PHP-FPM backend
upstream fastcgi_backend {
    server 127.0.0.1:9000;
    # Or use Unix socket (more efficient):
    # server unix:/opt/homebrew/var/run/php-fpm.sock;
}

server {
    listen 80;
    listen [::]:80;
    server_name local.magento.test;
    
    # Redirect HTTP to HTTPS
    return 301 https://$server_name$request_uri;
}

server {
    listen 443 ssl http2;
    listen [::]:443 ssl http2;
    server_name local.magento.test;
    
    # SSL Configuration
    ssl_certificate /opt/homebrew/etc/ssl/self-signed.crt;
    ssl_certificate_key /opt/homebrew/etc/ssl/self-signed.key;
    
    # Set Magento root directory and mode
    set $MAGE_ROOT /Users/yourusername/Sites/local.magento.test;
    set $MAGE_MODE developer; # or production
    
    # Include Adobe Commerce sample configuration
    include /Users/yourusername/Sites/local.magento.test/nginx.conf.sample;
}
```

**Configuration explained:**
- `upstream fastcgi_backend` - Defines PHP-FPM connection (port 9000 or Unix socket)
- `server_name` - Your local domain
- `$MAGE_ROOT` - Path to your Adobe Commerce installation
- `$MAGE_MODE` - Set to `developer` for development or `production` for production
- `include nginx.conf.sample` - Uses Adobe's official configuration

**Benefits of using nginx.conf.sample:**
- ✅ Always up-to-date with Adobe Commerce version
- ✅ Handles all edge cases correctly
- ✅ Includes security best practices
- ✅ Easier to maintain

**Option B: Full Custom Configuration**

If you prefer complete control or need custom modifications:

```nginx
# Define the PHP-FPM backend
upstream fastcgi_backend {
    server unix:/opt/homebrew/var/run/php-fpm.sock;
}

# HTTP server - redirect to HTTPS
server {
    listen 80;
    listen [::]:80;
    server_name local.magento.test;
    return 301 https://$server_name$request_uri;
}

# HTTPS server - main configuration
server {
    listen 443 ssl http2;
    listen [::]:443 ssl http2;
    server_name local.magento.test;

    # SSL Configuration
    ssl_certificate /opt/homebrew/etc/ssl/self-signed.crt;
    ssl_certificate_key /opt/homebrew/etc/ssl/self-signed.key;
    ssl_protocols TLSv1.2 TLSv1.3;
    ssl_ciphers HIGH:!aNULL:!MD5;
    ssl_prefer_server_ciphers on;

    # Magento variables
    set $MAGE_ROOT /Users/yourusername/Sites/local.magento.test;
    set $MAGE_MODE developer;

    # Document root
    root $MAGE_ROOT/pub;

    index index.php;
    autoindex off;
    charset UTF-8;
    error_page 404 403 = /errors/404.php;

    # Security headers
    add_header X-Frame-Options "SAMEORIGIN" always;
    add_header X-Content-Type-Options "nosniff" always;
    add_header X-XSS-Protection "1; mode=block" always;

    # PHP entry point for setup application
    location ~* ^/setup($|/) {
        root $MAGE_ROOT;
        location ~ ^/setup/index.php {
            fastcgi_pass fastcgi_backend;
            fastcgi_param PHP_FLAG "session.auto_start=off \n suhosin.session.cryptua=off";
            fastcgi_param PHP_VALUE "memory_limit=756M \n max_execution_time=600";
            fastcgi_read_timeout 600s;
            fastcgi_connect_timeout 600s;
            fastcgi_index index.php;
            fastcgi_param SCRIPT_FILENAME $document_root$fastcgi_script_name;
            include fastcgi_params;
        }

        location ~ ^/setup/(?!pub/). {
            deny all;
        }

        location ~ ^/setup/pub/ {
            add_header X-Frame-Options "SAMEORIGIN";
        }
    }

    # PHP entry point for update application
    location ~* ^/update($|/) {
        root $MAGE_ROOT;

        location ~ ^/update/index.php {
            fastcgi_split_path_info ^(/update/index.php)(/.+)$;
            fastcgi_pass fastcgi_backend;
            fastcgi_index index.php;
            fastcgi_param SCRIPT_FILENAME $document_root$fastcgi_script_name;
            fastcgi_param PATH_INFO $fastcgi_path_info;
            include fastcgi_params;
        }

        location ~ ^/update/(?!pub/). {
            deny all;
        }

        location ~ ^/update/pub/ {
            add_header X-Frame-Options "SAMEORIGIN";
        }
    }

    # Main location
    location / {
        try_files $uri $uri/ /index.php$is_args$args;
    }

    # Public files location
    location /pub/ {
        location ~ ^/pub/media/(downloadable|customer|import|theme_customization/.*\.xml) {
            deny all;
        }
        alias $MAGE_ROOT/pub/;
        add_header X-Frame-Options "SAMEORIGIN";
    }

    # Static files caching
    location /static/ {
        expires max;

        location ~ ^/static/version {
            rewrite ^/static/(version\d*/)?(.*)$ /static/$2 last;
        }

        location ~* \.(ico|jpg|jpeg|png|gif|svg|js|css|swf|eot|ttf|otf|woff|woff2|json)$ {
            add_header Cache-Control "public";
            add_header X-Frame-Options "SAMEORIGIN";
            expires +1y;

            if (!-f $request_filename) {
                rewrite ^/static/?(.*)$ /static.php?resource=$1 last;
            }
        }
        
        location ~* \.(zip|gz|gzip|bz2|csv|xml)$ {
            add_header Cache-Control "no-store";
            add_header X-Frame-Options "SAMEORIGIN";
            expires off;

            if (!-f $request_filename) {
               rewrite ^/static/?(.*)$ /static.php?resource=$1 last;
            }
        }
        
        if (!-f $request_filename) {
            rewrite ^/static/?(.*)$ /static.php?resource=$1 last;
        }
        add_header X-Frame-Options "SAMEORIGIN";
    }

    # Media files
    location /media/ {
        try_files $uri $uri/ /get.php$is_args$args;

        location ~ ^/media/theme_customization/.*\.xml {
            deny all;
        }

        location ~* \.(ico|jpg|jpeg|png|gif|svg|js|css|swf|eot|ttf|otf|woff|woff2)$ {
            add_header Cache-Control "public";
            add_header X-Frame-Options "SAMEORIGIN";
            expires +1y;
            try_files $uri $uri/ /get.php$is_args$args;
        }
        
        location ~* \.(zip|gz|gzip|bz2|csv|xml)$ {
            add_header Cache-Control "no-store";
            add_header X-Frame-Options "SAMEORIGIN";
            expires off;
            try_files $uri $uri/ /get.php$is_args$args;
        }
        add_header X-Frame-Options "SAMEORIGIN";
    }

    location /media/customer/ {
        deny all;
    }

    location /media/downloadable/ {
        deny all;
    }

    location /media/import/ {
        deny all;
    }

    # PHP entry point
    location ~ ^/(index|get|static|errors/report|errors/404|errors/503|health_check)\.php$ {
        try_files $uri =404;
        fastcgi_pass fastcgi_backend;
        fastcgi_buffers 1024 4k;

        fastcgi_param PHP_FLAG "session.auto_start=off \n suhosin.session.cryptua=off";
        fastcgi_param PHP_VALUE "memory_limit=756M \n max_execution_time=18000";
        fastcgi_read_timeout 600s;
        fastcgi_connect_timeout 600s;

        fastcgi_index index.php;
        fastcgi_param SCRIPT_FILENAME $document_root$fastcgi_script_name;
        include fastcgi_params;
    }

    # Gzip compression
    gzip on;
    gzip_disable "msie6";
    gzip_comp_level 6;
    gzip_min_length 1100;
    gzip_buffers 16 8k;
    gzip_proxied any;
    gzip_types
        text/plain
        text/css
        text/js
        text/xml
        text/javascript
        application/javascript
        application/x-javascript
        application/json
        application/xml
        application/xml+rss
        image/svg+xml;
    gzip_vary on;

    # Deny access to sensitive files
    location ~* (\.php$|\.htaccess$|\.git) {
        deny all;
    }
}
```

#### Step 4: Update Configuration Variables

**Important:** Replace `yourusername` with your actual macOS username:

```bash
# Find your username
whoami

# Update the configuration file
sed -i '' 's/yourusername/YOUR_ACTUAL_USERNAME/g' magento.conf
```

**Or manually edit these lines:**
- `set $MAGE_ROOT /Users/YOUR_USERNAME/Sites/local.magento.test;`
- `include /Users/YOUR_USERNAME/Sites/local.magento.test/nginx.conf.sample;` (if using Option A)

#### Step 5: Configure PHP-FPM Connection

**Choose between TCP socket or Unix socket:**

**Option 1: TCP Socket (port 9000)**
```nginx
upstream fastcgi_backend {
    server 127.0.0.1:9000;
}
```

**Option 2: Unix Socket (recommended - faster)**
```nginx
upstream fastcgi_backend {
    server unix:/opt/homebrew/var/run/php-fpm.sock;
}
```

**Configure PHP-FPM to match:**

Edit PHP-FPM configuration:
```bash
nano /opt/homebrew/etc/php/8.3/php-fpm.d/www.conf
```

**For TCP socket:**
```ini
listen = 127.0.0.1:9000
```

**For Unix socket:**
```ini
listen = /opt/homebrew/var/run/php-fpm.sock
listen.owner = nobody
listen.group = nobody
listen.mode = 0660
```

Restart PHP-FPM:
```bash
brew services restart php@8.3
```

#### Step 6: Update Main Nginx Configuration

Edit the main Nginx configuration:

```bash
nano /opt/homebrew/etc/nginx/nginx.conf
```

**Ensure the servers directory is included:**

```nginx
http {
    # ... other settings ...
    
    # Include server configurations
    include servers/*;
}
```

#### Step 7: Update Hosts File

Add your local domain to the hosts file:

```bash
sudo nano /etc/hosts
```

Add this line:
```
127.0.0.1    local.magento.test
```

**For IPv6 support:**
```
127.0.0.1    local.magento.test
::1          local.magento.test
```

#### Step 8: Test and Reload Nginx

**Test Nginx configuration for syntax errors:**

```bash
nginx -t
```

**Expected output:**
```
nginx: the configuration file /opt/homebrew/etc/nginx/nginx.conf syntax is ok
nginx: configuration file /opt/homebrew/etc/nginx/nginx.conf test is successful
```

**If there are errors:**
- Check file paths
- Verify SSL certificate paths
- Ensure $MAGE_ROOT is correct
- Check PHP-FPM socket/port matches

**Reload Nginx:**
```bash
nginx -s reload

# Or restart Nginx service
brew services restart nginx
```

#### Step 9: Verify Configuration

**Check Nginx is listening on correct ports:**

```bash
lsof -i :80
lsof -i :443
```

**Test HTTP to HTTPS redirect:**
```bash
curl -I http://local.magento.test
```

Should return `301` redirect to HTTPS.

**Test HTTPS:**
```bash
curl -k https://local.magento.test
```

Should return Adobe Commerce frontend HTML.

### Multiple Sites Configuration

If you need to run multiple Adobe Commerce installations:

#### Create Separate Configuration Files

```bash
cd /opt/homebrew/etc/nginx/servers

# Site 1
nano site1.magento.test.conf

# Site 2
nano site2.magento.test.conf
```

#### Update Hosts File

```bash
sudo nano /etc/hosts
```

```
127.0.0.1    site1.magento.test
127.0.0.1    site2.magento.test
```

#### Each Configuration Uses Different:
- `server_name`
- `$MAGE_ROOT` path
- SSL certificates (can be same for local dev)

**Example for multiple sites:**

```nginx
# site1.magento.test.conf
upstream fastcgi_backend_site1 {
    server unix:/opt/homebrew/var/run/php-fpm.sock;
}

server {
    listen 443 ssl http2;
    server_name site1.magento.test;
    set $MAGE_ROOT /Users/yourusername/Sites/site1.magento.test;
    # ... rest of configuration
}

# site2.magento.test.conf
upstream fastcgi_backend_site2 {
    server unix:/opt/homebrew/var/run/php-fpm.sock;
}

server {
    listen 443 ssl http2;
    server_name site2.magento.test;
    set $MAGE_ROOT /Users/yourusername/Sites/site2.magento.test;
    # ... rest of configuration
}
```

### Nginx Configuration Troubleshooting

#### Issue: 502 Bad Gateway

**Cause:** PHP-FPM not running or wrong socket/port

**Solution:**
```bash
# Check PHP-FPM status
brew services list | grep php

# Check PHP-FPM is listening
lsof -i :9000
# Or for Unix socket
ls -la /opt/homebrew/var/run/php-fpm.sock

# Restart PHP-FPM
brew services restart php@8.3
```

#### Issue: 404 Not Found

**Cause:** Incorrect document root or missing nginx.conf.sample

**Solution:**
```bash
# Verify MAGE_ROOT path exists
ls -la /Users/yourusername/Sites/local.magento.test

# Check pub directory exists
ls -la /Users/yourusername/Sites/local.magento.test/pub

# Verify nginx.conf.sample exists (for Option A)
ls -la /Users/yourusername/Sites/local.magento.test/nginx.conf.sample
```

#### Issue: Static files not loading (CSS/JS missing)

**Cause:** Permissions or incorrect static file configuration

**Solution:**
```bash
cd ~/Sites/local.magento.test

# Deploy static content
bin/magento setup:static-content:deploy -f

# Set permissions
chmod -R 755 pub/static

# Clear cache
bin/magento cache:flush
```

#### Issue: Admin panel not loading

**Cause:** Missing setup routes or security headers

**Solution:**
- Ensure setup location block is included
- Check for .htaccess blocking (shouldn't exist with Nginx)
- Verify admin URL: `bin/magento info:adminuri`

#### Issue: nginx.conf.sample not found (Option A)

**Cause:** File doesn't exist or wrong path

**Solution:**
```bash
# Check if file exists
ls -la ~/Sites/local.magento.test/nginx.conf.sample

# If missing, it should be in the Adobe Commerce root
# Verify you're using the correct path

# Copy from Adobe Commerce repository if needed
curl -o ~/Sites/local.magento.test/nginx.conf.sample \
  https://raw.githubusercontent.com/magento/magento2/2.4.8-p3/nginx.conf.sample
```

### Complete Nginx Configuration Example

Here's a complete, production-ready configuration:

**Recommended for most users - uses Adobe's maintained configuration with SSL and performance optimizations.**

```nginx
# PHP-FPM backend
upstream fastcgi_backend {
    server unix:/opt/homebrew/var/run/php-fpm.sock;
}

# HTTP - redirect to HTTPS
server {
    listen 80;
    listen [::]:80;
    server_name local.magento.test;
    return 301 https://$host$request_uri;
}

# HTTPS - main server block
server {
    listen 443 ssl http2;
    listen [::]:443 ssl http2;
    server_name local.magento.test;

    # SSL certificates
    ssl_certificate /opt/homebrew/etc/ssl/self-signed.crt;
    ssl_certificate_key /opt/homebrew/etc/ssl/self-signed.key;
    ssl_protocols TLSv1.2 TLSv1.3;
    ssl_ciphers 'ECDHE-ECDSA-AES128-GCM-SHA256:ECDHE-RSA-AES128-GCM-SHA256:ECDHE-ECDSA-AES256-GCM-SHA384:ECDHE-RSA-AES256-GCM-SHA384';
    ssl_prefer_server_ciphers on;

    # Magento root and mode
    set $MAGE_ROOT /Users/yourusername/Sites/local.magento.test;
    set $MAGE_MODE developer;

    # Include Adobe Commerce sample configuration
    include /Users/yourusername/Sites/local.magento.test/nginx.conf.sample;
}
```

**Remember to:**
1. Replace `yourusername` with your actual macOS username
2. Adjust `$MAGE_ROOT` to your installation path
3. Change `$MAGE_MODE` to `production` when ready

### Quick Setup Script

Save time with this automated setup script:

```bash
# Create setup script
nano ~/setup-nginx-magento.sh
```

**Script content:**
```bash
#!/bin/bash

# Get current user
CURRENT_USER=$(whoami)
SITE_NAME="local.magento.test"
MAGE_ROOT="/Users/${CURRENT_USER}/Sites/${SITE_NAME}"

echo "Setting up Nginx for Adobe Commerce..."
echo "Site: ${SITE_NAME}"
echo "Root: ${MAGE_ROOT}"

# Create servers directory if it doesn't exist
mkdir -p /opt/homebrew/etc/nginx/servers

# Create Nginx configuration
cat > /opt/homebrew/etc/nginx/servers/magento.conf << EOF
upstream fastcgi_backend {
    server unix:/opt/homebrew/var/run/php-fpm.sock;
}

server {
    listen 80;
    server_name ${SITE_NAME};
    return 301 https://\$host\$request_uri;
}

server {
    listen 443 ssl http2;
    server_name ${SITE_NAME};

    ssl_certificate /opt/homebrew/etc/ssl/self-signed.crt;
    ssl_certificate_key /opt/homebrew/etc/ssl/self-signed.key;
    ssl_protocols TLSv1.2 TLSv1.3;

    set \$MAGE_ROOT ${MAGE_ROOT};
    set \$MAGE_MODE developer;

    include ${MAGE_ROOT}/nginx.conf.sample;
}
EOF

# Update hosts file
if ! grep -q "${SITE_NAME}" /etc/hosts; then
    echo "Adding ${SITE_NAME} to /etc/hosts..."
    echo "127.0.0.1    ${SITE_NAME}" | sudo tee -a /etc/hosts
fi

# Test and reload Nginx
nginx -t && nginx -s reload

echo "✅ Nginx configuration complete!"
echo "Visit: https://${SITE_NAME}"
```

**Make executable and run:**
```bash
chmod +x ~/setup-nginx-magento.sh
~/setup-nginx-magento.sh
```

---

## Troubleshooting

### Common Issues and Solutions

#### Nginx won't start
```bash
# Check what's using port 80/443
lsof -i :80
lsof -i :443

# Kill processes if needed
sudo kill -9 <PID>
```

#### PHP-FPM not running
```bash
# Start PHP-FPM
brew services start php@8.2

# Check status
brew services list
```

#### MariaDB connection refused
```bash
# Restart MariaDB
brew services restart mariadb@11.4

# Check MariaDB is running
brew services list | grep mariadb

# Check error logs
tail -f /opt/homebrew/var/mysql/*.err
```

#### OpenSearch not responding
```bash
# Check OpenSearch status
curl -X GET "localhost:9200/_cluster/health?pretty"

# Restart OpenSearch
brew services restart opensearch

# Check logs
tail -f /opt/homebrew/var/log/opensearch/*.log
```

#### Redis/Valkey not responding
```bash
# Check Redis status
redis-cli ping

# Restart Redis
brew services restart redis

# Check Redis logs
tail -f /opt/homebrew/var/log/redis.log
```

#### RabbitMQ issues
```bash
# Check RabbitMQ status
rabbitmqctl status

# Restart RabbitMQ
brew services restart rabbitmq

# Check logs
tail -f /opt/homebrew/var/log/rabbitmq/*.log

# Reset RabbitMQ (WARNING: deletes all data)
rabbitmqctl stop_app
rabbitmqctl reset
rabbitmqctl start_app
```

#### Permission issues
```bash
cd ~/Sites/local.magento.test
sudo chmod -R 777 var generated pub/static pub/media app/etc
```

#### Clear caches
```bash
bin/magento cache:flush
bin/magento cache:clean
rm -rf var/cache/* var/page_cache/* generated/*
```

#### SSL certificate errors
```bash
# Remove and recreate certificate
sudo rm /opt/homebrew/etc/ssl/self-signed.*
# Then follow SSL installation steps again
```

---

## Useful Commands

### Magento CLI Commands

```bash
# Enable developer mode
bin/magento deploy:mode:set developer

# Disable maintenance mode
bin/magento maintenance:disable

# Reindex
bin/magento indexer:reindex

# Compile
bin/magento setup:di:compile

# Deploy static content
bin/magento setup:static-content:deploy -f

# Clear cache
bin/magento cache:flush

# Check module status
bin/magento module:status

# Enable/disable modules
bin/magento module:enable Vendor_Module
bin/magento module:disable Vendor_Module
```

### Homebrew Services

```bash
# List all services
brew services list

# Start all required services for Adobe Commerce 2.4.8
brew services start nginx
brew services start php@8.3
brew services start mariadb@11.4
brew services start opensearch
brew services start redis
brew services start rabbitmq

# Start development tools (optional)
brew services start mailhog

# Stop all services
brew services stop nginx
brew services stop php@8.3
brew services stop mariadb@11.4
brew services stop opensearch
brew services stop redis
brew services stop rabbitmq
brew services stop mailhog

# Restart all services
brew services restart nginx
brew services restart php@8.3
brew services restart mariadb@11.4
brew services restart opensearch
brew services restart redis
brew services restart rabbitmq
brew services restart mailhog

# Check status of all services
brew services list | grep -E "nginx|php|mariadb|opensearch|redis|rabbitmq|mailhog"
```

### Start All Services Script

Create a convenient script to start all services:

```bash
nano ~/start-magento-services.sh
```

**Add this content:**
```bash
#!/bin/bash

echo "Starting Adobe Commerce 2.4.8 services..."

# Core services
services=("nginx" "php@8.3" "mariadb@11.4" "opensearch" "redis" "rabbitmq")

# Development services (optional)
dev_services=("mailhog")

echo "Starting core services..."
for service in "${services[@]}"; do
    echo "Starting $service..."
    brew services start "$service"
done

echo ""
echo "Starting development services..."
for service in "${dev_services[@]}"; do
    echo "Starting $service..."
    brew services start "$service" 2>/dev/null || echo "$service not installed (optional)"
done

echo ""
echo "Service status:"
brew services list | grep -E "nginx|php|mariadb|opensearch|redis|rabbitmq|mailhog"

echo ""
echo "Verifying services..."
echo "- Nginx: $(curl -s -o /dev/null -w "%{http_code}" http://localhost:8080)"
echo "- PHP-FPM: $(php-fpm -t 2>&1 | grep -c "successful")"
echo "- MariaDB: $(mysql -u root -e "SELECT 1" 2>&1 | grep -c "1")"
echo "- OpenSearch: $(curl -s http://localhost:9200 | grep -c "opensearch")"
echo "- Redis: $(redis-cli ping)"
echo "- RabbitMQ: $(rabbitmqctl status | grep -c "RabbitMQ")"
echo "- MailHog: $(curl -s -o /dev/null -w "%{http_code}" http://localhost:8025 2>/dev/null || echo "Not running")"

echo ""
echo "All services started!"
echo ""
echo "Development URLs:"
echo "- Adobe Commerce: https://local.magento.test"
echo "- MailHog UI: http://127.0.0.1:8025"
echo "- OpenSearch: http://localhost:9200"
echo "- RabbitMQ UI: http://localhost:15672"
```

**Make executable:**
```bash
chmod +x ~/start-magento-services.sh
```

**Usage:**
```bash
~/start-magento-services.sh
```

### Stop All Services Script

```bash
nano ~/stop-magento-services.sh
```

**Add this content:**
```bash
#!/bin/bash

echo "Stopping Adobe Commerce 2.4.8 services..."

# Core services
services=("nginx" "php@8.3" "mariadb@11.4" "opensearch" "redis" "rabbitmq")

# Development services
dev_services=("mailhog")

for service in "${services[@]}"; do
    echo "Stopping $service..."
    brew services stop "$service"
done

for service in "${dev_services[@]}"; do
    echo "Stopping $service..."
    brew services stop "$service" 2>/dev/null
done

echo ""
echo "All services stopped!"
brew services list
```

**Make executable:**
```bash
chmod +x ~/stop-magento-services.sh
```

---

## Performance Optimization

### PHP Performance Tuning

**Enable and optimize PHP OPcache:**

Edit PHP configuration file:
```bash
nano /opt/homebrew/etc/php/8.3/php.ini
```

**Add/modify OPcache settings:**
```ini
; OPcache
opcache.enable=1
opcache.enable_cli=1
opcache.memory_consumption=512
opcache.interned_strings_buffer=16
opcache.max_accelerated_files=130000
opcache.validate_timestamps=1
opcache.revalidate_freq=2
opcache.save_comments=1
opcache.fast_shutdown=1

; Realpath cache
realpath_cache_size=10M
realpath_cache_ttl=7200
```

**Restart PHP-FPM:**
```bash
brew services restart php@8.3
```

### MariaDB Performance Tuning

**Optimize MariaDB configuration:**

Edit MariaDB config:
```bash
nano /opt/homebrew/etc/my.cnf
```

**Production-ready settings:**
```ini
[mysqld]
# Memory settings
innodb_buffer_pool_size=4G
innodb_log_file_size=512M
innodb_flush_log_at_trx_commit=2
innodb_flush_method=O_DIRECT

# Connection settings
max_connections=500
thread_cache_size=256

# Query cache (disabled for InnoDB)
query_cache_size=0
query_cache_type=0

# Temporary tables
tmp_table_size=512M
max_heap_table_size=512M

# InnoDB optimization
innodb_buffer_pool_instances=4
innodb_read_io_threads=8
innodb_write_io_threads=8
innodb_io_capacity=2000
innodb_io_capacity_max=4000
innodb_lock_wait_timeout=50

# Binary logging (optional)
binlog_format=ROW
binlog_row_image=MINIMAL
```

**Restart MariaDB:**
```bash
brew services restart mariadb@11.4
```

### Redis/Valkey Performance Tuning

**Optimize Redis configuration:**

```bash
nano /opt/homebrew/etc/redis.conf
```

**Performance settings:**
```conf
# Maximum memory
maxmemory 2gb
maxmemory-policy allkeys-lru

# Disable persistence for cache databases (not session)
# save ""
# appendonly no

# Lazy freeing
lazyfree-lazy-eviction yes
lazyfree-lazy-expire yes
lazyfree-lazy-server-del yes

# I/O threads (Redis 6+)
io-threads 4
io-threads-do-reads yes
```

**Restart Redis:**
```bash
brew services restart redis
```

### OpenSearch Performance Tuning

**Optimize heap size:**

```bash
nano /opt/homebrew/etc/opensearch/jvm.options.d/jvm.options
```

```
# Set to 50% of available RAM (max 32GB)
-Xms4g
-Xmx4g

# Garbage collection
-XX:+UseG1GC
-XX:MaxGCPauseMillis=200
-XX:InitiatingHeapOccupancyPercent=45
```

**Restart OpenSearch:**
```bash
brew services restart opensearch
```

### Nginx Performance Tuning

Edit Nginx configuration:
```bash
nano /opt/homebrew/etc/nginx/nginx.conf
```

**Add performance directives:**
```nginx
worker_processes auto;
worker_rlimit_nofile 100000;

events {
    worker_connections 4096;
    use kqueue;
    multi_accept on;
}

http {
    # Caching
    open_file_cache max=10000 inactive=30s;
    open_file_cache_valid 60s;
    open_file_cache_min_uses 2;
    open_file_cache_errors on;

    # Buffers
    client_body_buffer_size 128k;
    client_max_body_size 64m;
    client_header_buffer_size 1k;
    large_client_header_buffers 4 16k;
    output_buffers 1 32k;
    postpone_output 1460;

    # Timeouts
    client_header_timeout 3m;
    client_body_timeout 3m;
    send_timeout 3m;
    keepalive_timeout 65;
    keepalive_requests 100;

    # Gzip
    gzip on;
    gzip_vary on;
    gzip_comp_level 6;
    gzip_min_length 1000;
    gzip_proxied any;
    gzip_disable "msie6";
    
    # TCP optimization
    tcp_nopush on;
    tcp_nodelay on;
    sendfile on;
    sendfile_max_chunk 512k;
}
```

**Restart Nginx:**
```bash
nginx -t && nginx -s reload
```

### Enable Varnish Cache (Optional)

For high-traffic sites:

```bash
# Install Varnish
brew install varnish

# Configure Varnish for Magento
# Edit VCL file
nano /opt/homebrew/etc/varnish/default.vcl

# Start Varnish
brew services start varnish
```

### System-Level Optimizations (macOS)

**Increase file descriptor limits:**

```bash
# Check current limits
ulimit -n

# Increase limit (add to ~/.zshrc)
echo "ulimit -n 65536" >> ~/.zshrc
source ~/.zshrc
```

**Disable App Nap for Terminal:**
```bash
defaults write com.apple.Terminal NSAppSleepDisabled -bool YES
```

### Monitor Performance

**Check service resource usage:**
```bash
# Top processes
top -o cpu

# Memory usage
vm_stat

# Disk I/O
iostat -w 2
```

**Adobe Commerce performance commands:**
```bash
# Check slow logs
tail -f var/log/debug.log

# Enable profiler
bin/magento dev:profiler:enable

# Check cache performance
redis-cli INFO stats | grep hit

# Check OpenSearch performance
curl http://localhost:9200/_nodes/stats?pretty
```

---

## Access Your Site

After completing the setup, access your Adobe Commerce installation:

### Frontend (Store)
- **URL:** [https://local.magento.test](https://local.magento.test)
- **Port:** 443 (HTTPS)

### Admin Panel
- **URL:** [https://local.magento.test/admin](https://local.magento.test/admin)
- **Username:** `admin`
- **Password:** `Admin@123!` (or what you set during installation)

### Service URLs

| Service | URL | Default Credentials |
|---------|-----|---------------------|
| **OpenSearch** | http://localhost:9200 | No auth (local) |
| **Redis** | localhost:6379 | No password (local) |
| **RabbitMQ UI** | http://localhost:15672 | guest / guest |
| **MariaDB** | localhost:3306 | magento / your_password |
| **PHP-FPM** | /opt/homebrew/var/run/php-fpm.sock | - |

### Development Tool URLs

| Tool | URL | Purpose |
|------|-----|---------|
| **MailHog UI** | http://127.0.0.1:8025 | Email testing interface |
| **MailHog SMTP** | localhost:1025 | SMTP server for email capture |
| **Xdebug** | Port 9003 | Debug connection port |

### First-Time Access

**If you can't access the site:**

1. **Check services are running:**
   ```bash
   brew services list
   ```

2. **Verify hosts file:**
   ```bash
   cat /etc/hosts | grep local.magento.test
   ```
   Should show: `127.0.0.1 local.magento.test`

3. **Check Nginx configuration:**
   ```bash
   nginx -t
   nginx -s reload
   ```

4. **Clear Magento cache:**
   ```bash
   cd ~/Sites/local.magento.test
   bin/magento cache:flush
   ```

5. **Check file permissions:**
   ```bash
   chmod -R 777 var generated pub/static pub/media app/etc
   ```

### Test SSL Certificate

If you see SSL warnings:

1. Trust the certificate in macOS Keychain
2. Clear browser cache
3. Access using `https://` (not `http://`)

### Verify Installation

**Check Magento version:**
```bash
cd ~/Sites/local.magento.test
bin/magento --version
```

**Check module status:**
```bash
bin/magento module:status
```

**Run diagnostics:**
```bash
bin/magento setup:diag
```

---

## Additional Resources

### Official Documentation

- **Adobe Commerce 2.4.8 Release Notes:** [Adobe Experience League](https://experienceleague.adobe.com/docs/commerce-operations/release/notes/adobe-commerce/2-4-8.html)
- **Adobe Commerce Developer Guide:** [Experience League - Developer](https://experienceleague.adobe.com/docs/commerce.html)
- **System Requirements:** [Adobe Commerce 2.4.8 Requirements](https://experienceleague.adobe.com/docs/commerce-operations/installation-guide/system-requirements.html)
- **Installation Guide:** [Official Installation Guide](https://experienceleague.adobe.com/docs/commerce-operations/installation-guide/overview.html)

### Component Documentation

- **PHP 8.3 Documentation:** [php.net](https://www.php.net/docs.php)
- **MariaDB 11.4 Documentation:** [mariadb.com/kb](https://mariadb.com/kb/en/)
- **OpenSearch Documentation:** [opensearch.org/docs](https://opensearch.org/docs/latest/)
- **Nginx Documentation:** [nginx.org/en/docs](https://nginx.org/en/docs/)
- **Redis Documentation:** [redis.io/documentation](https://redis.io/documentation)
- **RabbitMQ Documentation:** [rabbitmq.com/documentation.html](https://www.rabbitmq.com/documentation.html)
- **Composer Documentation:** [getcomposer.org/doc](https://getcomposer.org/doc/)

### Community Resources

- **Magento Stack Exchange:** [magento.stackexchange.com](https://magento.stackexchange.com/)
- **Adobe Commerce Community:** [community.magento.com](https://community.magento.com/)
- **Magento GitHub:** [github.com/magento/magento2](https://github.com/magento/magento2)
- **Homebrew Documentation:** [docs.brew.sh](https://docs.brew.sh/)

### Tools and Extensions

- **Magento Marketplace:** [commercemarketplace.adobe.com](https://commercemarketplace.adobe.com/)
- **Mage2.tv (Training):** [mage2.tv](https://www.mage2.tv/)
- **MageComp Extensions:** [magecomp.com](https://magecomp.com/)

### Performance and Monitoring

- **New Relic for Adobe Commerce:** [docs.newrelic.com](https://docs.newrelic.com/docs/infrastructure/host-integrations/host-integrations-list/magento-monitoring-integration/)
- **Blackfire.io (Profiler):** [blackfire.io](https://blackfire.io/)

---

## Version-Specific Notes

### Adobe Commerce 2.4.8-p3 Changes

- ✅ **OpenSearch 3** now required (replaced Elasticsearch)
- ✅ **Valkey 8** support added (Redis fork)
- ✅ **RabbitMQ 4.1** required for message queues
- ✅ **PHP 8.4** support added
- ✅ **MariaDB 11.4** now required
- ✅ **Composer 2.8** required
- ✅ **Nginx 1.28** recommended

### Upgrade Path

If upgrading from previous versions:

**From 2.4.7 to 2.4.8:**
```bash
# Backup first!
composer require magento/product-community-edition=2.4.8-p3 --no-update
composer update
bin/magento setup:upgrade
bin/magento setup:di:compile
bin/magento setup:static-content:deploy -f
bin/magento indexer:reindex
bin/magento cache:flush
```

### Breaking Changes

- Elasticsearch no longer supported (use OpenSearch)
- Minimum PHP version is 8.3
- Removed deprecated payment methods
- Updated security requirements

---

## Alternative Setup Methods

### Using Docker

For containerized development:

```bash
# Adobe Commerce Docker (Cloud Docker)
composer create-project --repository-url=https://repo.magento.com/ \
  magento/magento-cloud-docker .

# Configure and start
./bin/magento-docker up
```

### Using MAMP/XAMPP

Alternative to Homebrew setup:

- **MAMP Pro:** [mamp.info](https://www.mamp.info/)
- **XAMPP:** [apachefriends.org](https://www.apachefriends.org/)

### Using Vagrant

For virtualized development:

```bash
# Magento 2 Vagrant
git clone https://github.com/paliarush/magento2-vagrant-for-developers.git
cd magento2-vagrant-for-developers
vagrant up
```

---

## Development Tools

### Installing Xdebug

Xdebug is a powerful debugging tool for PHP that allows step-by-step debugging, profiling, and code coverage analysis.

#### Why Xdebug?

- ✅ **Step-by-step debugging** - Set breakpoints and inspect variables
- ✅ **Stack traces** - Better error messages with detailed traces
- ✅ **Code profiling** - Identify performance bottlenecks
- ✅ **Code coverage** - Analyze test coverage
- ✅ **Remote debugging** - Debug PHP running on server

#### Step 1: Install Xdebug via Homebrew

```bash
brew install xdebug
```

**For PHP 8.3 specifically:**
```bash
pecl install xdebug
```

#### Step 2: Verify Xdebug Installation

```bash
php -m | grep xdebug
```

**Expected output:**
```
xdebug
```

**Check Xdebug version:**
```bash
php -v
```

Should show something like:
```
PHP 8.3.x (cli) (built: ...)
    with Xdebug v3.3.x, Copyright (c) 2002-2024, by Derick Rethans
```

#### Step 3: Configure Xdebug for Adobe Commerce

Edit your PHP configuration file:

```bash
nano /opt/homebrew/etc/php/8.3/php.ini
```

**Add or modify these Xdebug settings:**

```ini
[Xdebug]
; Load Xdebug extension
zend_extension="xdebug"

; Enable debugging mode
xdebug.mode=debug

; Start debugging automatically
xdebug.start_with_request=yes

; Client configuration
xdebug.client_host=127.0.0.1
xdebug.client_port=9003

; IDE key (optional, for PHPStorm)
xdebug.idekey=PHPSTORM

; Additional useful settings
xdebug.max_nesting_level=256
xdebug.var_display_max_depth=10
xdebug.var_display_max_children=256
xdebug.var_display_max_data=1024

; Step debugging
xdebug.step_debug=1

; Profiler (optional, disable in production)
; xdebug.profiler_enable=0
; xdebug.profiler_output_dir=/tmp/xdebug_profiler
```

**Configuration explained:**
- `xdebug.mode=debug` - Enables debugging features
- `xdebug.start_with_request=yes` - Automatically starts debugging
- `xdebug.client_host` - Where to send debug information (IDE)
- `xdebug.client_port` - Port for IDE communication (9003 is default for Xdebug 3)

#### Step 4: Restart PHP-FPM

Apply the configuration:

```bash
brew services restart php@8.3
```

#### Step 5: Verify Xdebug Configuration

**Check Xdebug status:**
```bash
php -i | grep xdebug
```

**Or create a phpinfo file:**
```bash
echo "<?php phpinfo(); ?>" > ~/Sites/local.magento.test/pub/xdebug-test.php
```

Visit: `https://local.magento.test/xdebug-test.php`

Search for "xdebug" to see all configuration.

**Remove test file after verification:**
```bash
rm ~/Sites/local.magento.test/pub/xdebug-test.php
```

#### Step 6: Configure Your IDE

**For VS Code:**

Install the **PHP Debug** extension, then add to `.vscode/launch.json`:

```json
{
    "version": "0.2.0",
    "configurations": [
        {
            "name": "Listen for Xdebug",
            "type": "php",
            "request": "launch",
            "port": 9003,
            "pathMappings": {
                "/Users/yourusername/Sites/local.magento.test": "${workspaceFolder}"
            }
        }
    ]
}
```

**For PHPStorm:**

1. Go to **Preferences** → **PHP** → **Debug**
2. Set Xdebug port to `9003`
3. Enable "Can accept external connections"
4. Go to **Preferences** → **PHP** → **Servers**
5. Add new server:
   - Name: `local.magento.test`
   - Host: `local.magento.test`
   - Port: `443`
   - Debugger: `Xdebug`
   - Use path mappings: Map project root to server path

#### Step 7: Test Xdebug

**Create a test file:**
```bash
nano ~/Sites/local.magento.test/pub/test-xdebug.php
```

**Add this code:**
```php
<?php
$message = "Testing Xdebug";
$numbers = [1, 2, 3, 4, 5];

foreach ($numbers as $num) {
    echo $num . "\n"; // Set breakpoint here
}

echo $message;
```

**Test debugging:**
1. Set a breakpoint on the `echo $num` line in your IDE
2. Start listening for debug connections in your IDE
3. Visit: `https://local.magento.test/test-xdebug.php`
4. Your IDE should stop at the breakpoint

**Clean up:**
```bash
rm ~/Sites/local.magento.test/pub/test-xdebug.php
```

#### Xdebug Performance Notes

**⚠️ Warning:** Xdebug significantly slows down PHP execution.

**For development, use conditional enabling:**

Edit `php.ini`:
```ini
; Disable by default
xdebug.mode=off
xdebug.start_with_request=trigger

; Enable only when XDEBUG_TRIGGER cookie/param is set
```

**Enable via browser extension:**
- Chrome: [Xdebug Helper](https://chrome.google.com/webstore/detail/xdebug-helper)
- Firefox: [Xdebug Helper](https://addons.mozilla.org/en-US/firefox/addon/xdebug-helper-for-firefox/)

**Or enable via CLI:**
```bash
# For specific command only
XDEBUG_MODE=debug php bin/magento cache:flush
```

#### Troubleshooting Xdebug

**Xdebug not loading:**
```bash
# Check if extension exists
ls -la /opt/homebrew/lib/php/pecl/*/xdebug.so

# Verify php.ini path
php --ini

# Check for syntax errors
php -m
```

**IDE not connecting:**
```bash
# Check firewall
sudo /usr/libexec/ApplicationFirewall/socketfilterfw --list

# Test port
lsof -i :9003

# Enable Xdebug log
echo "xdebug.log=/tmp/xdebug.log" >> /opt/homebrew/etc/php/8.3/php.ini
tail -f /tmp/xdebug.log
```

**Xdebug slowing down site:**
```bash
# Disable Xdebug temporarily
mv /opt/homebrew/etc/php/8.3/conf.d/ext-xdebug.ini /opt/homebrew/etc/php/8.3/conf.d/ext-xdebug.ini.disabled
brew services restart php@8.3

# Re-enable when needed
mv /opt/homebrew/etc/php/8.3/conf.d/ext-xdebug.ini.disabled /opt/homebrew/etc/php/8.3/conf.d/ext-xdebug.ini
brew services restart php@8.3
```

---

### Installing MailHog

MailHog is an email testing tool that captures outgoing emails during development, preventing accidental email sends.

#### Why MailHog?

- ✅ **Email testing** - Test email functionality without sending real emails
- ✅ **Web UI** - View captured emails in browser
- ✅ **SMTP server** - Acts as local mail server
- ✅ **No configuration** - Works out of the box
- ✅ **Safe development** - No accidental customer emails

#### Step 1: Install MailHog

```bash
brew update && brew install mailhog
```

**What gets installed:**
- MailHog SMTP server (port 1025)
- MailHog web UI (port 8025)
- mhsendmail binary for PHP integration

#### Step 2: Start MailHog Service

```bash
brew services start mailhog
```

**Verify MailHog is running:**
```bash
brew services list | grep mailhog
```

Expected output: `mailhog  started`

**Check MailHog is listening:**
```bash
lsof -i :8025  # Web UI
lsof -i :1025  # SMTP server
```

#### Step 3: Install mhsendmail (If Not Working)

If MailHog doesn't work after installation, install Go and mhsendmail:

```bash
# Install Go (if not already installed)
brew install go

# Install mhsendmail
go install github.com/mailhog/mhsendmail@latest
```

**Copy binaries to accessible location:**

```bash
# Find your Go bin directory
echo $GOPATH  # Usually ~/go

# Copy MailHog binary
sudo cp ~/go/bin/MailHog /usr/local/bin/mailhog

# Copy mhsendmail binary
sudo cp ~/go/bin/mhsendmail /usr/local/bin/mhsendmail

# Make executable
sudo chmod +x /usr/local/bin/mailhog
sudo chmod +x /usr/local/bin/mhsendmail
```

#### Step 4: Configure PHP to Use MailHog

Edit your PHP configuration:

```bash
nano /opt/homebrew/etc/php/8.3/php.ini
```

**Find and update the sendmail_path:**

```ini
[mail function]
; For PHP 8.3
sendmail_path = /Users/YOURUSERNAME/go/bin/mhsendmail

; Or if using system path
; sendmail_path = /usr/local/bin/mhsendmail

; SMTP settings (optional)
SMTP = localhost
smtp_port = 1025
```

**⚠️ Important:** Replace `YOURUSERNAME` with your actual macOS username:

```bash
# Find your username
whoami

# Update php.ini with correct path
CURRENT_USER=$(whoami)
sed -i '' "s|sendmail_path.*|sendmail_path = /Users/${CURRENT_USER}/go/bin/mhsendmail|g" /opt/homebrew/etc/php/8.3/php.ini
```

#### Step 5: Restart PHP-FPM

Apply the configuration:

```bash
brew services restart php@8.3
```

#### Step 6: Test MailHog

**Option 1: Simple PHP test script**

Create a test file:
```bash
nano ~/Sites/local.magento.test/pub/test-mail.php
```

**Add this code:**
```php
<?php
$to = 'test@test.com';
$subject = 'MailHog Test Email';
$message = 'This is a test email from Adobe Commerce local development.';
$headers = 'From: noreply@local.magento.test' . "\r\n" .
           'Reply-To: noreply@local.magento.test' . "\r\n" .
           'X-Mailer: PHP/' . phpversion();

if (mail($to, $subject, $message, $headers)) {
    echo "Email sent successfully! Check MailHog at http://127.0.0.1:8025/";
} else {
    echo "Email failed to send.";
}
```

**Visit:** `https://local.magento.test/test-mail.php`

**Clean up:**
```bash
rm ~/Sites/local.magento.test/pub/test-mail.php
```

**Option 2: Test via Adobe Commerce**

```bash
cd ~/Sites/local.magento.test

# Test email via Magento CLI
bin/magento dev:email test@test.com
```

#### Step 7: Access MailHog Web UI

Open your browser and visit: **http://127.0.0.1:8025/**

**MailHog Web UI features:**
- 📧 View all captured emails
- 🔍 Search emails by subject, from, to
- 📎 View email source and attachments
- 🗑️ Delete individual or all emails
- 🔄 Real-time updates

#### Configure Adobe Commerce to Use MailHog

**Method 1: Via Admin Panel**

1. Login to Admin: `https://local.magento.test/admin`
2. Go to **Stores** → **Configuration** → **Advanced** → **System**
3. Expand **Mail Sending Settings**
4. Set **Disable Email Communications** to **No**
5. Set **Transport** to **SMTP**
6. Configure:
   - Host: `localhost`
   - Port: `1025`
7. Save configuration

**Method 2: Via env.php**

Edit configuration:
```bash
nano ~/Sites/local.magento.test/app/etc/env.php
```

Add this section:
```php
'system' => [
    'default' => [
        'system' => [
            'smtp' => [
                'disable' => false,
                'host' => 'localhost',
                'port' => '1025'
            ]
        ]
    ]
],
```

**Method 3: Via CLI**

```bash
bin/magento config:set system/smtp/disable 0
bin/magento config:set system/smtp/host localhost
bin/magento config:set system/smtp/port 1025
bin/magento cache:flush
```

#### Testing Email Features

**Test customer registration email:**
```bash
# Create test customer
bin/magento customer:hash:upgrade
```

**Test order confirmation:**
1. Place a test order on frontend
2. Check MailHog for order confirmation email

**Test newsletter subscription:**
1. Subscribe to newsletter on frontend
2. Check MailHog for confirmation email

#### MailHog Configuration Options

**Custom ports (if conflicts exist):**

```bash
# Stop default MailHog
brew services stop mailhog

# Start with custom ports
mailhog -smtp-bind-addr 127.0.0.1:1026 -ui-bind-addr 127.0.0.1:8026 &
```

**Update PHP sendmail_path for custom SMTP port:**
```ini
sendmail_path = "/usr/local/bin/mhsendmail --smtp-addr=127.0.0.1:1026"
```

#### Troubleshooting MailHog

**MailHog not starting:**
```bash
# Check if ports are in use
lsof -i :1025
lsof -i :8025

# Kill conflicting process
sudo kill -9 <PID>

# Restart MailHog
brew services restart mailhog
```

**Emails not being captured:**
```bash
# Verify PHP configuration
php -i | grep sendmail_path

# Should output something like:
# sendmail_path => /Users/username/go/bin/mhsendmail

# Test mhsendmail directly
echo "Test email" | /Users/username/go/bin/mhsendmail test@test.com
```

**Web UI not accessible:**
```bash
# Check MailHog is running
ps aux | grep mailhog

# Check logs
brew services info mailhog

# Restart
brew services restart mailhog
```

**Permission denied errors:**
```bash
# Fix mhsendmail permissions
chmod +x ~/go/bin/mhsendmail
sudo chmod +x /usr/local/bin/mhsendmail
```

#### MailHog Management Commands

```bash
# Start MailHog
brew services start mailhog

# Stop MailHog
brew services stop mailhog

# Restart MailHog
brew services restart mailhog

# Check status
brew services list | grep mailhog

# View MailHog help
mailhog --help

# Run MailHog in foreground (for debugging)
mailhog
```

#### MailHog API

MailHog provides a REST API for automation:

```bash
# Get all messages
curl http://127.0.0.1:8025/api/v2/messages

# Delete all messages
curl -X DELETE http://127.0.0.1:8025/api/v1/messages

# Get specific message
curl http://127.0.0.1:8025/api/v1/messages/<message-id>
```

---

### Recommended IDE Extensions

**VS Code:**
- PHP Intelephense
- Magento 2 Snippets
- PHP Debug (for Xdebug)
- Xdebug Helper

**PHPStorm:**
- Magento 2 Plugin
- Composer Support
- Database Tools
- .env files support

### CLI Tools

```bash
# Magento CLI enhancements
composer require --dev magento/magento-coding-standard
composer require --dev magento/magento2-functional-testing-framework

# Code quality tools
composer require --dev phpstan/phpstan
composer require --dev squizlabs/php_codesniffer
```

---

## License

This guide is provided as-is for educational purposes.

## Contributing

Feel free to suggest improvements or report issues with this guide.

## Support and Contributions

### Getting Help

If you encounter issues:

1. **Check logs:** Look in `/opt/homebrew/var/log/` for service logs
2. **Search Stack Exchange:** Most common issues have solutions
3. **Check Adobe DevDocs:** Official troubleshooting guides
4. **Community Forums:** Adobe Commerce community is active

### Report Issues

If you find issues with this guide, please report them.

### Contribute

Suggestions and improvements are welcome!

---

- 
