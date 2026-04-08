# 🧱 Complete Guide: Apache + Docker PHP (Multi-Version)

> ⚠️ **IMPORTANT NOTE**
> Wherever you see `path` in this guide:
> 👉 Replace it with your actual project path
> Example: `/home/user/my-project`

---

# 🧠 Architecture

```text
Browser (localhost:810x)
        ↓
Apache (host)
        ↓
proxy_fcgi
        ↓
Docker PHP-FPM
        ↓
path (your project)
```

---

# 📁 Step 1: Prepare Project

```bash
mkdir -p path
nano path/index.php
```

```php
<?php phpinfo();
```

---

# 🐳 Step 2: docker-compose.yml

```bash
nano docker-compose.yml
```

```yaml
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
```

---

# 🚀 Step 3: Start Containers

```bash
docker compose up -d
```

Check:

```bash
docker ps
```

---

# ⚙️ Step 4: Apache Config

Edit:

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

## 🔹 Ensure modules

```apache
LoadModule proxy_module /usr/lib/apache2/modules/mod_proxy.so
LoadModule proxy_fcgi_module /usr/lib/apache2/modules/mod_proxy_fcgi.so
```

---

# 🌐 Step 5: VirtualHosts

---

## 🧾 PHP 7.4

```apache
<VirtualHost *:8101>

    DocumentRoot "path"

    <Directory "path">
        Require all granted
        AllowOverride All
    </Directory>

    <FilesMatch \.php$>
        SetHandler "proxy:fcgi://127.0.0.1:9101"
    </FilesMatch>

    ProxyPassMatch ^/(.*\.php(/.*)?)$ fcgi://127.0.0.1:9101/var/www/html/$1

</VirtualHost>
```

---

## 🧾 PHP 8.2

```apache
<VirtualHost *:8102>

    DocumentRoot "path"

    <Directory "path">
        Require all granted
        AllowOverride All
    </Directory>

    <FilesMatch \.php$>
        SetHandler "proxy:fcgi://127.0.0.1:9102"
    </FilesMatch>

    ProxyPassMatch ^/(.*\.php(/.*)?)$ fcgi://127.0.0.1:9102/var/www/html/$1

</VirtualHost>
```

---

## 🧾 PHP 8.4

```apache
<VirtualHost *:8103>

    DocumentRoot "path"

    <Directory "path">
        Require all granted
        AllowOverride All
    </Directory>

    <FilesMatch \.php$>
        SetHandler "proxy:fcgi://127.0.0.1:9103"
    </FilesMatch>

    ProxyPassMatch ^/(.*\.php(/.*)?)$ fcgi://127.0.0.1:9103/var/www/html/$1

</VirtualHost>
```

---

# 🔗 Step 6: Enable Sites

```bash
ln -s ~/apache/sites-available/php74.conf ~/apache/sites-enabled/
ln -s ~/apache/sites-available/php82.conf ~/apache/sites-enabled/
ln -s ~/apache/sites-available/php84.conf ~/apache/sites-enabled/
```

---

# 🔁 Step 7: Restart Apache

```bash
apache2 -t -f ~/apache/conf/httpd.conf
apache2 -f ~/apache/conf/httpd.conf -k restart
```

---

# 🌐 Step 8: Access

* [http://localhost:8101](http://localhost:8101) → PHP 7.4
* [http://localhost:8102](http://localhost:8102) → PHP 8.2
* [http://localhost:8103](http://localhost:8103) → PHP 8.4

---

# 🧪 Step 9: Verify

```php
<?php phpinfo();
```

---

# 🧠 Key Concepts

### 🔹 Path Mapping

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

→ Fix `ProxyPassMatch`

---

### ❌ 403 Forbidden

→ Fix `<Directory>` permissions

---

### ❌ Connection refused

→ Docker not running

---

# 🔥 Final Result

* One project (`path`)
* Multiple PHP versions
* Switch via ports
* Production-style setup


Just tell me 👍
