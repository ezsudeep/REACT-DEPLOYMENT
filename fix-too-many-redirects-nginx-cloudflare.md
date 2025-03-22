
# 🛠 How I Fixed the “Too Many Redirects” Error After Deploying My Site with Nginx, Certbot, and Cloudflare

After deploying my React-based website to a VPS and pointing my domain `trashsettechnology.com` to it, I hit a frustrating wall:  
**“This page isn’t working — redirected you too many times (ERR_TOO_MANY_REDIRECTS).”**  

If you’ve faced the same, here’s a breakdown of what caused the problem, how I fixed it, and how you can avoid it.

---

## ❗ The Problem

I had:

- A frontend deployed to `/var/www/html`
- Nginx configured as a web server
- HTTPS managed via **Certbot**
- DNS and proxy handled by **Cloudflare**

But instead of loading, the site kept throwing:

> `ERR_TOO_MANY_REDIRECTS`

---

## 🪛 Troubleshooting Steps

### ✅ Step 1: Check Your Nginx Configuration

My file was at: `/etc/nginx/sites-available/frontend.trashsettechnology.com`

If you see multiple `return` directives or SSL blocks with missing certs, it's a red flag.

You can test your config with:

```bash
sudo nginx -t
```

---

### ✅ Step 2: Reissue SSL Certs with Certbot

If you deleted or corrupted your SSL certs, reissue them:

```bash
sudo certbot --nginx -d trashsettechnology.com -d www.trashsettechnology.com
```

---

### ✅ Step 3: Debug Redirect Chain

Use `curl` to check how your site is redirecting:

```bash
curl -IL https://trashsettechnology.com
```

If it shows endless `301` redirects to the same URL — it's likely a Cloudflare loop.

---

### ✅ Step 4: Check DNS Resolution

To confirm that your domain resolves correctly to your VPS IP, use:

```bash
dig +short trashsettechnology.com
```

Make sure it returns your VPS IP and not `192.0.2.x` (Cloudflare dummy) unless you’re intentionally routing through Cloudflare.

---

### ✅ Step 5: Set Cloudflare SSL Mode to Full (Strict)

Go to your [Cloudflare Dashboard](https://dash.cloudflare.com/):

1. Select your site
2. Go to **SSL/TLS → Overview**
3. Change mode from **Flexible** to **Full (Strict)**

This ensures Cloudflare connects to your origin **via HTTPS**, preventing loops.

---

## ⚙️ Final Nginx Configuration That Worked

```nginx
# Redirect HTTP to HTTPS
server {
    listen 80;
    server_name trashsettechnology.com www.trashsettechnology.com;

    return 301 https://$host$request_uri;
}

# Serve HTTPS content
server {
    listen 443 ssl;
    server_name trashsettechnology.com www.trashsettechnology.com;

    root /var/www/html;
    index index.html;

    ssl_certificate /etc/letsencrypt/live/trashsettechnology.com/fullchain.pem;
    ssl_certificate_key /etc/letsencrypt/live/trashsettechnology.com/privkey.pem;
    include /etc/letsencrypt/options-ssl-nginx.conf;
    ssl_dhparam /etc/letsencrypt/ssl-dhparams.pem;

    location / {
        try_files $uri /index.html;
    }
}
```

---

## ✅ TL;DR Fix Summary

| Problem                        | Solution                                |
|-------------------------------|-----------------------------------------|
| Nginx misconfigured            | Cleaned up server blocks                |
| Certbot cert missing/deleted  | Reissued with `--nginx` flag            |
| Cloudflare redirect loop      | Set SSL mode to **Full (Strict)**       |
| DNS misconfigured             | Verified with `dig` and fixed if needed |

---

## 💡 Key Takeaways

- Never use **Cloudflare Flexible SSL** with HTTPS-enabled Nginx.
- Always check redirect chains with `curl -IL`.
- Check DNS resolution with `dig +short yourdomain.com`.
- Let Certbot manage your SSL for simplicity.
- Clean up your Nginx configs — avoid duplicate or conflicting blocks.

---

**Hope this helps someone else save hours of debugging!**
