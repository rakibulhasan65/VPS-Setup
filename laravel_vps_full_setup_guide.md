#  Laravel Full VPS Setup Guide (PHP + Composer + MySQL + Nginx + Supervisor)

Complete production-ready deployment guide for Laravel project.
Ubuntu 22.04 / 24.04 compatible.

---

#  1. Server Initial Update

```bash
sudo apt update && sudo apt upgrade -y
```

---

#  2. Install Required Packages

```bash
sudo apt install software-properties-common curl git unzip -y
```

---

#  3. Install PHP (Example: PHP 8.3)

```bash
sudo add-apt-repository ppa:ondrej/php -y
sudo apt update
sudo apt install php8.3 php8.3-cli php8.3-common php8.3-mysql php8.3-zip php8.3-gd php8.3-mbstring php8.3-curl php8.3-xml php8.3-bcmath php8.3-fpm -y
```

Check Version:

```bash
php -v
```

Check MySQL Extension:

```bash
php -m | grep -i mysql
```

Output should contain:

```
mysqli
mysqlnd
pdo_mysql
```

---

#  4. Install Composer

```bash
cd ~
curl -sS https://getcomposer.org/installer -o composer-setup.php
sudo php composer-setup.php --install-dir=/usr/local/bin --filename=composer
```

Check:

```bash
composer -V
```

---

#  5. Install MySQL Server

```bash
sudo apt install mysql-server -y
sudo mysql_secure_installation
```

Login:

```bash
sudo mysql
```

Create Database & User:

```sql
CREATE DATABASE emrdtracking;
CREATE USER 'emrdtracking'@'localhost' IDENTIFIED BY 'your_password';
GRANT ALL PRIVILEGES ON emrdtracking.* TO 'emrdtracking'@'localhost';
FLUSH PRIVILEGES;
EXIT;
```

---

#  6. Clone Laravel Project

```bash
cd /var/www
sudo git clone https://github.com/your-username/your-project.git
cd your-project
```

Set Permission:

```bash
sudo chown -R www-data:www-data /var/www/your-project
sudo chmod -R 755 /var/www/your-project
```

---

#  7. Install Laravel Dependencies

```bash
composer install --no-dev --optimize-autoloader
```

Copy .env:

```bash
cp .env.example .env
```

Generate Key:

```bash
php artisan key:generate
```

Update .env:

```
APP_ENV=production
APP_DEBUG=false

DB_DATABASE=emrdtracking
DB_USERNAME=emrdtracking
DB_PASSWORD=your_password

CACHE_DRIVER=database
QUEUE_CONNECTION=database
```

---

#  8. Create Cache & Queue Tables

```bash
php artisan cache:table
php artisan queue:table
php artisan migrate --force
```

Optimize:

```bash
php artisan config:cache
php artisan route:cache
php artisan view:cache
```

---

#  9. Install Nginx

```bash
sudo apt install nginx -y
```

Create Config:

```bash
sudo nano /etc/nginx/sites-available/laravel
```

Paste:

```nginx
server {
    listen 80;
    server_name your_domain.com;
    root /var/www/your-project/public;

    index index.php index.html;

    location / {
        try_files $uri $uri/ /index.php?$query_string;
    }

    location ~ \.php$ {
        include snippets/fastcgi-php.conf;
        fastcgi_pass unix:/var/run/php/php8.3-fpm.sock;
    }

    location ~ /\.ht {
        deny all;
    }
}
```

Enable Site:

```bash
sudo ln -s /etc/nginx/sites-available/laravel /etc/nginx/sites-enabled/
sudo nginx -t
sudo systemctl restart nginx
```

---

#  10. Setup Supervisor (Queue Worker)

Install:

```bash
sudo apt install supervisor -y
```

Create Config:

```bash
sudo nano /etc/supervisor/conf.d/laravel-worker.conf
```

Paste:

```ini
[program:laravel-worker]
process_name=%(program_name)s_%(process_num)02d
command=php /var/www/your-project/artisan queue:work --sleep=3 --tries=3 --timeout=90
autostart=true
autorestart=true
user=www-data
numprocs=1
redirect_stderr=true
stdout_logfile=/var/www/your-project/worker.log
```

Reload:

```bash
sudo supervisorctl reread
sudo supervisorctl update
sudo supervisorctl start laravel-worker:*
```

Check:

```bash
sudo supervisorctl status
```

---

#  11. (Optional) Run Development Mode

Not recommended for production.

```bash
php artisan serve --host=0.0.0.0 --port=8002
```

Background Mode:

```bash
nohup php artisan serve --host=0.0.0.0 --port=8002 > /dev/null 2>&1 &
```

Stop:

```bash
pkill -f "artisan serve"
```

---

#  12. Final Production Checklist

 PHP Installed
 Composer Installed
 MySQL Database Created
 .env Configured
 Migration Completed
 Cache & Queue Tables Created
 Nginx Configured
 PHP-FPM Running
 Supervisor Running
 Permissions Correct

---

#  Production Ready Laravel VPS Deployment Complete

You can now push this file to GitHub as:

```
LARAVEL_VPS_DEPLOYMENT_GUIDE.md
```

---

If needed, you can also convert this into a bash auto-deployment script.

