## 🔄 **Reverse Proxy Setup Guide**  

### ✅ **What is a Reverse Proxy?**  
A **reverse proxy** sits between clients and backend servers, handling requests and improving:  
✔ **Security** (hides backend servers, DDoS protection)  
✔ **Performance** (load balancing, caching, SSL termination)  
✔ **Flexibility** (route traffic based on domains/paths)  

Common use cases:  
- Hosting multiple apps on one server  
- Securing Node.js/Python apps  
- Load balancing between servers  

---

## 🛠️ **Step 1: Choose Your Reverse Proxy**  

| Tool          | Best For                          | Config Complexity |  
|--------------|----------------------------------|------------------|  
| **Nginx**    | High performance, static files   | Moderate         |  
| **Caddy**    | Automatic HTTPS, simplicity      | Easy             |  
| **Apache**   | Legacy apps, .htaccess support   | High             |  
| **Traefik**  | Docker/Kubernetes integration    | Moderate         |  

---

## 🌐 **Step 2: Basic Reverse Proxy Setup**  

### **Nginx Example** (`/etc/nginx/conf.d/example.conf`)  
```nginx
server {
    listen 80;
    server_name app.yourdomain.com;

    location / {
        proxy_pass http://localhost:3000;  # Forward to Node.js app
        proxy_set_header Host $host;
        proxy_set_header X-Real-IP $remote_addr;
    }
}
```
Test & reload:  
```bash
sudo nginx -t && sudo systemctl reload nginx
```

### **Caddy Example** (`/etc/caddy/Caddyfile`)  
```plaintext
app.yourdomain.com {
    reverse_proxy localhost:3000
}
```
Restart:  
```bash
sudo systemctl restart caddy
```

---

## 🔒 **Step 3: Add HTTPS (SSL Termination)**  

### **With Certbot (Nginx/Apache)**  
```bash
sudo certbot --nginx -d app.yourdomain.com
```
Certbot automatically:  
1. Gets Let's Encrypt certificate  
2. Updates Nginx config for HTTPS  
3. Sets up auto-renewal  

### **Caddy (Automatic HTTPS)**  
Just add domain - Caddy handles SSL automatically:  
```plaintext
app.yourdomain.com {
    reverse_proxy localhost:3000
}
```

---

## 🧩 **Step 4: Advanced Routing**  

### **Path-Based Routing**  
```nginx
location /api/ {
    proxy_pass http://localhost:8000/;  # Python API
}

location /app/ {
    proxy_pass http://localhost:3000/;  # React App
}
```

### **WebSocket Support**  
```nginx
location /chat/ {
    proxy_pass http://localhost:8080;
    proxy_http_version 1.1;
    proxy_set_header Upgrade $http_upgrade;
    proxy_set_header Connection "upgrade";
}
```

---

## ⚖️ **Step 5: Load Balancing**  

### **Nginx Load Balancer**  
```nginx
upstream backend {
    server 192.168.1.10:3000;
    server 192.168.1.11:3000;
}

server {
    location / {
        proxy_pass http://backend;
    }
}
```

### **Caddy Load Balancing**  
```plaintext
app.yourdomain.com {
    reverse_proxy server1:3000 server2:3000 {
        lb_policy round_robin
    }
}
```

---

## 🛡️ **Step 6: Security Headers**  
Add to your proxy config:  
```nginx
proxy_set_header X-Forwarded-Proto $scheme;
proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
proxy_set_header X-Content-Type-Options "nosniff";
add_header Strict-Transport-Security "max-age=63072000" always;
```

---

## 🔄 **Step 7: Test & Monitor**  
1. Check access logs:  
   ```bash
   tail -f /var/log/nginx/access.log
   ```
2. Verify SSL: [https://ssllabs.com/ssltest](https://ssllabs.com/ssltest)  
3. Test load balancing:  
   ```bash
   curl http://app.yourdomain.com
   ```

---

## 📚 **Cheat Sheet**  

| Task                  | Nginx                          | Caddy                     |  
|----------------------|-------------------------------|--------------------------|  
| Basic Proxy          | `proxy_pass http://backend;`  | `reverse_proxy backend`  |  
| SSL Termination      | Certbot                       | Automatic                |  
| Load Balancing       | `upstream` blocks             | `lb_policy`              |  
| WebSockets           | `Connection "upgrade"`        | Automatic                |  

---
