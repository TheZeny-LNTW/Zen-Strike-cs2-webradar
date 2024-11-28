### VPS 

```markdown
# NGINX Reverse Proxy with SSL Setup

This guide explains how to set up an NGINX reverse proxy with SSL using Certbot for **`put.your.domain`**.

---

## Prerequisites
- A Linux server (e.g., Ubuntu 20.04).
- A registered domain name (replace `put.your.domain`).
- Backend services running on:
  - `http://localhost:35174` for the main app.
  - `http://localhost:32006` for `/cs2_webradar`.

---

## Steps

### 1. Install NGINX and Certbot
Run the following commands:
```bash
sudo apt update && sudo apt install nginx certbot python3-certbot-nginx -y
```

---

### 2. Create NGINX Configuration
1. Create a new NGINX configuration file:
   ```bash
   sudo nano /etc/nginx/sites-available/put.your.domain
   ```

2. Add the following content:
   ```nginx
   server {
       listen 443 ssl;
       server_name put.your.domain;

       ssl_certificate /etc/letsencrypt/live/put.your.domain/fullchain.pem; # managed by Certbot
       ssl_certificate_key /etc/letsencrypt/live/put.your.domain/privkey.pem; # managed by Certbot

       location / {
           proxy_pass http://localhost:35174;
           proxy_http_version 1.1;
           proxy_set_header Upgrade $http_upgrade;
           proxy_set_header Connection 'upgrade';
           proxy_set_header Host $host;
       }

       location /cs2_webradar {
           proxy_pass http://localhost:32006;
           proxy_http_version 1.1;
           proxy_set_header Upgrade $http_upgrade;
           proxy_set_header Connection "upgrade";
           proxy_set_header Host $host;
           proxy_read_timeout 86400;
       }
   }

   server {
       listen 80;
       server_name put.your.domain;
       return 301 https://$host$request_uri; # Redirect HTTP to HTTPS
   }
   ```

3. Enable the configuration:
   ```bash
   sudo ln -s /etc/nginx/sites-available/put.your.domain /etc/nginx/sites-enabled/
   ```

---

### 3. Obtain SSL Certificate
Run Certbot to get a free SSL certificate:
```bash
sudo certbot --nginx -d put.your.domain
```

Follow the prompts. Certbot will configure SSL automatically.

---

### 4. Restart NGINX
Reload NGINX to apply the changes:
```bash
sudo nginx -t
sudo systemctl reload nginx
```

---

### 5. Verify Setup
- Visit `https://put.your.domain` to confirm the main service works.
- Test `https://put.your.domain/cs2_webradar`.

---

### 6. Auto-Renew SSL
Certbot handles automatic SSL renewal. To test:
```bash
sudo certbot renew --dry-run
```

---

## Notes
- Ensure your backend services are running on the correct ports.
- Use `sudo tail -f /var/log/nginx/error.log` to debug any issues.

---

Here’s how you can format this step for your **README.md** in a clear and structured way:

---

### Next Step: Create a `.bat` File for SSH Tunneling

You need to create a `.bat` file to establish SSH tunneling between your local machine and your VPS. This ensures that the backend services are accessible via the configured ports on your VPS.

1. **Create the `.bat` File:**
   - Open a text editor (e.g., Notepad) and paste the following content:
     ```bat
     @echo off
     ssh -R 0.0.0.0:35174:localhost:5173 -R 0.0.0.0:32006:localhost:22006 yourusername@vpsip -N
     ```
   - Replace:
     - `yourusername` with your VPS username.
     - `vpsip` with the IP address of your VPS.
     - `localhost:5173` with the port where your **usermode** runs locally.
     - `localhost:22006` with the port where your **/cs2_webradar** service runs locally.

2. **Save the File:**
   - Save the file as `start-tunnel.bat` (or any name you prefer) in a convenient location.

3. **Usage:**
   - After starting **usermode** and your **webapp**, double-click the `start-tunnel.bat` file to create the SSH tunnel and ofc put ur password for vps nothing should pop up after it meaning that tunneling works.

4. **Keep the SSH Tunnel Open:**
   - The terminal window for the `.bat` file must remain open for the tunnel to stay active.

---


