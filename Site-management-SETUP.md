# 0️⃣ Verify Apache is Installed

First confirm Apache exists on the system.

```bash
apache2 -v
```

Example output:

```
Server version: Apache/2.4.x
```

If this works, you can run Apache **in user mode**.

---

# 1️⃣ Create Apache Directory Structure

Create a small Apache environment in your home directory.

```bash
mkdir -p ~/apache/{conf,logs,public_html,sites-available,sites-enabled}
```

Result:

```
~/apache
 ├── conf
 ├── logs
 ├── public_html
 ├── sites-available
 └── sites-enabled
```

This structure is similar to a real Apache installation.



# 2️⃣ Create a Test Website

Create a simple test page.

```bash
echo "Apache local server working 🚀" > ~/apache/public_html/index.html
```

---

# 3️⃣ Create Apache Configuration File

Create the main Apache config file.

```bash
nano ~/apache/conf/httpd.conf
```

Paste this configuration:

```apache
ServerRoot "/usr/lib/apache2"

ServerName localhost
Listen 8080

ServerTokens Prod
ServerSignature Off

Timeout 60
KeepAlive On
MaxKeepAliveRequests 100
KeepAliveTimeout 5

HostnameLookups Off

# Load required modules
LoadModule mpm_event_module /usr/lib/apache2/modules/mod_mpm_event.so
LoadModule authz_core_module /usr/lib/apache2/modules/mod_authz_core.so
LoadModule authz_host_module /usr/lib/apache2/modules/mod_authz_host.so
LoadModule dir_module /usr/lib/apache2/modules/mod_dir.so
LoadModule mime_module /usr/lib/apache2/modules/mod_mime.so
LoadModule rewrite_module /usr/lib/apache2/modules/mod_rewrite.so

# MIME types file
TypesConfig /etc/mime.types

AddDefaultCharset UTF-8

# Logging format
LogFormat "%h %l %u %t \"%r\" %>s %b" common
LogFormat "%h %l %u %t \"%r\" %>s %b \"%{Referer}i\" \"%{User-Agent}i\"" combined

# Logs
PidFile "/home/username/apache/logs/httpd.pid"
ErrorLog "/home/username/apache/logs/error.log"
CustomLog "/home/username/apache/logs/access.log" combined

# Default website
DocumentRoot "/home/username/apache/public_html"

<Directory "/home/username/apache/public_html">
    Options Indexes FollowSymLinks
    AllowOverride All
    Require all granted
</Directory>

DirectoryIndex index.html

# Load enabled sites
IncludeOptional /home/username/apache/sites-enabled/*.conf
```

⚠️ Replace **`username`** with your real username everywhere.

---

# 4️⃣ Fix Permissions

Ensure Apache can read the directories.

```bash
chmod -R 755 ~/apache
```

---

# 5️⃣ Test Apache Configuration

Always test the config before starting.

```bash
apache2 -t -f ~/apache/conf/httpd.conf
```

Expected output:

```
Syntax OK
```

---

# 6️⃣ Start Apache

```bash
apache2 -f ~/apache/conf/httpd.conf -k start
```

---

# 7️⃣ Verify the Server Works

Open your browser:

```
http://localhost:8080
```

You should see:

```
Apache local server working 🚀
```

Or test with curl:

```bash
curl http://localhost:8080
```

---

# 8️⃣ Check Apache Processes

```bash
ps aux | grep apache2
```

You should see Apache processes running.

---

# 9️⃣ Check Listening Port

```bash
ss -lntp | grep 8080
```

Expected:

```
LISTEN 0 511 0.0.0.0:8080
```

---

# 🔟 Check Logs

Logs are the **first place to debug problems**.

Error log:

```bash
tail -f ~/apache/logs/error.log
```

Access log:

```bash
tail -f ~/apache/logs/access.log
```

---

# 1️⃣1️⃣ Apache Control Commands

Start server:

```bash
apache2 -f ~/apache/conf/httpd.conf -k start
```

Stop server:

```bash
apache2 -f ~/apache/conf/httpd.conf -k stop
```

Restart server:

```bash
apache2 -f ~/apache/conf/httpd.conf -k restart
```

Reload configuration (recommended):

```bash
apache2 -f ~/apache/conf/httpd.conf -k graceful
```

Test configuration:

```bash
apache2 -t -f ~/apache/conf/httpd.conf
```

---

# 1️⃣2️⃣ Run Apache in Debug Mode (Very Useful)

To see errors directly in the terminal:

```bash
apache2 -f ~/apache/conf/httpd.conf -DFOREGROUND
```

This is extremely helpful when debugging configs.

---

# 1️⃣3️⃣ Test HTTP Headers

Backend developers often test responses like this:

```bash
curl -I http://localhost:8080
```

Example response:

```
HTTP/1.1 200 OK
Server: Apache
Content-Type: text/html
```

---

# 1️⃣4️⃣ Create a New Website (VirtualHost)

Create a new site config.

```bash
nano ~/apache/sites-available/site1.conf
```

Example configuration:

```apache
<VirtualHost *:8080>

    ServerName site1.local

    DocumentRoot "/home/username/apache/site1"

    <Directory "/home/username/apache/site1">
        Require all granted
    </Directory>

</VirtualHost>
```

Create site directory:

```bash
mkdir ~/apache/site1
echo "Site 1 working" > ~/apache/site1/index.html
```

Enable the site:

```bash
ln -s ~/apache/sites-available/site1.conf ~/apache/sites-enabled/
```

Reload Apache:

```bash
apache2 -f ~/apache/conf/httpd.conf -k graceful
```

---

# 1️⃣5️⃣ Final Directory Structure

Your finished Apache environment should look like this:

```
~/apache
 ├── conf
 │   └── httpd.conf
 ├── logs
 │   ├── access.log
 │   └── error.log
 ├── public_html
 │   └── index.html
 ├── sites-available
 │   └── site1.conf
 └── sites-enabled
     └── site1.conf
```
