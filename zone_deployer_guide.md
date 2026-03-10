# Laravel Deployment to zone.ee using Deployer.org

## Pre-Deployment Setup

### 1. SSH Key Configuration
- [ ] Generate SSH key pair (if not already created)
  ```bash
  ssh-keygen -t ed25519 -C "your-email@example.com"
  ```
- [ ] Add SSH key to SSH agent
  ```bash
  eval "$(ssh-agent -s)"
  ssh-add ~/.ssh/id_ed25519
  ```
- [ ] Copy public key to zone.ee account
  ```bash
  cat ~/.ssh/id_ed25519.pub
  ```
- [ ] Log into zone.ee control panel → SSH Keys
- [ ] Add public key to authorized keys
- [ ] Test SSH connection to zone.ee
  ```bash
  ssh virt_xxxx@www.domain.ee
  ```
- [ ] Verify you can SSH without password prompt

### 2. Firewall & Access Configuration
- [ ] Log into zone.ee control panel
- [ ] Navigate to SSH/IP whitelisting
- [ ] **Option A: Add Current IP**
  - [ ] Find your public IP: https://whatismyipaddress.com
  - [ ] Add your IP to whitelist
  - [ ] Verify SSH access from your IP works
- [ ] **Option B: Allow Access Anywhere**
  - [ ] Configure firewall rules to allow SSH from any IP

### 2.5 Domain/Subdomain Linking (CRITICAL - Must Be Done First)
- [ ] Log into zone.ee control panel
- [ ] Navigate to **Domains** or **Domain Management** section
- [ ] **Option A: Using Existing Domain**
  - [ ] Select your domain from the list
  - [ ] Verify the directory path points to `~/domeenid/{{www_domain.ee}}/{{root_folder}}`
  - [ ] **Common path for zone.ee:**
    - [ ] `~/domeenid/example.ee/folder` (for domain root)
- [ ] **Option B: Creating New Subdomain**
  - [ ] Click "Add Subdomain"
  - [ ] Enter subdomain name (e.g., "app" for app.example.ee)
  - [ ] Set directory to `~/domeenid/{{www_domain.ee}}/{{root_folder}}`
  - [ ] Apply/Save configuration
- [ ] **CRITICAL: Path Configuration for Initial Deployment**
  - [ ] **FOR INITIAL DEPLOYMENT ONLY:** Point domain to deployment root directory
    - [ ] Set path to: `~/domeenid/{{www_domain.ee}}/{{root_folder}}`
    - [ ] This must be an **empty directory** for deployment to succeed
    - [ ] Do NOT point to subdirectory yet
  - [ ] **AFTER FIRST SUCCESSFUL DEPLOYMENT:** Update path to public directory
    - [ ] After deployment completes, change path to: `~/domeenid/{{www_domain.ee}}/{{root_folder}}/current/public`
    - [ ] This points to the Laravel public directory where the application is served from
    - [ ] This prevents access to sensitive files (`.env`, source code, etc.)
  - [ ] **Why two steps?**
    - [ ] Deployer needs empty directory to create `releases/`, `shared/`, and `current/` symlink
    - [ ] Zone.ee web server needs to point to `public/` directory after deployment for security
- [ ] **Verify Domain/Subdomain Linking**
  - [ ] Wait 5-10 minutes for DNS propagation (if DNS changed)
  - [ ] Test ping to domain: `ping www.domain.ee`
  - [ ] Access domain in browser: `http://www.domain.ee` (should show zone.ee default page initially)
  - [ ] Verify no 404 or "domain not linked" errors
- [ ] **Important:** Without proper domain linking, your deployed application will NOT be accessible via the domain
- [ ] Note: zone.ee typically uses the `domeenid/` directory structure for domain organization

### 3. Local Environment Setup
- [ ] Verify Deployer is installed
  ```bash
  composer require --dev deployer/deployer
  ```
