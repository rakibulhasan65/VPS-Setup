# FastAPI VPS Deployment Guide (Production Ready)

## 📌 Overview

এই গাইডে দেখানো হয়েছে কিভাবে VPS-এ FastAPI প্রজেক্ট deploy করতে হয়।

### Included:
- Server setup
- Python & venv setup
- FastAPI run with uvicorn
- nohup দিয়ে background run
- Process check & kill
- systemd service (production best practice)
- Nginx reverse proxy
- SSL setup
- Firewall setup

---

## 🖥️ 1️⃣ VPS Initial Setup

SSH দিয়ে সার্ভারে ঢুকুন:
```
ssh root@your_server_ip
```

Update করুন:
```
apt update && apt upgrade -y
```

---

## 🐍 2️⃣ Python & Virtual Environment Setup

Install Python & venv:
```
apt install python3 python3-pip python3-venv -y
```

Project folder তৈরি করুন:
```
mkdir fastapi-app
cd fastapi-app
```

Virtual environment তৈরি করুন:
```
python3 -m venv venv
source venv/bin/activate
```

Dependencies install করুন:
```
pip install fastapi uvicorn
```

যদি requirements.txt থাকে:
```
pip install -r requirements.txt
```

---

## ▶️ 3️⃣ FastAPI Run (Basic)

One time run:
```
uvicorn main:app --host 0.0.0.0 --port 8002
```

⚠️ সমস্যা: Terminal বন্ধ করলে server বন্ধ হয়ে যাবে।

---

## 🔄 4️⃣ Background Run (nohup Method)

Terminal বন্ধ করলেও server চালু রাখতে:
```
nohup uvicorn main:app --host 0.0.0.0 --port 8002 &
```

✔️ Terminal বন্ধ করলেও server চলবে
✔️ Log যাবে nohup.out ফাইলে

Log check করতে:
```
tail -f nohup.out
```

---

## 🔍 5️⃣ Server Running Check

```
ps aux | grep uvicorn
```

Output থেকে PID দেখুন।

---

## 🛑 6️⃣ Stop Server

Kill process:
```
kill -9 PID
```

Example:
```
kill -9 12345
```

---

## ⭐ 7️⃣ Production Recommended Method (systemd Service)

nohup ভালো কিন্তু production-এর জন্য best না।

Create service file:
```
nano /etc/systemd/system/fastapi.service
```

Paste করুন:
```
[Unit]
Description=FastAPI App
After=network.target

[Service]
User=root
WorkingDirectory=/root/fastapi-app
ExecStart=/root/fastapi-app/venv/bin/uvicorn main:app --host 0.0.0.0 --port 8002
Restart=always

[Install]
WantedBy=multi-user.target
```

Reload:
```
systemctl daemon-reload
```

Start:
```
systemctl start fastapi
```

Enable on boot:
```
systemctl enable fastapi
```

Status check:
```
systemctl status fastapi
```

Stop:
```
systemctl stop fastapi
```

---

## 🌍 8️⃣ Nginx Reverse Proxy Setup

Install nginx:
```
apt install nginx -y
```

Config create করুন:
```
nano /etc/nginx/sites-available/fastapi
```

Add:
```
server {
    listen 80;
    server_name your_domain.com;

    location / {
        proxy_pass http://127.0.0.1:8002;
        proxy_set_header Host $host;
        proxy_set_header X-Real-IP $remote_addr;
    }
}
```

Enable site:
```
ln -s /etc/nginx/sites-available/fastapi /etc/nginx/sites-enabled
```

Restart nginx:
```
systemctl restart nginx
```

---

## 🔐 9️⃣ SSL Setup (Let's Encrypt)

Install certbot:
```
apt install certbot python3-certbot-nginx -y
```

Run:
```
certbot --nginx -d your_domain.com
```

Auto renew check:
```
certbot renew --dry-run
```

---

## 🔥 🔟 Firewall Setup

Allow ports:
```
ufw allow 22
ufw allow 80
ufw allow 443
ufw enable
```

Check status:
```
ufw status
```

---

## 📂 1️⃣1️⃣ GitHub Deployment Method

Project clone করতে:
```
git clone https://github.com/username/repo.git
cd repo
```

Update করতে:
```
git pull
```

Restart service:
```
systemctl restart fastapi
```

---

## 🧠 Bonus Tips

Production-এ uvicorn না, gunicorn ব্যবহার ভালো:
```
pip install gunicorn
```

Run:
```
gunicorn -w 4 -k uvicorn.workers.UvicornWorker main:app
```

---

## 🧾 Common Commands Summary

| কাজ | Command |
|---|---|
| Run server | uvicorn main:app --host 0.0.0.0 --port 8002 |
| Background run | nohup uvicorn main:app --host 0.0.0.0 --port 8002 & |
| Check process | ps aux |
| Kill process | kill -9 PID |
| Service status | systemctl status fastapi |
| Restart service | systemctl restart fastapi |

---

## 🎯 Final Recommendation

✅ Development → uvicorn
✅ Small VPS → nohup
✅ Production → systemd + Nginx + SSL
✅ High Traffic → Gunicorn + Nginx

