# Docker AutoSSL Nginx Setup

This project provides an automated **Nginx reverse proxy with Let's Encrypt SSL certificates** using [`nginx-proxy`](https://github.com/nginx-proxy/nginx-proxy) and [`acme-companion`](https://github.com/nginx-proxy/acme-companion).  
It allows you to run multiple services behind a secure HTTPS reverse proxy with minimal configuration.

---

## 📂 Project Structure

.
├── LICENSE
├── README.md
├── certs
│   └── dhparam.pem
├── docker-compose.yaml
├── html
│   └── index.html
├── proxy-service
│   └── default.conf
└── structure.txt

4 directories, 7 files

---

## 🚀 Services

### 1. **nginx-proxy**

- Image: `nginxproxy/nginx-proxy`
- Acts as the reverse proxy, automatically configuring virtual hosts for containers.
- Listens on ports `80` and `443`.
- Mounts Docker socket to detect new containers with `VIRTUAL_HOST`.

### 2. **nginx-proxy-acme**

- Image: `nginxproxy/acme-companion`
- Works alongside `nginx-proxy` to automatically issue and renew Let's Encrypt SSL certificates.
- Certificates are stored in `./certs`.

### 3. **proxy-service**

- Image: `nginx:alpine`
- Loads custom Nginx configuration from `proxy-service/default.conf`.
- Environment variables:
  - `VIRTUAL_HOST` → domain name to route.
  - `LETSENCRYPT_HOST` → domain for SSL certificate.
  - `LETSENCRYPT_EMAIL` → email for Let's Encrypt.
- **Important Note:**  
  The `TARGET_SERVICE` should **not** be set via environment variables inside `default.conf`.  
  Instead, you must **hardcode the backend service address** in the `proxy_pass` directive.  
  Example:
  ```nginx
  proxy_pass http://host.docker.internal:4000;
  ```

### 4. hello-world (example service)

- A sample nginx:alpine container serving static files from ./html.

###

### ⚙️ Usage

1. Clone the repository

````bash
git clone https://github.com/yourusername/docker-autossl-nginx.git
cd docker-autossl-nginx```
````

2. Create an .env file

```bash
DEFAULT_EMAIL=your-email@example.com
VIRTUAL_HOST=yourdomain.com
LETSENCRYPT_HOST=yourdomain.com
LETSENCRYPT_EMAIL=your-email@example.com

```

3. Update proxy-service/default.conf

- Set your target backend service manually:

````bash
proxy_pass http://host.docker.internal:4000;```
````

4. Start the stack

```bash
docker compose up -d
```

5. Access your services

https://yourdomain.com → routes to your hardcoded backend
https://yourdomain.com/hello-world → example Nginx static site

###

### 🛠 Notes

Certificates are renewed automatically by acme-companion.

To add a new proxied service, define it in docker-compose.yaml with:

````bash
environment:

- VIRTUAL_HOST=newdomain.com
- LETSENCRYPT_HOST=newdomain.com
- LETSENCRYPT_EMAIL=your-email@example.com```

````

### Reminder: proxy_pass inside proxy-service/default.conf must always be hardcoded.