- [ ] Or install globally
  ```bash
  composer global require deployer/deployer
  ```
- [ ] Verify Deployer version
  ```bash
  dep --version
  ```
- [ ] **Alternative: If `dep` command not found** (symlink missing)
  ```bash
  vendor/bin/dep --version
  ```
- [ ] **If using vendor/bin path**, create alias in your shell (optional)
  ```bash
  alias dep='vendor/bin/dep'
  ```
- [ ] Ensure you have Git installed and configured
  ```bash
  git config --global user.name "Your Name"
  git config --global user.email "your-email@example.com"
  ```

### 4. GitHub Access Configuration
- [ ] Generate GitHub SSH key or use existing
- [ ] Add GitHub SSH key to your GitHub account
- [ ] Test GitHub SSH connection
  ```bash
  ssh -T git@github.com
  ```
- [ ] Verify you can clone your repository
  ```bash
  git clone git@github.com:{{github_user}}/{{project_name}}.git
  ```

---

## deploy.yaml Configuration

### 5. Update Deployer Configuration File
- [ ] Create/locate `deploy.yaml` in project root
- [ ] Replace placeholder variables:
  - [ ] `{{project_name}}` → your actual project name (e.g., "myapp")
  - [ ] `{{github_user}}` → your GitHub username
  - [ ] `{{www_domain.ee}}` → your actual domain (e.g., "example.ee")
  - [ ] `{{virt_xxxx}}` → your zone.ee virtual user (e.g., "virt_12345")
  - [ ] `{{root_folder}}` → target folder (e.g., "public_html" or "")
- [ ] Verify hostname format matches zone.ee server address
- [ ] Confirm `deploy_path` points to correct location on zone.ee
- [ ] Save configuration file

### 6. Verify Deploy Path Structure
- [ ] SSH into zone.ee server
  ```bash
  ssh virt_xxxx@www.domain.ee
  ```
- [ ] Navigate to deploy location
  ```bash
  cd ~/domeenid/{{www_domain.ee}}/{{root_folder}}
  ```
- [ ] Check directory exists and is writable
  ```bash
  ls -la
  pwd
  ```
- [ ] If directory doesn't exist, create it
  ```bash
  mkdir -p ~/domeenid/{{www_domain.ee}}/{{root_folder}}
  ```
- [ ] Ensure you have write permissions
- [ ] Note: zone.ee typically uses `domeenid/` structure

---

## Environment Configuration

