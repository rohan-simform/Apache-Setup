 **fully working Apache instance in your home directory** running on **port 8080**.

---

# 1️⃣ Verify Apache is Installed

First check Apache exists.

```bash
apache2 -v
```

Example output:

```
Server version: Apache/2.4.x
```

If this works, you can run Apache in **user mode**.

---

# 2️⃣ Create Project Structure

Create a small Apache environment inside your home directory.

```bash
mkdir -p ~/apache/{conf,logs,public_html}
```

Result:

```
~/apache
 ├── conf
 ├── logs
 └── public_html
```

---

# 3️⃣ Create a Test Website

```bash
echo "Apache local server working 🚀" > ~/apache/public_html/index.html
```

---

# 4️⃣ Find Your Username

Apache configs need **absolute paths**.

```bash
whoami
```

Your paths will look like:

```
/home/xyz.xyz@simform.dom/apache
```

---

# 5️⃣ Create Apache Config File

Create:

```bash
nano ~/apache/conf/httpd.conf
```

Paste this **minimal working configuration**.

```apache
ServerRoot "/usr/lib/apache2"

ServerName localhost

Listen 8080

# Required modules
LoadModule mpm_event_module /usr/lib/apache2/modules/mod_mpm_event.so
LoadModule authz_core_module /usr/lib/apache2/modules/mod_authz_core.so
LoadModule authz_host_module /usr/lib/apache2/modules/mod_authz_host.so
LoadModule dir_module /usr/lib/apache2/modules/mod_dir.so
LoadModule mime_module /usr/lib/apache2/modules/mod_mime.so

# MIME file
TypesConfig /etc/mime.types

# Logs
PidFile "/home/username/apache/logs/httpd.pid"
ErrorLog "/home/username/apache/logs/error.log"
CustomLog "/home/username/apache/logs/access.log" combined

# Website directory
DocumentRoot "/home/username/apache/public_html"

<Directory "/home/username/apache/public_html">
    Options Indexes FollowSymLinks
    AllowOverride All
    Require all granted
</Directory>

DirectoryIndex index.html
```

---

# 6️⃣ Fix Permissions

```bash
chmod -R 755 ~/apache
```

---

# 7️⃣ Test Configuration

Always test before starting.

```bash
apache2 -t -f ~/apache/conf/httpd.conf
```

Expected:

```
Syntax OK
```

---

# 8️⃣ Start Apache

```bash
apache2 -f ~/apache/conf/httpd.conf -k start
```

---

# 9️⃣ Verify Server is Running

### Open browser

```
http://localhost:8080
```

You should see:

```
Apache local server working 🚀
```

---

### Or test using curl

```bash
curl http://localhost:8080
```

---

# 🔟 Check Apache Processes

```bash
ps aux | grep apache2
```

You should see Apache running.

---

# 1️⃣1️⃣ Check Listening Port

```bash
ss -lntp | grep 8080
```

Expected:

```
LISTEN 0 511 0.0.0.0:8080
```

---

# 1️⃣2️⃣ View Logs

Logs are very important for debugging.

Error log:

```bash
tail -f ~/apache/logs/error.log
```

Access log:

```bash
tail -f ~/apache/logs/access.log
```

---

# 1️⃣3️⃣ Apache Control Commands

### Start

```bash
apache2 -f ~/apache/conf/httpd.conf -k start
```

### Stop

```bash
apache2 -f ~/apache/conf/httpd.conf -k stop
```

### Restart

```bash
apache2 -f ~/apache/conf/httpd.conf -k restart
```
### Reload Configuration

Equivalent to `systemctl reload apache2` on a normal server:

```bash
apache2 -f ~/apache/conf/httpd.conf -k graceful
```
---

# 1️⃣4️⃣ Test HTTP Headers

Backend developers often test like this:

```bash
curl -I http://localhost:8080
```

Example response:

```
HTTP/1.1 200 OK
Server: Apache/2.4
Content-Type: text/html
```

---

# 1️⃣5️⃣ Final Folder Structure

Your finished local Apache setup:

```
~/apache
 ├── conf
 │   └── httpd.conf
 ├── logs
 │   ├── access.log
 │   └── error.log
 └── public_html
     └── index.html
```
