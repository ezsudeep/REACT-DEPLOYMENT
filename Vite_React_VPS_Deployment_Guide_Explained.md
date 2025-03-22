
# 🚀 Ultimate Guide: Deploying a Vite + React App on a VPS with Nginx (Ubuntu Server)

This guide helps you **deploy a modern React app built with Vite** to a Virtual Private Server (VPS) using **Ubuntu** and **Nginx** as the web server. We'll also secure your site with HTTPS using **Let's Encrypt SSL**. Every step includes **commands**, **why you're doing it**, and **how to fix common errors**.

---

## ✨ Prerequisites

Before starting, ensure you have:
- A VPS (e.g., Hostinger, DigitalOcean, Vultr)
- Ubuntu 20.04 or 22.04 installed on the VPS
- Root or sudo access
- A domain name (e.g., `trashsettechnology.com`)
- A React app using [Vite](https://vitejs.dev)

---

## 🔐 Step 1: Connect to Your VPS

Open Terminal (macOS/Linux) or use **PuTTY** (on Windows) and connect:

```bash
ssh root@your-server-ip
```

If your server uses a custom user or SSH port:

```bash
ssh your-user@your-server-ip -p your-port
```

**Why?**  
This gives you remote access to manage your VPS.

---

## 🌐 Step 2: Install Nginx

Nginx is a lightweight and high-performance web server.

```bash
apt update && apt upgrade -y
apt install nginx -y
```

Enable and start it:

```bash
systemctl enable nginx
systemctl start nginx
```

Check status:

```bash
systemctl status nginx
```

**Test:** Open your server IP in a browser (`http://your-server-ip`). You should see Nginx's default page.

---

## 🛠 Step 3: Install Node.js, npm, and Git

We need these to build and fetch your project.

```bash
apt install nodejs npm git -y
node -v
npm -v
git --version
```

---

## 📁 Step 4: Clone Your Project

Go to your home directory, then clone your GitHub repo:

```bash
cd ~
git clone https://github.com/your-username/your-repo.git
cd your-repo
```

---

## 📦 Step 5: Install Dependencies and Build

Install dependencies:

```bash
npm install
```

Build your project:

```bash
npm run build
```

This creates a `dist/` folder with production-ready files.

Check it:

```bash
ls -la dist
```

---

## 🧹 Step 6: Deploy the Build to Web Root

Clear out the Nginx root directory:

```bash
rm -rf /var/www/html/*
```

Copy the built project to it:

```bash
cp -r dist/* /var/www/html
```

Verify:

```bash
ls -la /var/www/html
```

You should see `index.html` and `assets/`.

---

## ⚙️ Step 7: Configure Nginx for React App

Create a new site config:

```bash
nano /etc/nginx/sites-available/frontend.trashsettechnology.com
```

Paste this:

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

**Why `try_files $uri /index.html;`?**  
This ensures that React routing (SPA-style) works. Without it, direct page refreshes might return 404.

Enable the config:

```bash
ln -s /etc/nginx/sites-available/frontend.trashsettechnology.com /etc/nginx/sites-enabled/
```

### ✅ Test the config

```bash
nginx -t
```

If it says:
```
syntax is ok
test is successful
```

Reload Nginx:

```bash
systemctl reload nginx
```

Test your site at `http://your-domain.com`.

---

## ⚠️ Fix: "This is a homepage" Showing Instead of App

### Problem:
You're seeing basic HTML instead of your React app.

### Fix:

```bash
cat /var/www/html/index.html
```

If it's not your React file:

```bash
rm -rf /var/www/html/*
cp -r dist/* /var/www/html
systemctl reload nginx
```

---

# 🔒 Part 2: SSL and HTTPS Configuration

---

## 🌍 Step 8: Point Your Domain to Your VPS

In your domain DNS settings, add:

| Type | Name | Value          |
|------|------|-----------------|
| A    | @    | `your-server-ip` |
| A    | www  | `your-server-ip` |

Test:

```bash
dig +short trashsettechnology.com
```

---

## 🔐 Step 9: Secure with SSL (HTTPS)

Install Certbot:

```bash
apt install certbot python3-certbot-nginx -y
```

Run Certbot to fetch and install certificates:

```bash
certbot --nginx -d trashsettechnology.com -d www.trashsettechnology.com
```

Choose option `2` to redirect HTTP to HTTPS.

---

## ❌ Error Fix: NXDOMAIN

If you get:

```
NXDOMAIN looking up A for www.trashsettechnology.com
```

It means `www` is not pointed correctly. Go back to DNS settings and add:

| Type | Name | Value       |
|------|------|-------------|
| A    | www  | your VPS IP |

Wait 5–30 mins, then re-run Certbot.

---

## 🔁 Optional: Only Root Domain First

```bash
certbot --nginx -d trashsettechnology.com
```

Add `www` later:

```bash
certbot --expand -d trashsettechnology.com -d www.trashsettechnology.com
```

---

## ✅ Final Checklist

- [x] Project built using `npm run build`
- [x] Files copied to `/var/www/html`
- [x] Nginx configured and tested
- [x] Domain DNS pointed to VPS
- [x] Certbot SSL installed and redirect working

---

## 🏁 Done!

You've deployed your Vite + React app to a VPS with Nginx, connected a custom domain, and installed HTTPS.

Now your site is **secure**, **live**, and **production-ready**.  
Next steps? Add a backend, CI/CD pipeline, or host multiple apps using subdomains. 🎯
