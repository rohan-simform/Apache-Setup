

# 🧱 Complete Guide: Add phpMyAdmin to Existing Apache + Docker PHP Setup

> ⚠️ Replace `path` wherever needed
> ⚠️ Assumes you already have:
>
> * Apache (custom config in `~/apache`)
> * PHP-FPM containers (9101–9103)
> * MySQL container (`mysql`, port 3307)

---

# 🐳 1. Create Separate phpMyAdmin Docker File

```bash
cat <<EOF > docker-compose-phpmyadmin.yml
services:
  phpmyadmin:
    image: phpmyadmin/phpmyadmin
    container_name: phpmyadmin_container
    ports:
      - "9200:80"
    environment:
      PMA_HOST: mysql
      PMA_PORT: 3306
    networks:
      - phpmyadmin_net

networks:
  phpmyadmin_net:
EOF
```

---

# 🚀 2. Start phpMyAdmin

```bash
docker compose -f docker-compose-phpmyadmin.yml up -d
```

Verify:

```bash
docker ps
```

---

# 🔗 3. Connect phpMyAdmin to MySQL Container

Since MySQL was created using `docker run`, connect manually:

```bash
docker network connect php_phpmyadmin_net mysql
```

---

# 🔍 Verify Network

```bash
docker network inspect php_phpmyadmin_net
```

You should see:

* `phpmyadmin_container`
* `mysql`

---

# 🌐 4. Test Direct Access (IMPORTANT)

Open:

```
http://localhost:9200
```

Login:

* Server: `mysql`
* Username: `root`
* Password: `pass`

---

# ⚙️ 5. Update Apache Config (VERY IMPORTANT)

Open:

```bash
nano ~/apache/conf/httpd.conf
```

---

## 🔹 Add Required Modules

```apache
LoadModule proxy_module /usr/lib/apache2/modules/mod_proxy.so
LoadModule proxy_fcgi_module /usr/lib/apache2/modules/mod_proxy_fcgi.so
LoadModule proxy_http_module /usr/lib/apache2/modules/mod_proxy_http.so
```

---

## 🔹 Add Port

```apache
Listen 8104
```

---

# 🌐 6. Create VirtualHost for phpMyAdmin

```bash
cat <<EOF > ~/apache/sites-available/phpmyadmin.conf
<VirtualHost *:8104>

    ProxyPreserveHost On

    ProxyPass / http://127.0.0.1:9200/
    ProxyPassReverse / http://127.0.0.1:9200/

</VirtualHost>
EOF
```

---

# 🔗 7. Enable Site

```bash
ln -s ~/apache/sites-available/phpmyadmin.conf ~/apache/sites-enabled/phpmyadmin.conf
```

---

# 🔁 8. Restart Apache

```bash
apache2 -t -f ~/apache/conf/httpd.conf
apache2 -f ~/apache/conf/httpd.conf -k restart
```

---

# 🌐 9. Access phpMyAdmin

### Direct:

```
http://localhost:9200
```

### Via Apache (recommended):

```
http://localhost:8104
```

---

# 🧠 Key Concepts (IMPORTANT)

---

## 🔹 Container Networking

| From              | To               | Use |
| ----------------- | ---------------- | --- |
| Browser → MySQL   | `localhost:3307` |     |
| Container → MySQL | `mysql:3306` ✅   |     |

---

## 🔹 Why NOT `host.docker.internal`

* ❌ Not reliable on Linux
* ❌ Causes DNS errors
* ✅ Use container name instead

---

## 🔹 Proxy Types

| Service    | Apache Module  |
| ---------- | -------------- |
| PHP-FPM    | `proxy_fcgi`   |
| phpMyAdmin | `proxy_http` ✅ |

---

## 🔹 Request Flow

```text
Browser → Apache (8104)
        → proxy_http
        → phpMyAdmin (9200)
        → MySQL container
```

---

# ⚠️ Common Errors & Fixes

---

## ❌ Cannot log in to MySQL

✔ Ensure:

```yaml
PMA_HOST: mysql
PMA_PORT: 3306
```

✔ Ensure network connected:

```bash
docker network connect php_phpmyadmin_net mysql
```

---

## ❌ Internal Server Error (Apache)

✔ Missing module:

```apache
LoadModule proxy_http_module ...
```

---

## ❌ Name not known (mysql)

✔ Different networks → connect them

---

## ❌ Port not opening

```bash
ss -lntp | grep 9200
```

---

# 🔥 Final Setup Overview

| Service           | Port | Notes               |
| ----------------- | ---- | ------------------- |
| PHP 7.4           | 8101 | Apache → FPM        |
| PHP 8.2           | 8102 | Apache → FPM        |
| PHP 8.4           | 8103 | Apache → FPM        |
| phpMyAdmin        | 8104 | Apache → HTTP proxy |
| phpMyAdmin direct | 9200 | Docker              |
| MySQL             | 3307 | Host mapped         |

---

# 🚀 You Now Have

✅ Multi-version PHP
✅ Apache reverse proxy
✅ MySQL container
✅ phpMyAdmin integrated
✅ Clean separation of services

---
