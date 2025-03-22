
# Part 1: How to Deploy a Vite + React App on a VPS with Nginx (Ubuntu Server)

In this ultimate step-by-step guide, you'll learn how to deploy a **Vite-powered React.js app** on a VPS running **Ubuntu Linux** using the **Nginx web server**. We’ll not only cover deployment but also deep-dive into all common issues, errors, and how to fix them.

---

## ✨ Prerequisites
Before you begin, make sure you have the following:
- A VPS (e.g. from Hostinger, DigitalOcean, or any provider)
- Ubuntu 20.04 or 22.04 installed on your VPS
- Root or sudo access to the server
- A registered domain name (we used `trashsettechnology.com`)
- A Vite + React app ready for deployment

---

## 🔐 Step 1: Connect to Your VPS via SSH
Use **PuTTY** on Windows or Terminal on macOS/Linux:
```bash
ssh root@your-server-ip
```
If you changed your default port or user, make sure to use:
```bash
ssh your-user@your-server-ip -p your-port
```

---

## 🌐 Step 2: Install Nginx
Update packages and install Nginx:
```bash
apt update && apt upgrade -y
apt install nginx -y
```

Enable and start the Nginx service:
```bash
systemctl enable nginx
systemctl start nginx
```

Check if it's running:
```bash
systemctl status nginx
```

Navigate to `http://your-server-ip` — you should see the default Nginx welcome page.

---

## 🛠️ Step 3: Install Node.js, npm, and Git
```bash
apt install nodejs npm git -y
node -v
npm -v
git --version
```

---

## 📁 Step 4: Clone Your React + Vite Project
```bash
cd ~
git clone https://github.com/your-username/your-repo.git
cd your-repo
```

---

## 📦 Step 5: Install Dependencies and Build the App
```bash
npm install
npm run build
ls -la dist
```

---

## 🧹 Step 6: Clean Nginx Web Root and Deploy
```bash
rm -rf /var/www/html/*
cp -r dist/* /var/www/html
ls -la /var/www/html
```

---

## ⚙️ Step 7: Configure Nginx to Serve React App
```bash
nano /etc/nginx/sites-available/frontend.trashsettechnology.com
```
Paste:
```nginx
server {
    listen 80;
    server_name trashsettechnology.com www.trashsettechnology.com;

    root /var/www/html;
    index index.html;

    location / {
        try_files $uri /index.html;
    }
}
```
```bash
ln -s /etc/nginx/sites-available/frontend.trashsettechnology.com /etc/nginx/sites-enabled/
nginx -t
systemctl reload nginx
```

---

## ⚠️ Issue: Seeing "This is a homepage"
Check file:
```bash
cat /var/www/html/index.html
```
Fix:
```bash
rm -rf /var/www/html/*
cp -r dist/* /var/www/html
systemctl reload nginx
```

---

# Part 2: SSL, Domain, and Troubleshooting for Production Deployment

---

## 🌍 Step 8: Add DNS Records
| Type | Name | Value          |
|------|------|----------------|
| A    | @    | 95.216.235.114 |
| A    | www  | 95.216.235.114 |

Check:
```bash
dig +short trashsettechnology.com
```

---

## 🔒 Step 9: Install SSL with Certbot
```bash
apt install certbot python3-certbot-nginx -y
certbot --nginx -d trashsettechnology.com -d www.trashsettechnology.com
```

---

## 🚨 Error: NXDOMAIN
If this happens:
```text
NXDOMAIN looking up A for www.trashsettechnology.com
```
Fix by adding DNS A record for `www`, then:
```bash
certbot --nginx -d trashsettechnology.com -d www.trashsettechnology.com
```

---

## 🔁 Optional: Add SSL later for www
```bash
certbot --nginx -d trashsettechnology.com
certbot --expand -d trashsettechnology.com -d www.trashsettechnology.com
```

---

## ✅ Final Checklist
- [x] Built app with `npm run build`
- [x] Deployed to `/var/www/html`
- [x] Nginx config tested
- [x] DNS records correct
- [x] SSL installed with Certbot
- [x] Accessible at https://trashsettechnology.com

---

## 🏁 Conclusion
You’ve deployed a production-ready Vite + React app on a VPS with Nginx and SSL! Now scale up with multi-app hosting, subdomains, or Docker. Happy deploying 🚀
