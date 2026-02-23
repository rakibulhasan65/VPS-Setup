# VPS Multi-Project Deployment Guide

Production-ready setup guide for running multiple projects (Laravel, Django, FastAPI, React, Redmine) on Ubuntu VPS.

Ubuntu 22.04 / 24.04 compatible.

---

# 1. Check Which Process Is Using Port 3000

```bash
ss -ltnp | grep :3000
```

Output example:

```bash
LISTEN 0 511 0.0.0.0:3000 users:(("node",pid=12345,fd=21))
```

---

# 2. Kill the Process Using Port 3000

```bash
kill -9 12345
```

Replace `12345` with the actual PID from the previous command.

Verify port is free:

```bash
ss -ltnp | grep :3000
```

---

# 3. Run Redmine on Port 3000 (Production Mode)

```bash
cd /opt/redmine
bundle exec rails server -e production -b 0.0.0.0 -p 3000
```

Check if running locally:

```bash
curl http://127.0.0.1:3000
```

If HTML response appears, server is running correctly.

---

# 4. Open Firewall Port 3000

If using firewalld:

```bash
firewall-cmd --add-port=3000/tcp --permanent
firewall-cmd --reload
```

Verify:

```bash
firewall-cmd --list-ports
```

If using UFW (Ubuntu default):

```bash
sudo ufw allow 3000/tcp
sudo ufw reload
sudo ufw status
```

---

# 5. Find Correct Django Project Name

Method 1:

```bash
ls -d */ | grep wsgi
```

Method 2 (Recommended):

```bash
find . -name wsgi.py
```

Example result:

```bash
./attendance/wsgi.py
```

Project name is: `attendance`

---

# 6. Run Django with Gunicorn (Production)

Foreground run (test mode):

```bash
gunicorn attendance.wsgi:application --bind 0.0.0.0:8001
```

Replace `attendance` with your project name.

---

# 7. Run Gunicorn in Background (Temporary)

```bash
nohup gunicorn attendance.wsgi:application --bind 0.0.0.0:8001 > gunicorn.log 2>&1 &
```

Check process:

```bash
ps aux | grep gunicorn
```

---

# 8. Recommended Production Setup (Best Practice)

Do NOT expose application servers directly.

Recommended architecture:

Nginx (Port 80 / 443)
    → Laravel (PHP-FPM socket)
    → Django (127.0.0.1:8001 via Gunicorn)
    → FastAPI (127.0.0.1:8002 via Gunicorn)
    → React (static build)
    → Redmine (127.0.0.1:3000 reverse proxy)

Only Nginx should be publicly accessible.

---

# 9. Quick Diagnostics Commands

Check running services:

```bash
ss -tulpn
```

Check CPU usage:

```bash
top
```

Check memory:

```bash
free -h
```

Check server load:

```bash
uptime
```

---

# 10. Important Production Rules

1. Do NOT use development servers in production.
2. Always use Gunicorn for Django/FastAPI.
3. Use PHP-FPM for Laravel.
4. Use Nginx as reverse proxy.
5. Use Supervisor or systemd for auto restart.
6. Keep all internal services bound to 127.0.0.1.

---

Deployment Guide Ready for GitHub.

Suggested filename:

MULTI_PROJECT_VPS_DEPLOYMENT_GUIDE.md

