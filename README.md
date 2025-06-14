# 🔐 Secure Web Deployment: Cloudflare + AWS EC2

This project demonstrates a secure web app deployment on AWS EC2 using Cloudflare’s CDN, Origin Certificates, and HTTPS setup. It includes Git deployment, DNS routing, and EC2 hardening practices.

---

## 📌 Key Notes

- **Cloudflare Origin Certificate** is valid **only** between Cloudflare and your origin server (not browsers directly).
- **Do NOT expose** your private key (`key.pem`) in public repositories.
- **DNS propagation** after nameserver update may take up to **24–48 hours**.
- Always use **SSL Mode: Full (Strict)** in Cloudflare.
- Open only required ports on EC2 (22, 80, 443).

---

## ✅ Prerequisites

- AWS EC2 instance (Amazon Linux / Ubuntu)
- Domain (e.g., Namecheap)
- Cloudflare account
- Git installed on EC2
- Apache or NGINX installed

---

## ⚙️ Step-by-Step Setup

### 1️⃣ EC2 Instance Setup

```bash
ssh -i "your-key.pem" ec2-user@your-ec2-ip

# Install Apache (Amazon Linux example)
sudo yum update -y
sudo yum install httpd -y
sudo systemctl start httpd
sudo systemctl enable httpd
```

---

### 2️⃣ Git Clone Web Files

```bash
# Install Git
sudo yum install git -y

# Clone repo
git clone https://github.com/yourusername/your-repo.git

# Copy to web root
sudo cp -r your-repo/* /var/www/html/
```

---

### 3️⃣ Generate Origin Certificate (Cloudflare)

1. Go to Cloudflare → **SSL/TLS > Origin Server**
2. Click **Create Certificate**
3. Choose:
   - RSA, 15 years
   - Domains: `yourdomain.com`, `*.yourdomain.com`
4. Download/save `cert.pem` and `key.pem`

---

### 4️⃣ Install SSL on EC2

```bash
sudo mkdir -p /etc/ssl/certs/cloudflare
sudo nano /etc/ssl/certs/cloudflare/cert.pem
# Paste cert content

sudo nano /etc/ssl/certs/cloudflare/key.pem
# Paste private key content

sudo chmod 600 /etc/ssl/certs/cloudflare/key.pem
```

#### Apache SSL Config (`/etc/httpd/conf.d/ssl.conf`)

```apache
<VirtualHost *:443>
    ServerName yourdomain.com
    DocumentRoot /var/www/html

    SSLEngine on
    SSLCertificateFile /etc/ssl/certs/cloudflare/cert.pem
    SSLCertificateKeyFile /etc/ssl/certs/cloudflare/key.pem
</VirtualHost>
```

```bash
sudo systemctl restart httpd
```

---

### 5️⃣ Point Domain to EC2 in Cloudflare

- Add A record for:
  - `@` → EC2 public IP → Proxy enabled (orange cloud)
  - `www` → EC2 public IP → Proxy enabled

---

### 6️⃣ Set Cloudflare SSL to Full (Strict)

- Go to **SSL/TLS > Overview**
- Set to **Full (Strict)**

---

### 7️⃣ Enable HTTPS Redirect

- Cloudflare > **SSL/TLS > Edge Certificates**
  - Turn on **Always Use HTTPS**
  - Enable **Automatic HTTPS Rewrites**

---

### 8️⃣ Secure EC2 Security Group

Allow only:

| Port | Purpose | Source       |
|------|---------|--------------|
| 22   | SSH     | Your IP only |
| 80   | HTTP    | 0.0.0.0/0    |
| 443  | HTTPS   | 0.0.0.0/0    |

---

### 9️⃣ DNS Propagation

- Wait up to 48 hours after changing nameservers in your domain registrar.
- Use https://dnschecker.org to confirm DNS propagation.

---

### 🔍 Debugging Workers (Optional)

- Go to Cloudflare > Workers
- Use `console.log()` to debug requests
- Monitor Worker logs in the dashboard

---

## ✅ Final Checks

- [ ] HTTPS lock in browser
- [ ] Cloudflare proxy ON
- [ ] EC2 security hardened
- [ ] GitHub repo excludes secrets
