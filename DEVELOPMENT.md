# Development Guide

> Force Sensitivity Detector - ICS v4.7.20 Extension  
> **Version**: 1.0.0 | **Last Updated**: January 18, 2026

This guide covers setting up a local development environment for the Force Sensitivity Detector extension.

---

## Prerequisites

Before setting up your development environment, ensure you have:

### Required Software

| Software | Version | Notes |
|----------|---------|-------|
| PHP | 7.4+ (8.0+ recommended) | CLI and web server |
| MySQL | 5.7+ | Or MariaDB 10.2+ |
| Composer | 2.x | PHP dependency manager |
| Git | Any recent version | Version control |
| Web Server | Apache/Nginx | Or Laravel Valet, DDEV, etc. |

### Optional but Recommended

| Software | Purpose |
|----------|---------|
| PHPStorm / VS Code | IDE with PHP support |
| Xdebug | PHP debugging |
| Sequel Ace / TablePlus | Database management |
| Postman / Insomnia | API testing |

---

## Local IPS Installation Options

You need a working Invision Community Suite v4.7.20 installation. Choose one method:

### Option 1: Local IPS Installation (Recommended)

1. **Obtain ICS**
   - Purchase or trial from [Invision Community](https://invisioncommunity.com)
   - Download the v4.7.20 release package

2. **Set Up Web Server**
   ```bash
   # Using Laravel Valet (macOS)
   brew install php@8.0
   composer global require laravel/valet
   valet install
   
   # Create project directory
   mkdir -p ~/Sites/ips-local
   cd ~/Sites/ips-local
   
   # Install IPS (follow official docs)
   # Extract IPS package contents here
   ```

3. **Create Database**
   ```bash
   mysql -u root -p
   CREATE DATABASE ips_local CHARACTER SET utf8mb4 COLLATE utf8mb4_unicode_ci;
   CREATE USER 'ips_dev'@'localhost' IDENTIFIED BY 'your_password';
   GRANT ALL PRIVILEGES ON ips_local.* TO 'ips_dev'@'localhost';
   FLUSH PRIVILEGES;
   ```

4. **Run ICS Installer**
   - Navigate to `http://ips-local.test` (or your valet domain)
   - Follow the installation wizard
   - Note your admin credentials

### Option 2: Docker / DDEV

```bash
# Clone IPS into a directory
mkdir ips-ddev && cd ips-ddev

# Create DDEV config
ddev config --project-type=php --php-version=8.0

# Add MySQL configuration
echo "MYSQL_ROOT_PASSWORD=root
MYSQL_DATABASE=ips_local
MYSQL_USER=ips_dev
MYSQL_PASSWORD=ips_password" > .env

# Start environment
ddev start

# Mount your IPS files and proceed with installation
```

### Option 3: MAMP / XAMPP

1. Install MAMP or XAMPP
2. Configure PHP version (8.0+ recommended)
3. Set up MySQL and create a database
4. Point document root to IPS directory
5. Access via `http://localhost:8888` (or configured port)

---

## Environment Configuration

### PHP Configuration

Create or edit `php.ini`:
```ini
; Memory and execution
memory_limit = 256M
max_execution_time = 300

; Error reporting (development)
display_errors = On
error_reporting = E_ALL

; Extensions needed
extension=mysqli
extension=pdo_mysql
extension=curl
extension=gd
extension=mbstring
extension=openssl
extension=json
extension=filter
extension=zip

; For Xdebug (development)
xdebug.mode = debug
xdebug.start_with_request = yes
xdebug.client_host = 127.0.0.1
```

### IPS Configuration

After installation, edit `conf_global.php`:
```php
<?php

$INFO = array(
    // ... existing config ...
    
    // Debug mode for development
    'debug' => true,
    'use_dev_toolbar' => true,
    
    // Log settings
    'log_level' => 1, // 0=none, 1=errors, 2=all
    'log_sql_queries' => true,
);
```

### Developer Mode

Enable developer mode in IPS for helpful debugging:

1. Edit `conf_global.php`:
```php
$INFO['dev_mode'] = true;
```

2. Or via AdminCP:
```
AdminCP → System → Overview → Developer Settings → Enable Developer Mode
```

---

## Installing the Extension

### Method 1: Symlink (Recommended for Development)

This allows you to work on the extension code without reinstalling:

```bash
# Navigate to your IPS applications directory
cd /path/to/your/ips/application

# Create symlink to your development copy
ln -s /path/to/ICS-ext-forcesensitivity/forcesensitivity forcesensitivity

# Verify
ls -la forcesensitivity
```

### Method 2: Direct Clone

```bash
# Clone into IPS applications directory
cd /path/to/your/ips/application
git clone https://github.com/jimstratus/ICS-ext-forcesensitivity.git forcesensitivity
```

### Method 3: Package Installation (Testing Production)

1. Build the package:
```bash
cd /path/to/ICS-ext-forcesensitivity
composer install --no-dev
tar -czvf forcesensitivity.tar.gz forcesensitivity/
```

2. Install via AdminCP:
```
AdminCP → System → Applications & Plugins → Install → Upload Package
```

### Post-Installation Steps

1. **Clear Caches**
   ```
   AdminCP → System → Cache Management → Clear All Caches
   ```

2. **Verify Hook Status**
   ```
   AdminCP → System → Plugins → Hooks
   ```
   Ensure `forcesensitivity_hook_memberCreate` is active.

3. **Configure Settings**
   ```
   AdminCP → Community → Force Sensitivity → Settings
   ```

4. **Test Detection**
   - Create a test account
   - Check if Force Sensitivity is assigned
   - Review audit logs

---

## Development Workflow

### 1. Enable Developer Mode

```php
// In conf_global.php
$INFO['dev_mode'] = true;
$INFO['debug'] = true;
```

### 2. Configure Settings

Access the extension settings:
```
AdminCP → Community → Force Sensitivity → Settings
```

Recommended development settings:
- **Base Probability**: 50% (for easier testing)
- **Detection Enabled**: ON
- **Ratio Enforcement**: None (simpler testing)
- **Badge Style**: Simple (faster loading)

### 3. Make Changes

```bash
# Create a feature branch
cd /path/to/ICS-ext-forcesensitivity
git checkout -b feature/my-new-feature

# Edit files using your preferred editor
# PHPStorm: phpstorm .
# VS Code: code .
```

### 4. Test Changes

#### Clear Cache After Changes
```
AdminCP → System → Cache Management → Clear All Caches
```

#### Testing Registration Detection
1. Create a new test user in a private browser window
2. Check user profile for Force Sensitivity status
3. Review audit logs

#### Testing Admin Features
1. Navigate to AdminCP → Community → Force Sensitivity
2. Test each module (Settings, Members, Modifiers, Logs)
3. Verify database changes

### 5. Debugging

#### Using Xdebug

1. Configure your IDE to listen for Xdebug connections
2. Set breakpoints in PHP files
3. Trigger the code path in your browser
4. Inspect variables and step through code

#### Logging

Enable SQL logging in `conf_global.php`:
```php
$INFO['log_sql_queries'] = true;
```

View logs in `datastore/logs/` directory.

#### Common Issues

| Issue | Solution |
|-------|----------|
| Changes not appearing | Clear cache, check hook is active |
| Database errors | Verify database credentials in `conf_global.php` |
| Permission errors | Check file permissions on extension directory |
| Hook not firing | Verify hook is enabled in AdminCP → Plugins → Hooks |

---

## Testing

### Running Tests

```bash
cd /path/to/ICS-ext-forcesensitivity

# Install dependencies
composer install

# Run PHPUnit tests
./vendor/bin/phpunit

# Run specific test file
./vendor/bin/phpunit tests/DetectorTest.php

# Run with coverage
./vendor/bin/phpunit --coverage-html coverage/
```

### Writing Tests

Place tests in the `tests/` directory:
```php
<?php

namespace IPS\forcesensitivity\tests;

use PHPUnit\Framework\TestCase;

class DetectorTest extends TestCase
{
    public function testProbabilityCalculation(): void
    {
        // Your test code here
    }
}
```

See [CONTRIBUTING.md](./CONTRIBUTING.md#testing) for more testing guidelines.

---

## Code Standards

Follow the coding standards outlined in [CONTRIBUTING.md](./CONTRIBUTING.md#coding-standards):

- **PHP**: PSR-12, strict types, PHPDoc comments
- **JavaScript**: ES6+, ICS patterns
- **CSS**: BEM naming, `fs-` prefix, theme support

---

## Database Management

### Accessing the Database

```bash
mysql -u ips_dev -p ips_local
```

### Useful Commands

```sql
-- View Force Sensitivity status table
SELECT * FROM forcesensitivity_status LIMIT 10;

-- View audit log
SELECT * FROM forcesensitivity_log ORDER BY timestamp DESC;

-- View modifiers
SELECT * FROM forcesensitivity_modifiers WHERE is_active = 1;

-- Count Force Sensitive users
SELECT 
    COUNT(*) as total,
    SUM(is_force_sensitive) as sensitive,
    AVG(is_force_sensitive) * 100 as ratio
FROM forcesensitivity_status;
```

### Resetting Test Data

```sql
-- Clear all Force Sensitivity data (keeps extension settings)
DELETE FROM forcesensitivity_log;
DELETE FROM forcesensitivity_status;
DELETE FROM forcesensitivity_modifiers;

-- Or reset everything including settings
TRUNCATE forcesensitivity_log;
TRUNCATE forcesensitivity_status;
TRYNCATE forcesensitivity_modifiers;
```

---

## IDE Setup

### PHPStorm

1. Open project at repository root
2. Mark `sources/` as Sources root
3. Configure PHP 8.0+ interpreter
4. Enable Xdebug
5. Set PSR-12 code style in Preferences → PHP → Code Style

### VS Code

Recommended extensions:
- PHP Intelephense
- PHP Debug
- PHP CS Fixer
- Prettier

Create `.vscode/settings.json`:
```json
{
    "php.suggest.basic": false,
    "php.validate.executablePath": "/usr/local/bin/php",
    "intelephense.environment.phpVersion": "8.0",
    "editor.formatOnSave": true,
    "[php]": {
        "editor.defaultFormatter": "junstyle.php-cs-fixer"
    }
}
```

---

## Resources

- [Invision Community Documentation](https://invisioncommunity.com/documentation)
- [IPS Developer Portal](https://invisioncommunity.com/developers)
- [PSR-12 Coding Style](https://www.php-fig.org/psr/psr-12/)
- [PHPUnit Documentation](https://phpunit.readthedocs.io/)

---

## Troubleshooting

### Extension Not Appearing

1. Clear all caches
2. Verify file structure is correct
3. Check PHP error logs
4. Ensure database tables were created

### Hook Not Firing

1. Check hook is enabled in AdminCP
2. Verify hook file is in correct location
3. Check PHP error logs for hook errors

### Database Issues

1. Verify database credentials
2. Check table prefix matches
3. Ensure MySQL user has correct permissions

---

*For contribution guidelines, see [CONTRIBUTING.md](./CONTRIBUTING.md)*