### 7. Zone.ee Database Setup (CRITICAL - Do Before .env Configuration)
- [ ] Log into zone.ee control panel (https://zone.ee/et/dashboard)
- [ ] Navigate to **Databases** section
- [ ] **Option A: Create New Database**
  - [ ] Click "Add Database" or similar button
  - [ ] Enter database name (e.g., `project_db`)
  - [ ] Select database type: **MySQL**
  - [ ] Create database user:
    - [ ] Username (e.g., `project_user`)
    - [ ] Password (use strong password, save in password manager)
  - [ ] Confirm user has full permissions on the database
  - [ ] Save/Create database
  - [ ] **Note down these credentials:**
    ```
    Database Name: ___________________
    Database User: ___________________
    Database Password: ___________________
    Database Host: zone specific host
    ```
- [ ] **Option B: Use Existing Database**
  - [ ] Select existing database from list
  - [ ] Verify existing user credentials
  - [ ] Add new user if needed for this project
- [ ] **If Having Connection Issues:**
  - [ ] Check that user has proper permissions
  - [ ] Verify database exists in control panel

### 7.5 Initial Deployment Configuration (CRITICAL)
- [ ] Review `deploy:` task in `deploy.yaml/php`
- [ ] **FOR INITIAL DEPLOYMENT ONLY** - Comment out database-dependent tasks:
  ```php
  tasks:
    deploy:
      - 'deploy:prepare'
      - 'deploy:vendors'
      - 'npm:production'
      - 'artisan:storage:link'
      - 'artisan:optimize:clear'
      # - 'artisan:migrate'              // COMMENT OUT FOR INITIAL DEPLOY
      # - 'artisan:optimize'             // COMMENT OUT FOR INITIAL DEPLOY
      - 'deploy:publish'
  ```
- [ ] **Reason:** Laravel cannot run artisan commands without `.env` file
- [ ] Save the modified `deploy.yaml`

### 8. Create .env File on Server
- [ ] SSH into zone.ee server
  ```bash
  ssh virt_xxxx@www.domain.ee
  ```
- [ ] Navigate to release directory (check latest release)
  ```bash
  ls -la ~/domeenid/{{www_domain.ee}}/{{root_folder}}/releases/
  cd ~/domeenid/{{www_domain.ee}}/{{root_folder}}/releases/[latest-release]/
  ```
- [ ] Copy `.env.example` to `.env`
  ```bash
  cp .env.example .env
  ```
- [ ] Open `.env` with vim
  ```bash
  vim .env
  ```
- [ ] **Configure these critical variables:**
  - [ ] `APP_NAME` → Your application name
  - [ ] `APP_ENV` → `production`
  - [ ] `APP_DEBUG` → `false` (CRITICAL for production)
  - [ ] `APP_URL` → `https://www.domain.ee`
  - [ ] `APP_KEY` → Generate via `php artisan key:generate` (or use existing)

- [ ] **ZONE.EE DATABASE CONFIGURATION (from step 7):**
  - [ ] `DB_CONNECTION` → `mysql`
  - [ ] `DB_HOST` → `zone db host`
  - [ ] `DB_PORT` → `3306` (standard MySQL port)
  - [ ] `DB_DATABASE` → [Database name from zone.ee setup]
  - [ ] `DB_USERNAME` → [Database user from zone.ee setup]
  - [ ] `DB_PASSWORD` → [Database password from zone.ee setup]
  - [ ] **Example configuration:**
    ```
    DB_CONNECTION=mysql
    DB_HOST=localhost
    DB_PORT=3306
    DB_DATABASE=project_db
    DB_USERNAME=project_user
    DB_PASSWORD=your_secure_password_here
    ```
  - [ ] **Verify zone.ee credentials match what you created in step 7**
  - [ ] If credentials don't work, double-check in zone.ee control panel → Databases

- [ ] **Additional important variables:**
  - [ ] `MAIL_DRIVER` → Configure email settings
  - [ ] `MAIL_HOST`, `MAIL_PORT`, `MAIL_USERNAME`, `MAIL_PASSWORD`
  - [ ] Check zone.ee documentation for mail server details
  - [ ] `CACHE_DRIVER` → `file` (recommended for shared hosting)
  - [ ] `SESSION_DRIVER` → `file` or `cookie`
  - [ ] `QUEUE_CONNECTION` → `sync` (unless PM2 is configured)

- [ ] Save and exit vim (`:wq`)
- [ ] Verify `.env` was created
  ```bash
  cat .env | head -20
  ```
- [ ] **Test database connection (optional but recommended):**
  ```bash
  php artisan tinker
  >>> DB::connection()->getPdo();
  ```
  - [ ] If successful, you'll see connection info
  - [ ] If error, verify `.env` database credentials match zone.ee setup

---


### 9. Run Database Migrations
- [ ] After initial deployment and `.env` configuration:
  - [ ] SSH into server
  - [ ] Navigate to current symlink
    ```bash
    cd ~/domeenid/{{www_domain.ee}}/{{root_folder}}/current
    ```
  - [ ] Run migrations manually
    ```bash
    php artisan migrate
    ```
  - [ ] Verify no migration errors
- [ ] Or uncomment `artisan:migrate` in `deploy.yaml` for future deployments

---

## Application Configuration

### 10. Node.js / npm Configuration
- [ ] Verify `package.json` exists in project root
- [ ] Verify `package-lock.json` is committed to Git
- [ ] Confirm Vite is configured correctly
  - [ ] Check `vite.config.js` exists
  - [ ] Verify `npm run build` builds assets locally
- [ ] Test locally first
  ```bash
  npm ci
  npm run build
  ```

### 11. Laravel/Inertia/Vue Configuration
- [ ] Verify Inertia.js is installed
  ```bash
  composer show | grep inertia
  npm list @inertiajs/vue3
  ```
- [ ] Check `config/inertia.php` exists and is configured
- [ ] Verify Vue 3 is installed in `package.json`
- [ ] Check `resources/js/app.js` imports Inertia properly
- [ ] Test locally
  ```bash
  npm run dev
  ```

### 12. PHP Configuration on zone.ee
- [ ] Verify PHP 8.4 is available (as noted in `opcache:clear` task)
  ```bash
  ssh virt_xxxx@www.domain.ee
  php84 --version
  ```
- [ ] If using different PHP version, update `deploy.yaml`:
  ```php
  opcache:clear:
    - run: killall php82-cgi || true  // Change to correct version
  ```
- [ ] **Change default PHP version via CLI** (if needed)
  - [ ] Refer to zone.ee documentation: https://www.zone.ee/help/kb/konsoolis-php-vaikeversiooni-muutmine/
  - [ ] SSH into zone.ee
    ```bash
    ssh virt_xxxx@www.domain.ee
    ```
  - [ ] List available PHP versions
    ```bash
    php --version
    ```
  - [ ] Change default PHP version (follow zone.ee guide)
  - [ ] Verify change was applied
    ```bash
    php --version
    ```
- [ ] Verify required PHP extensions are installed:
  - [ ] `php-mbstring`
  - [ ] `php-openssl`
  - [ ] `php-pdo`
  - [ ] `php-mysql` (if using MySQL)
  - [ ] `php-curl`
  - [ ] `php-gd` (if image handling needed)
- [ ] Check in zone.ee control panel or contact support

---

## Pre-Deployment Testing

### 13. Local Testing
- [ ] Run all tests locally
  ```bash
  php artisan test
  npm run test
  ```
- [ ] Build assets locally to verify no errors
  ```bash
  npm run build
  ```
- [ ] Clear any local artifacts
  ```bash
  rm -rf node_modules
  rm -rf bootstrap/cache/*
  rm -rf storage/logs/*
  ```
- [ ] Commit all changes to Git
  ```bash
  git add .
  git commit -m "Pre-deployment configuration"
  git push origin main
  ```

### 14. Verify Git Repository
- [ ] Ensure code is pushed to GitHub
  ```bash
  git push origin main
  ```
- [ ] Verify latest commit is on GitHub
- [ ] Test clone on different machine (if possible)
- [ ] Verify all files are tracked (no missing files)

### 15. Deployer Configuration Validation
- [ ] Validate `deploy.yaml` syntax
  ```bash
  php -l deploy.yaml
  ```
- [ ] Run deployer in dry-run mode (if available)
  ```bash
  dep deploy stage --dry-run
  ```
- [ ] Check for any configuration warnings
- [ ] Review all variable substitutions in output

---

## Initial Deployment Execution

### 16. First Deployment
- [ ] Execute initial deployment (with database tasks commented out)
  ```bash
  dep deploy stage
  ```
- [ ] Monitor output for errors
- [ ] Key checkpoints to watch:
  - [ ] Repository cloning completes
  - [ ] `composer install` succeeds
  - [ ] `npm ci` and `npm run build` succeed
  - [ ] Storage link is created
  - [ ] Deploy succeeds without database connection errors

### 17. Verify Initial Deployment
- [ ] SSH into zone.ee
  ```bash
  ssh virt_xxxx@www.domain.ee
  ```
- [ ] Navigate to deployment directory
  ```bash
  cd ~/domeenid/{{www_domain.ee}}/{{root_folder}}/current
  ```
- [ ] Verify key files exist:
  - [ ] `.env` file
  - [ ] `public/` directory
  - [ ] `storage/` directory with correct permissions
  - [ ] `vendor/` directory
  - [ ] Built assets in `public/build/` (or similar)
- [ ] Check file permissions
  ```bash
  ls -la
  ls -la storage/
  ```
- [ ] Verify symlinks
  ```bash
  ls -la ~/domeenid/{{www_domain.ee}}/{{root_folder}}/
  ```

### 17.5 UPDATE DOMAIN PATH IN ZONE.EE (CRITICAL AFTER FIRST DEPLOYMENT)
- [ ] **This step must be done after first successful deployment**
- [ ] Log into zone.ee control panel
- [ ] Navigate to **Domains** → Your domain
- [ ] Edit domain/subdomain configuration
- [ ] **Change path from:**
  ```
  ~/domeenid/{{www_domain.ee}}/{{root_folder}}
  ```
- [ ] **Change path to:**
  ```
  ~/domeenid/{{www_domain.ee}}/{{root_folder}}/current/public
  ```
- [ ] Save/Apply changes
- [ ] **Why?** The `public/` directory is the web root where Laravel serves files from
- [ ] This prevents direct access to sensitive files (`.env`, source code, etc.)
- [ ] Test domain in browser: `https://www.domain.ee`
- [ ] Verify application loads correctly
- [ ] Check that assets load properly
- [ ] If blank page or 500 error, verify path is correct

### 18. Database Setup Post-Deployment
- [ ] SSH into server
  ```bash
  ssh virt_xxxx@www.domain.ee
  cd ~/domeenid/{{www_domain.ee}}/{{root_folder}}/current
  ```
- [ ] Run migrations
  ```bash
  php artisan migrate
  ```
- [ ] Seed database if needed
  ```bash
  php artisan db:seed
  ```
- [ ] Generate application key if needed
  ```bash
  php artisan key:generate
  ```

### 19. Test Web Application
- [ ] Open application in browser
  ```
  https://www.domain.ee
  ```
- [ ] Check for errors (check logs if issues occur)
  ```bash
  tail -f storage/logs/laravel.log
  ```
- [ ] Test basic functionality:
  - [ ] Homepage loads
  - [ ] Assets load (CSS, JavaScript, images)
  - [ ] Database queries work
  - [ ] Form submissions work
  - [ ] Inertia/Vue components render correctly

---

## Post-Deployment Configuration

### 20. Update deploy.yaml for Future Deployments
- [ ] Uncomment database-dependent tasks
  ```php
  tasks:
    deploy:
      - 'deploy:prepare'
      - 'deploy:vendors'
      - 'npm:production'
      - 'artisan:storage:link'
      - 'artisan:optimize:clear'
      - 'artisan:migrate'              
      - 'artisan:optimize'           
      - 'deploy:publish'
  ```
- [ ] If using PM2 for queue workers, uncomment:
  ```php
  after:
    deploy:success: [opcache:clear, pm2:restart]
  ```
- [ ] Commit updated `deploy.yaml`
  ```bash
  git add deploy.yaml
  git commit -m "Enable database tasks for production"
  git push origin main
  ```

### 21. Configure Queue Workers (if needed)
- [ ] If using queue jobs, set up PM2
  ```bash
  ssh virt_xxxx@www.domain.ee
  npm install -g pm2
  ```
- [ ] Create PM2 configuration file
  ```bash
  cat > ecosystem.config.js << 'EOF'
  module.exports = {
    apps: [{
      name: 'laravel-queue-worker',
      script: './artisan',
      args: 'queue:work',
      exec_mode: 'cluster',
      instances: 1,
      autorestart: true,
      watch: false,
      env: {
        APP_ENV: 'production',
        NODE_ENV: 'production'
      }
    }]
  };
  EOF
  ```
- [ ] Start worker with PM2
  ```bash
  pm2 start ecosystem.config.js
  pm2 save
  ```
- [ ] Verify worker is running
  ```bash
  pm2 list
  ```

### 22. SSL/HTTPS Configuration
- [ ] Verify SSL certificate is installed (zone.ee should provide)
- [ ] Test HTTPS access
  ```
  https://www.domain.ee
  ```
- [ ] Verify SSL certificate is valid (no browser warnings)
- [ ] Update `APP_URL` in `.env` to use HTTPS
  ```
  APP_URL=https://www.domain.ee
  ```
- [ ] Clear application cache
  ```bash
  php artisan cache:clear
  php artisan config:clear
  ```

### 23. Configure Email (if needed)
- [ ] Check zone.ee for mail server details
- [ ] Update `.env` mail configuration
  ```
  MAIL_DRIVER=smtp
  MAIL_HOST=smtp.zone.ee
  MAIL_PORT=587
  MAIL_USERNAME=your-email@domain.ee
  MAIL_PASSWORD=xxxx
  MAIL_ENCRYPTION=tls
  MAIL_FROM_ADDRESS=your-email@domain.ee
  ```
- [ ] Test email sending
  ```bash
  php artisan tinker
  >>> Mail::raw('Test', fn($msg) => $msg->to('test@example.com'));
  ```

---

## Performance & Optimization

### 24. Caching and Optimization
- [ ] Clear all caches on server
  ```bash
  ssh virt_xxxx@www.domain.ee
  cd ~/domeenid/{{www_domain.ee}}/{{root_folder}}/current
  php artisan cache:clear
  php artisan route:cache
  php artisan config:cache
  php artisan view:cache
  ```
- [ ] Verify OPcache is being cleared on deployment
  - [ ] Check `before: deploy:success: opcache:clear` in `deploy.yaml`
  - [ ] Monitor that `killall php84-cgi` executes successfully
- [ ] Test application responsiveness
- [ ] Monitor logs for errors
  ```bash
  tail -f storage/logs/laravel.log
  ```

### 25. Logging and Monitoring
- [ ] Verify logs are being written
  ```bash
  ls -la storage/logs/
  tail -20 storage/logs/laravel.log
  ```
- [ ] Check storage directory has correct permissions
  ```bash
  chmod -R 775 storage/
  chmod -R 775 bootstrap/cache/
  ```
- [ ] Set up log rotation (optional)
  ```bash
  # zone.ee may have logrotate configured
  ```
- [ ] Test error logging by triggering an error
- [ ] Monitor application for errors in production

---

## Subsequent Deployments

### 26. Deployment Workflow for Updates
- [ ] Make changes locally
  ```bash
  git add .
  git commit -m "Feature/fix description"
  git push origin main
  ```
- [ ] Pull latest on test machine to verify
- [ ] Deploy to production
  ```bash
  dep deploy stage
  ```
- [ ] Monitor deployment output
- [ ] Verify application loads and works
- [ ] Check logs for any warnings

### 27. Rollback Procedures
- [ ] If deployment fails, Deployer keeps releases
  ```bash
  dep rollback stage
  ```
- [ ] Verify `keep_releases: 1` in `deploy.yaml` (adjust as needed)
- [ ] Test rollback procedure before first deployment
  ```bash
  dep deploy stage
  dep rollback stage
  ```
- [ ] Verify previous version is restored

---

## Troubleshooting Checklist

### Common Issues & Solutions

#### Deployer Command Not Found
- [ ] If `dep` command returns "command not found":
  ```bash
  # Use vendor/bin path instead
  vendor/bin/dep --version
  vendor/bin/dep deploy stage
  ```
- [ ] Verify Deployer is installed via Composer
  ```bash
  ls -la vendor/bin/dep
  ```
- [ ] If vendor directory is missing, reinstall dependencies
  ```bash
  composer install
  ```
- [ ] Create shell alias for convenience
  ```bash
  alias dep='vendor/bin/dep'
  ```
- [ ] Or add to your `.bashrc` or `.zshrc` permanently
  ```bash
  echo "alias dep='vendor/bin/dep'" >> ~/.bashrc
  source ~/.bashrc
  ```

#### SSH Connection Failed
- [ ] Verify IP is whitelisted in zone.ee firewall
- [ ] Check SSH key permissions: `chmod 600 ~/.ssh/id_ed25519`
- [ ] Test SSH directly: `ssh virt_xxxx@www.domain.ee`
- [ ] Verify SSH key is added to ssh-agent
- [ ] Check zone.ee system for SSH issues (contact support)

#### Database Connection Error
- [ ] Verify `.env` database credentials match zone.ee setup (from step 7)
  - [ ] Compare `DB_DATABASE`, `DB_USERNAME`, `DB_PASSWORD` with zone.ee control panel
  - [ ] Ensure database exists in zone.ee control panel
  - [ ] Verify database user has permissions on that database
- [ ] Test connection from server using PHP tinker:
  ```bash
  cd ~/domeenid/{{www_domain.ee}}/{{root_folder}}/current
  php artisan tinker
  >>> DB::connection()->getPdo();
  ```
  - [ ] If successful, connection info will display
  - [ ] If error, check `.env` credentials against zone.ee panel
- [ ] Alternative: Test with MySQL client (if available)
  ```bash
  mysql -h localhost -u dbuser -p dbname
  ```
- [ ] Verify database host is `localhost` (zone.ee uses local database)
- [ ] Check database user has correct permissions in zone.ee control panel
- [ ] Verify database was actually created in zone.ee (not just the user)
- [ ] Contact zone.ee support if credentials don't work despite correct configuration

#### PHP/Artisan Command Errors
- [ ] Verify correct PHP version is being used
  ```bash
  php --version
  ```
- [ ] If wrong version is default, change it via CLI
  - [ ] Reference zone.ee guide: https://www.zone.ee/help/kb/konsoolis-php-vaikeversiooni-muutmine/
  - [ ] SSH into server and follow guide to change PHP version
- [ ] Check for missing PHP extensions
  ```bash
  php -m | grep mbstring
  ```
- [ ] Test PHP directly:
  ```bash
  php --version
  php -m | grep mbstring
  ```
- [ ] Check file permissions on artisan file
- [ ] Review full error message in logs

#### Assets Not Loading / Vite Build Fails
- [ ] Verify `npm ci` completed successfully
- [ ] Check for build errors in deployment output
- [ ] Verify `public/build/` directory exists
- [ ] Check manifest file exists:
  ```bash
  ls -la public/build/manifest.json
  ```
- [ ] Review npm log for detailed errors

#### OPcache Issues
- [ ] Verify PHP version in `opcache:clear` task matches actual version
- [ ] Test manually:
  ```bash
  killall php84-cgi
  ```
- [ ] Check that PHP-FPM restarts automatically
- [ ] Monitor application after clearing cache

#### Permission Denied Errors
- [ ] Check directory ownership
  ```bash
  ls -la ~/domeenid/{{www_domain.ee}}/
  ```
- [ ] Verify http_user has correct permissions
- [ ] Check storage and bootstrap cache permissions
  ```bash
  chmod -R 775 storage/ bootstrap/cache/
  ```

#### Deployment Fails - "Directory Not Empty" or Similar Error
- [ ] This occurs when the deploy path already contains files
- [ ] **Solution for initial deployment:**
  - [ ] SSH into zone.ee
  - [ ] Navigate to deploy directory
    ```bash
    cd ~/domeenid/{{www_domain.ee}}/{{root_folder}}
    ls -la
    ```
  - [ ] If directory has files (other than `.htaccess` or `index.html`), it's not empty
  - [ ] Back up any critical files if needed
  - [ ] Remove all files from the directory
    ```bash
    rm -rf *
    ```
  - [ ] Verify directory is empty
    ```bash
    ls -la
    ```
  - [ ] Retry deployment
    ```bash
    dep deploy stage
    ```
- [ ] **Prevention:**
  - [ ] Ensure domain path points to empty directory before first deployment
  - [ ] After deployment succeeds, update domain path to `{{root_folder}}/current/public`

#### Domain Shows Blank Page or 404 After Deployment
- [ ] **Check if domain path was updated to `/current/public`**
  - [ ] Log into zone.ee control panel
  - [ ] Verify domain path is: `~/domeenid/{{www_domain.ee}}/{{root_folder}}/current/public`
  - [ ] If still pointing to root, update it
  - [ ] Allow 5-10 minutes for changes to take effect
- [ ] Check that `/current/public` directory exists
  ```bash
  ssh virt_xxxx@www.domain.ee
  ls -la ~/domeenid/{{www_domain.ee}}/{{root_folder}}/current/
  ls -la ~/domeenid/{{www_domain.ee}}/{{root_folder}}/current/public/
  ```
- [ ] Verify `.env` file exists in parent directory (not public)
  ```bash
  ls -la ~/domeenid/{{www_domain.ee}}/{{root_folder}}/current/.env
  ```
- [ ] Check Laravel logs for errors
  ```bash
  tail -f ~/domeenid/{{www_domain.ee}}/{{root_folder}}/current/storage/logs/laravel.log
  ```

#### Deployment Freezes or Hangs
- [ ] Check server resources (disk space, RAM)
- [ ] Review any long-running tasks in deploy.yaml
- [ ] Verify network connectivity
- [ ] Check zone.ee server status
- [ ] Try canceling with Ctrl+C and retrying

---

## Security Checklist

### Security Configuration
- [ ] Verify `APP_DEBUG=false` in production `.env`
- [ ] Use strong, random `APP_KEY`
- [ ] Enable HTTPS/SSL (check zone.ee configuration)
- [ ] Restrict file permissions appropriately
- [ ] Verify `.env` file is not world-readable
  ```bash
  chmod 600 .env
  ```
- [ ] Ensure sensitive files are in `.gitignore`
- [ ] Regular security updates:
  ```bash
  composer update --security-only
  npm audit fix
  ```

## Final Verification Checklist

### 33. Before Marking as Complete
- [ ] SSH access from current IP works
- [ ] Deployer configuration is valid
- [ ] Database credentials are secure and working
- [ ] Application loads in browser without errors
- [ ] Assets (CSS, JS, images) load correctly
- [ ] Database operations work (read/write)
- [ ] Inertia/Vue components render correctly
- [ ] Forms submit successfully
- [ ] Email sending works (if applicable)
- [ ] Logs are being written correctly
- [ ] SSL/HTTPS certificate is valid
- [ ] All sensitive data is environment-protected
- [ ] Backup strategy is in place
- [ ] Team members have deployment instructions
- [ ] Documentation is up to date

---

## Useful Commands Reference

```bash
# SSH into zone.ee
ssh virt_xxxx@www.domain.ee

# Navigate to deployment directory
cd ~/domeenid/{{www_domain.ee}}/{{root_folder}}

# Check current symlink
ls -la current

# View recent releases
ls -la releases/

# Check Laravel status
cd current
php artisan status
php artisan migrate:status

# Clear all caches
php artisan cache:clear
php artisan config:clear
php artisan route:cache

# View logs
tail -f storage/logs/laravel.log

# Deploy locally (use one of these)
dep deploy stage
# OR if symlink missing
vendor/bin/dep deploy stage

# Rollback
dep deploy stage
# OR
vendor/bin/dep rollback stage

# List deployments
dep releases stage
# OR
vendor/bin/dep releases stage
```

---
