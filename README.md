# CS2 WebRadar with NGINX and SSL Setup

This guide explains how to set up and deploy a CS2 radar system using NGINX with SSL, SSH tunneling, and backend services.
## Demo Video

[![Watch the video on YouTube](https://img.youtube.com/vi/AieihkpiDms/maxresdefault.jpg)](https://youtu.be/AieihkpiDms)


---

## Features and To-Do List

### ✅ **Added Features**
1. **Grenade Prediction with Lightning Strike**  
   - Displays predicted grenade trajectory with a visual lightning effect along the path.
2. **Player Names on Radar**  
   - Shows the names of players directly on the radar for better identification.
3. **Movement Trails**  
   - Visual trails showing players' recent movement paths for tracking.
4. **Player Tracing / Free Cam**  
   - Allows free-camera mode to trace any player's movement and actions in real-time.
5. **Grenade Trajectory**  
   - Predicts and displays grenade paths on the radar.

---

### 🚧 **To-Do List**
1. **Fix Heatmap**  
   - Improve heatmap functionality to display the most common spots:
     - **Per Match**: Highlight areas where players frequently gather during the match.  
     - **Per Player**: Highlight individual players' most visited locations.
2. **Fix AI Analysis**  
   - Current AI analysis logic is flawed (produced by AI). Revise and make it more dynamic and intuitive.
3. **Add Lightning Strike Effect**  
   - Implement a lightning effect between the killer and the killed player to indicate kills on the radar.
4. **Refine Grenade Trajectory**  
   - Add collision detection for grenade trajectories to account for:
     - Walls
     - Map boundaries
     - Other in-game objects

---

This section provides a clear overview of your progress and next steps for potential contributors or users of your project. Let me know if you'd like to expand on any point!
---

## VPS Setup: NGINX Reverse Proxy with SSL

### Prerequisites
- A Linux VPS (e.g., Ubuntu 20.04).
- A registered domain name (replace `put.your.domain`).
- Backend services running on:
  - `http://localhost:35174` for the main app.
  - `http://localhost:32006` for `/cs2_webradar`.

---

### 1. Install NGINX and Certbot
Run the following commands on your VPS:
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

### 5. Verify the Setup
- Visit `https://put.your.domain` to confirm the main service works.
- Test `https://put.your.domain/cs2_webradar`.

---

### 6. Auto-Renew SSL
Certbot handles automatic SSL renewal. Test with:
```bash
sudo certbot renew --dry-run
```

---

## SSH Tunnel Setup: `.bat` File

1. **Create the `.bat` File:**
   - Open a text editor (e.g., Notepad) and paste the following:
     ```bat
     @echo off
     ssh -R 0.0.0.0:35174:localhost:5173 -R 0.0.0.0:32006:localhost:22006 yourusername@vpsip -N
     ```
   - Replace:
     - `yourusername` with your VPS username.
     - `vpsip` with your VPS IP address.
     - `localhost:5173` and `localhost:22006` with the local ports for your services.

2. **Save the File:**
   Save it as `start-tunnel.bat` in a convenient location.

3. **Usage:**
   - After launching **usermode** and **webapp**, run `start-tunnel.bat` to set up the SSH tunnel.
   - Provide your VPS password when prompted. If the terminal stays open without output, the tunnel is working.

---

## Final Configuration and Running the Application

### 1. Update the WebApp
1. Open the file:
   ```text
   webapp/src/app.jsx
   ```
2. Locate the following line:
   ```javascript
   const webSocketURL = 'wss://yourdomain.name/cs2_webradar';
   ```
3. Replace `yourdomain.name` with your actual domain (e.g., `put.your.domain`).

---

### 2. Install Dependencies
1. **Install Node.js** (if not installed):
   - Download and install from [https://nodejs.org/](https://nodejs.org/).
   - Ensure `node` and `npm` are in your system path.

2. **Run the Installation Script**:
   - Double-click `install.bat` in the main folder.

---

### 3. Build the `usermode` Application
1. Navigate to the `usermode` folder.
2. Open `cs2_webradar.sln` using **Visual Studio**.
3. Build the project:
   - In Visual Studio, go to `Build > Build Solution`.

---

### 4. Launch the Application
1. **Start the SSH Tunnel**:
   - Run `start-tunnel.bat`.
2. **Launch the Components**:
   - Double-click `start.bat` in the main folder to start the web server.
   - Run `usermode.exe` from the `Release` folder inside the `usermode` directory.

---

### 5. Verify the Setup
1. Open your browser and visit:
   ```text
   https://put.your.domain
   ```
2. Join a CS2 match.
3. If everything works, the radar should display on your domain.

---

## Troubleshooting
- **Radar not working?**
  - Ensure `start-tunnel.bat` is running.
  - Verify `usermode.exe` was built and launched successfully.
  - Check NGINX logs:
    ```bash
    sudo tail -f /var/log/nginx/error.log
    ```
- **Backend ports:**
  Confirm services are running on `localhost:5173` and `localhost:22006`.

---

Now your CS2 radar is fully set up and accessible via your domain. 🎯

--- 

# Code inspired by Clauadv's project: [cs2_webradar](https://github.com/clauadv/cs2_webradar)
