# -hamo.autos-Secure-EC2-Web-Hosting-with-Cloudflare-Integration
This project documents the step-by-step process of purchasing a domain, configuring DNS via Cloudflare, securing an EC2 instance with SSL, and handling traffic using Cloudflare Workers.
🔗 Project Overview
We will:

Purchase a custom domain from Namecheap.

Use Cloudflare for DNS and traffic management.

Deploy a login page on an AWS EC2 instance.

Secure the connection using SSL (Cloudflare origin certificate).

Use a Cloudflare Worker to manage traffic.

🛠️ Technologies Used
Namecheap – Domain provider

Cloudflare – DNS and traffic management

AWS EC2 – Hosting the login page

Ubuntu (on EC2) – Operating system

MobSSL – Tool to install SSL certificates

Cloudflare Workers – Custom edge logic

✅ Step-by-Step Setup
1. Domain Purchase
Go to Namecheap.

Search and purchase the domain: hamo.autos.

2. Setup Cloudflare
Create an account on Cloudflare.

Add your domain (hamo.autos) to Cloudflare.

Cloudflare will scan for existing DNS records.

3. Update Nameservers on Namecheap
Go to Namecheap dashboard.

Select your domain → Manage → Nameservers.

Change to Custom DNS.

Enter Cloudflare's nameservers (provided during onboarding).

Save changes.

4. Add A Record in Cloudflare
Navigate to DNS → Add Record:

Type: A

Name: @ (or leave blank)

IPv4 Address: Your EC2 instance's public IP

TTL: Auto

Proxy Status: Proxy (orange cloud)

5. Launch and Configure EC2 Instance
Go to AWS EC2.

Launch an instance with:

OS: Ubuntu (e.g., 22.04)

Security Group: Allow ports 80, 443, and 22.

SSH into your EC2 instance:

bash
Copy
Edit
ssh -i /path/to/key.pem ubuntu@your-ec2-public-ip
6. Deploy Login Page
Copy your login HTML/CSS/JS files to /var/www/html/:

bash
Copy
Edit
sudo apt update
sudo apt install apache2
sudo cp -r your-login-page/* /var/www/html/
7. Install SSL using Cloudflare Origin Certificate
On Cloudflare:

Go to SSL/TLS → Origin Server.

Click Create Certificate.

Choose "Let Cloudflare generate a private key and CSR".

Set expiration (e.g., 15 years).

Copy the certificate and private key.

On EC2:

bash
Copy
Edit
sudo mkdir -p /etc/ssl/cloudflare
sudo nano /etc/ssl/cloudflare/cert.pem  # paste certificate
sudo nano /etc/ssl/cloudflare/key.pem   # paste private key
Install mob_ssl or configure Apache:

bash
Copy
Edit
sudo apt install libapache2-mod-ssl
sudo a2enmod ssl
sudo nano /etc/apache2/sites-available/default-ssl.conf
Update Apache SSL configuration:

apacheconf
Copy
Edit
SSLEngine on
SSLCertificateFile /etc/ssl/cloudflare/cert.pem
SSLCertificateKeyFile /etc/ssl/cloudflare/key.pem
Enable SSL site and restart Apache:

bash
Copy
Edit
sudo a2ensite default-ssl
sudo systemctl restart apache2
8. Force HTTPS in Cloudflare
Go to SSL/TLS → Edge Certificates.

Turn on:

Always Use HTTPS

Automatic HTTPS Rewrites

9. Deploy a Basic Cloudflare Worker
Go to Workers & Pages → Create Worker.

Paste a simple worker script:

js
Copy
Edit
export default {
  async fetch(request) {
    const url = new URL(request.url);
    return fetch(request);
  }
}
Save and deploy the worker.

Bind it to your route (e.g., https://hamo.autos/*).

📡 Verification
Access https://hamo.autos in your browser.

Confirm:

HTTPS is working with Cloudflare SSL.

Traffic reaches your EC2 instance.

Worker is responding to requests (use logging or modify response).

📁 Folder Structure (Example)
pgsql
Copy
Edit
project-root/
├── login-page/
│   ├── index.html
│   ├── styles.css
│   └── script.js
├── cloudflare-worker/
│   └── worker.js
├── README.md
└── ssl/
    ├── cert.pem
    └── key.pem
🧠 Future Improvements
Add user authentication logic (e.g., JWT, OAuth).

Monitor traffic with Cloudflare analytics.

Configure WAF (Web Application Firewall).

Enable HTTP/3 or QUIC support via Cloudflare.

📌 Notes
Cloudflare Origin Certificate is only trusted between Cloudflare and your server. It's not trusted in browsers directly.

Always secure your private key.
