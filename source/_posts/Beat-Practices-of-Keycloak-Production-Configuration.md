---
title: Beat Practices of Keycloak Production Configuration
date: 2025-03-22 18:26:05
categories:
- Keycloak
tags:
- Keycloak
---

### **1. Hostname Configuration in `keycloak.conf`**
The `hostname` setting in `keycloak.conf` is **mandatory for production deployments** if Keycloak needs to generate URLs (e.g., for redirects, emails, or tokens). This ensures that Keycloak uses the correct public-facing URL (e.g., `www.keytomarvel.com`) instead of the internal server address.

- **Why is it important?**
  - Keycloak uses the `hostname` to construct URLs for redirects, emails, and tokens.
  - If not set, Keycloak might generate URLs using the internal server IP or hostname, which won’t work for external clients.

- **Example Configuration**:
  ```ini
  hostname=www.keytomarvel.com
  ```

---

### **2. Proxy Configuration in `keycloak.conf`**
Since NGINX is handling HTTPS termination, you need to configure Keycloak to trust the proxy. This is done using the `proxy` setting.

- **Key Configuration**:
  ```ini
  proxy=edge
  ```

  - `proxy=edge`: Use this when NGINX terminates HTTPS and forwards requests to Keycloak over HTTP.
  - If NGINX forwards requests over HTTPS to Keycloak, use `proxy=reencrypt`.

- **Why is it important?**
  - Keycloak needs to know it’s behind a proxy to correctly handle headers like `X-Forwarded-For` and `X-Forwarded-Proto`.

---

### **3. NGINX Configuration**
Here’s an example NGINX configuration for routing traffic to Keycloak:

#### **NGINX Configuration Example**
```nginx
server {
    listen 80;
    server_name www.keytomarvel.com;
    return 301 https://$host$request_uri;
}

server {
    listen 443 ssl;
    server_name www.keytomarvel.com;

    ssl_certificate /etc/nginx/ssl/certificate.crt;
    ssl_certificate_key /etc/nginx/ssl/private.key;

    location / {
        proxy_pass http://keycloak-server:8080; # Forward to Keycloak over HTTP
        proxy_set_header Host $host;
        proxy_set_header X-Real-IP $remote_addr;
        proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
        proxy_set_header X-Forwarded-Proto $scheme;
    }
}
```

- **Key Points**:
  - NGINX terminates HTTPS and forwards requests to Keycloak over HTTP.
  - The `proxy_set_header` directives ensure Keycloak receives the correct client IP and protocol information.

---

### **4. Additional Production Considerations**
Here are some additional settings and best practices for a production deployment:

#### **a. Database Connection Pooling**
Ensure the database connection pool is configured for production workloads:
```ini
db-pool-initial-size=10
db-pool-max-size=100
```

#### **b. Caching**
Enable distributed caching for high availability:
```ini
cache=ispn
cache-stack=kubernetes # or 'tcp' for non-Kubernetes environments
```

#### **c. Health Checks and Metrics**
Enable health checks and metrics for monitoring:
```ini
health-enabled=true
metrics-enabled=true
```

#### **d. Token and Session Timeouts**
Adjust token and session timeouts for security and usability:
```ini
token-lifespan=3600 # Access token lifespan (1 hour)
refresh-token-lifespan=86400 # Refresh token lifespan (24 hours)
session-max-lifespan=86400 # Maximum session lifespan (24 hours)
session-idle-timeout=1800 # Session idle timeout (30 minutes)
```

#### **e. Logging**
Configure logging for production:
```ini
log-level=INFO
log-console-output=json
log-file=/var/log/keycloak/keycloak.log
```

#### **f. Email Configuration**
Set up email for password resets and notifications:
```ini
smtp-host=smtp.example.com
smtp-port=587
smtp-username=user@example.com
smtp-password=changeit
smtp-from=keycloak@example.com
smtp-ssl=false
smtp-starttls=true
```

#### **g. Security Headers**
Ensure NGINX adds security headers:
```nginx
add_header Strict-Transport-Security "max-age=31536000; includeSubDomains" always;
add_header X-Content-Type-Options "nosniff" always;
add_header X-Frame-Options "SAMEORIGIN" always;
add_header X-XSS-Protection "1; mode=block" always;
```

---

### **5. Full `keycloak.conf` for Production**
Here’s a complete `keycloak.conf` tailored for your setup:

```ini
# Keycloak Production Configuration
# Database Configuration
db=postgres
db-url=jdbc:postgresql://localhost:5432/keycloak
db-username=keycloak
db-password=changeit
db-pool-initial-size=10
db-pool-max-size=100

# HTTP and HTTPS Configuration
http-enabled=true
http-port=8080
hostname=www.keytomarvel.com
proxy=edge

# Cache Configuration
cache=ispn
cache-stack=kubernetes # or 'tcp' for non-Kubernetes environments

# Logging Configuration
log-level=INFO
log-console-output=json
log-file=/var/log/keycloak/keycloak.log

# Health and Metrics
health-enabled=true
metrics-enabled=true

# Token and Session Configuration
token-lifespan=3600
refresh-token-lifespan=86400
session-max-lifespan=86400
session-idle-timeout=1800

# Email Configuration
smtp-host=smtp.example.com
smtp-port=587
smtp-username=user@example.com
smtp-password=changeit
smtp-from=keycloak@example.com
smtp-ssl=false
smtp-starttls=true
```

---

### **6. Deployment Checklist**
1. **Set `hostname`**: Ensure `hostname` is set to the public-facing URL (`www.keytomarvel.com`).
2. **Configure NGINX**: Terminate HTTPS at NGINX and forward requests to Keycloak over HTTP.
3. **Enable Proxy Mode**: Set `proxy=edge` in `keycloak.conf`.
4. **Database Tuning**: Configure connection pooling and ensure the database is optimized.
5. **Caching**: Enable distributed caching for high availability.
6. **Monitoring**: Enable health checks and metrics.
7. **Security**: Add security headers in NGINX and ensure tokens/sessions are securely configured.

