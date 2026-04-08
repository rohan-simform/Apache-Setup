# 🧱 Complete Guide: Apache + Docker PHP (Multi-Version)

> ⚠️ **IMPORTANT**
> Replace every `path` with your actual project path
> Example: `/home/user/my-project`

---

# 📁 1. Create Project Folder

```bash
mkdir -p path
```

Create test file:

```bash
cat <<EOF > path/index.php
<?php phpinfo();
EOF
```

---

# 🐳 2. Create docker-compose.yml

```bash
cat <<EOF > docker-compose.yml
version: '3.8'

services:
  php74:
    image: php:7.4-fpm
    ports:
      - "9101:9000"
    volumes:
      - path:/var/www/html

  php82:
    image: php:8.2-fpm
    ports:
      - "9102:9000"
    volumes:
      - path:/var/www/html

  php84:
    image: php:8.4-fpm
    ports:
      - "9103:9000"
    volumes:
      - path:/var/www/html
EOF
```

---

# 🚀 3. Start Docker

```bash
docker compose up -d
```

Verify:

```bash
docker ps
```

---

# ⚙️ 4. Update Apache Config

```bash
nano ~/apache/conf/httpd.conf
```

---

## 🔹 Add ports

```apache
Listen 8101
Listen 8102
Listen 8103
```

---

## 🔹 Ensure modules exist

```apache
LoadModule proxy_module /usr/lib/apache2/modules/mod_proxy.so
LoadModule proxy_fcgi_module /usr/lib/apache2/modules/mod_proxy_fcgi.so
```

---

# 🌐 5. Create VirtualHost Files

---

## 🧾 PHP 7.4

```bash
cat <<EOF > ~/apache/sites-available/php74.conf
<VirtualHost *:8101>

    DocumentRoot "path"

    <Directory "path">
        Require all granted
        AllowOverride All
    </Directory>

    <FilesMatch \\.php$>
        SetHandler "proxy:fcgi://127.0.0.1:9101"
    </FilesMatch>

    ProxyPassMatch ^/(.*\\.php(/.*)?)$ fcgi://127.0.0.1:9101/var/www/html/\$1

</VirtualHost>
EOF
```

---

## 🧾 PHP 8.2

```bash
cat <<EOF > ~/apache/sites-available/php82.conf
<VirtualHost *:8102>

    DocumentRoot "path"

    <Directory "path">
        Require all granted
        AllowOverride All
    </Directory>

    <FilesMatch \\.php$>
        SetHandler "proxy:fcgi://127.0.0.1:9102"
    </FilesMatch>

    ProxyPassMatch ^/(.*\\.php(/.*)?)$ fcgi://127.0.0.1:9102/var/www/html/\$1

</VirtualHost>
EOF
```

---

## 🧾 PHP 8.4

```bash
cat <<EOF > ~/apache/sites-available/php84.conf
<VirtualHost *:8103>

    DocumentRoot "path"

    <Directory "path">
        Require all granted
        AllowOverride All
    </Directory>

    <FilesMatch \\.php$>
        SetHandler "proxy:fcgi://127.0.0.1:9103"
    </FilesMatch>

    ProxyPassMatch ^/(.*\\.php(/.*)?)$ fcgi://127.0.0.1:9103/var/www/html/\$1

</VirtualHost>
EOF
```

---

# 🔗 6. Enable Sites

```bash
ln -s ~/apache/sites-available/php74.conf ~/apache/sites-enabled/php74.conf
ln -s ~/apache/sites-available/php82.conf ~/apache/sites-enabled/php82.conf
ln -s ~/apache/sites-available/php84.conf ~/apache/sites-enabled/php84.conf
```

---

# 🔁 7. Restart Apache

```bash
apache2 -t -f ~/apache/conf/httpd.conf
apache2 -f ~/apache/conf/httpd.conf -k restart
```

---

# 🌐 8. Open in Browser

* [http://localhost:8101](http://localhost:8101) → PHP 7.4
* [http://localhost:8102](http://localhost:8102) → PHP 8.2
* [http://localhost:8103](http://localhost:8103) → PHP 8.4

---

# 🧪 9. Verify

You should see different PHP versions on each port.

---

# 🧠 Key Concepts

### 🔹 Path mapping

| Apache | Docker          |
| ------ | --------------- |
| `path` | `/var/www/html` |

---

### 🔹 Ports

| Layer   | Ports     |
| ------- | --------- |
| Apache  | 8101–8103 |
| PHP-FPM | 9101–9103 |

---

### 🔹 Flow

```text
Browser → Apache → proxy_fcgi → PHP → Response
```

---

# ⚠️ Common Errors

### ❌ Primary script unknown

→ Fix with `ProxyPassMatch`

---

### ❌ 403 Forbidden

```bash
chmod -R 755 path
```

---

### ❌ Containers not running

```bash
docker compose up -d
```

---

# 🔥 Final Result

* One folder (`path`)
* 3 PHP versions
* Switch via port
* Real production-style setup

---
