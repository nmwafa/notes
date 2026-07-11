# Hardening Web Server: Apache & Nginx di Ubuntu Linux

**Versi:** 1.2

>Semua teknik yang ditulis sudah dites pada server Ubuntu 22.04 LTS
>
>Di bagian akhir ada trik singkat untuk [Hardening Wordpress dan phpMyAdmin](#hardening-wordpress-dan-phpmyadmin)

---

## Daftar Area Hardening

| No | Area |
|---|------|
| 1 | Update sistem & firewall |
| 2 | Sembunyikan versi/banner |
| 3 | Nonaktifkan modul/fitur tidak perlu |
| 4 | Konfigurasi SSL/TLS kuat |
| 5 | Security headers HTTP |
| 6 | Nonaktifkan directory listing |
| 7 | Batasi metode HTTP |
| 8 | Proteksi file & direktori sensitif |
| 9 | Timeout & ukuran request |
| 10 | Rate limiting & anti-DoS |
| 11 | WAF (Web Application Firewall) |
| 12 | Permission file konfigurasi |
| 13 | Konfigurasi logging |
| 14 | Anti PHP Shell & Anti bypass disable function |


---

## Bagian 1: Persiapan Sistem

### 1.1 Update Sistem

```bash
sudo apt update && sudo apt upgrade -y
sudo apt autoremove -y
```

### 1.2 Konfigurasi Firewall (UFW)

```bash
# Aktifkan UFW
sudo ufw enable

# Izinkan SSH, HTTP, HTTPS
sudo ufw limit ssh
sudo ufw allow 80/tcp
sudo ufw allow 443/tcp

# Tolak semua koneksi masuk lainnya (default)
sudo ufw default deny incoming
sudo ufw default allow outgoing

# Verifikasi status
sudo ufw status verbose
```

### 1.3 Nonaktifkan Layanan Tidak Perlu

```bash
# Cek layanan yang berjalan
sudo systemctl list-units --type=service --state=running

# Nonaktifkan layanan yang tidak dibutuhkan (sesuaikan)
sudo systemctl disable --now avahi-daemon
sudo systemctl disable --now cups
sudo systemctl disable --now bluetooth
```

---

## Bagian 2: Hardening Apache

### 2.1 Instalasi

```bash
sudo apt install apache2 -y
sudo systemctl enable apache2
```

### 2.2 Sembunyikan Versi dan Banner Server

Edit file konfigurasi utama:

```bash
sudo nano /etc/apache2/conf-available/security.conf
```

Tambahkan atau ubah:

```apache
# Sembunyikan versi Apache dari response header
ServerTokens Prod

# Matikan signature di error page
ServerSignature Off

# Nonaktifkan ETag (cegah kebocoran inode)
FileETag None
```

### 2.3 Nonaktifkan Modul yang Tidak Perlu

```bash
# Cek modul aktif
apache2ctl -M

# Nonaktifkan modul berisiko/tidak perlu
sudo a2dismod status
sudo a2dismod info
sudo a2dismod userdir
sudo a2dismod cgi

# Aktifkan modul keamanan
sudo a2enmod headers
sudo a2enmod rewrite
sudo a2enmod ssl

sudo systemctl restart apache2
```

### 2.4 Nonaktifkan Directory Listing

```bash
sudo a2dismod --force autoindex

sudo systemctl restart apache2
```

### 2.5 Batasi Metode HTTP

Tambahkan ke konfigurasi virtualhost `/etc/apache2/sites-available/000-default.conf`:

```apache
<Directory /var/www/html>
    <LimitExcept GET POST>
        Require all denied
    </LimitExcept>
</Directory>
```

Atau untuk menonaktifkan TRACE secara global di `/etc/apache2/conf-available/security.conf`:

```apache
TraceEnable off
```

### 2.6 Security Headers HTTP

```bash
sudo nano /etc/apache2/conf-available/security.conf
```

Tambahkan blok berikut:

```apache
<IfModule mod_headers.c>
    # Cegah clickjacking
    Header always set X-Frame-Options "SAMEORIGIN"

    # Cegah MIME-type sniffing
    Header always set X-Content-Type-Options "nosniff"

    # HTTP Strict Transport Security (aktifkan hanya jika sudah full HTTPS)
    Header always set Strict-Transport-Security "max-age=31536000; includeSubDomains; preload"

    # XSS Protection (legacy browsers)
    Header always set X-XSS-Protection "1; mode=block"

    # Referrer Policy
    Header always set Referrer-Policy "strict-origin-when-cross-origin"

    # Permissions Policy (matikan fitur berbahaya)
    Header always set Permissions-Policy "geolocation=(), microphone=(), camera=()"

    # Content Security Policy (sesuaikan dengan kebutuhan aplikasi)
    Header always set Content-Security-Policy "default-src 'self'; script-src 'self'; object-src 'none';"

    # Hapus header X-Powered-By jika ada
    Header unset X-Powered-By
    Header unset Server
</IfModule>
```

```bash
sudo a2enconf security
sudo systemctl reload apache2
```

### 2.7 Konfigurasi SSL/TLS

```bash
sudo nano /etc/apache2/mods-available/ssl.conf
```

Ubah konfigurasi SSL:

```apache
# Hanya izinkan TLS 1.2 dan 1.3
SSLProtocol -all +TLSv1.2 +TLSv1.3

# Gunakan cipher suite yang kuat
SSLCipherSuite ECDHE-ECDSA-AES128-GCM-SHA256:ECDHE-RSA-AES128-GCM-SHA256:ECDHE-ECDSA-AES256-GCM-SHA384:ECDHE-RSA-AES256-GCM-SHA384:ECDHE-ECDSA-CHACHA20-POLY1305:ECDHE-RSA-CHACHA20-POLY1305:DHE-RSA-AES128-GCM-SHA256:DHE-RSA-AES256-GCM-SHA384

# Prioritaskan cipher server
SSLHonorCipherOrder on

# Aktifkan OCSP Stapling
SSLUseStapling on
SSLStaplingCache "shmcb:logs/ssl_stapling(32768)"

# Nonaktifkan compression (CRIME attack)
SSLCompression off

# Aktifkan session ticket (performa)
SSLSessionTickets off
```

### 2.8 Proteksi File Sensitif

```bash
sudo nano /etc/apache2/conf-available/security.conf
```

Tambahkan:

```apache
# Blokir akses ke file konfigurasi
<FilesMatch "^\.ht">
    Require all denied
</FilesMatch>

<FilesMatch "\.(env|log|bak|sql|config|conf|ini|sh|git|svn)$">
    Require all denied
</FilesMatch>

# Blokir akses ke direktori .git
<DirectoryMatch "/\.git">
    Require all denied
</DirectoryMatch>
```

### 2.9 Batasi Ukuran Request dan Timeout

```bash
sudo nano /etc/apache2/apache2.conf
```

Tambahkan:

```apache
# Batas ukuran body request (10MB)
LimitRequestBody 10485760

# Batas ukuran header request
LimitRequestFieldSize 8190
LimitRequestFields 100

# Timeout konfigurasi
Timeout 60
KeepAliveTimeout 5
MaxKeepAliveRequests 100
```

### 2.10 Instalasi dan Konfigurasi mod_security (WAF)

```bash
sudo apt install libapache2-mod-security2 -y
sudo a2enmod security2

# Salin konfigurasi default sebagai aktif
sudo cp /etc/modsecurity/modsecurity.conf-recommended /etc/modsecurity/modsecurity.conf

# Aktifkan mode enforcement
sudo sed -i 's/SecRuleEngine DetectionOnly/SecRuleEngine On/' /etc/modsecurity/modsecurity.conf
```

Install OWASP Core Rule Set:

```bash
sudo apt install modsecurity-crs -y

# Aktifkan CRS
sudo ln -s /usr/share/modsecurity-crs /etc/modsecurity/rules

# Tambahkan include ke konfigurasi modsecurity
echo 'IncludeOptional /etc/modsecurity/rules/*.conf' | sudo tee -a /etc/apache2/mods-enabled/security2.conf
```

### 2.11 Instalasi mod_evasive (Anti-DoS)

```bash
sudo apt install libapache2-mod-evasive -y
sudo a2enmod evasive

sudo nano /etc/apache2/mods-enabled/evasive.conf
```

```apache
<IfModule mod_evasive20.c>
    DOSHashTableSize    3097
    DOSPageCount        5
    DOSSiteCount        50
    DOSPageInterval     1
    DOSSiteInterval     1
    DOSBlockingPeriod   60
    DOSEmailNotify      admin@domain.com
    DOSLogDir           /var/log/apache2/mod_evasive
</IfModule>
```

```bash
sudo mkdir -p /var/log/apache2/mod_evasive
sudo chown www-data:www-data /var/log/apache2/mod_evasive
sudo systemctl restart apache2
```

### 2.12 Permission File Konfigurasi Apache

```bash
# Konfigurasi hanya dapat dibaca root dan grup apache
sudo chown -R root:www-data /etc/apache2/
sudo chmod -R 640 /etc/apache2/*.conf
sudo chmod -R 750 /etc/apache2/

# Root document web
sudo chown -R www-data:www-data /var/www/html/
sudo chmod -R 755 /var/www/html/
sudo find /var/www/html/ -type f -exec chmod 644 {} \;
```

### 2.13 Konfigurasi Logging Apache

```bash
sudo nano /etc/apache2/apache2.conf
```

Pastikan logging sudah dikonfigurasi:

```apache
LogLevel warn
ErrorLog ${APACHE_LOG_DIR}/error.log
CustomLog ${APACHE_LOG_DIR}/access.log combined
```

Aktifkan log rotation:

```bash
sudo nano /etc/logrotate.d/apache2
```

```
/var/log/apache2/*.log {
    daily
    missingok
    rotate 14
    compress
    delaycompress
    notifempty
    create 640 root adm
    sharedscripts
    postrotate
        /usr/sbin/apache2ctl graceful > /dev/null
    endscript
}
```

### 2.14 Verifikasi Konfigurasi Apache

```bash
sudo apache2ctl configtest
sudo systemctl restart apache2

# Uji security headers
curl -I https://your-domain.com
```

---

## Bagian 3: Hardening Nginx

### 3.1 Instalasi

```bash
sudo apt install nginx -y
sudo systemctl enable nginx
```

### 3.2 Sembunyikan Versi Server

```bash
sudo nano /etc/nginx/nginx.conf
```

Di dalam blok `http { }`, tambahkan atau pastikan ada:

```nginx
http {
    # Sembunyikan versi Nginx
    server_tokens off;

    # ... konfigurasi lainnya
}
```

### 3.3 Konfigurasi SSL/TLS

Buat file parameter Diffie-Hellman terlebih dahulu:

```bash
sudo openssl dhparam -out /etc/nginx/dhparam.pem 2048
```

Edit konfigurasi SSL di `nginx.conf` atau dalam blok server:

```nginx
# Dalam blok http {} di nginx.conf
ssl_protocols TLSv1.2 TLSv1.3;
ssl_ciphers ECDHE-ECDSA-AES128-GCM-SHA256:ECDHE-RSA-AES128-GCM-SHA256:ECDHE-ECDSA-AES256-GCM-SHA384:ECDHE-RSA-AES256-GCM-SHA384:ECDHE-ECDSA-CHACHA20-POLY1305:ECDHE-RSA-CHACHA20-POLY1305;
ssl_prefer_server_ciphers off;
ssl_session_timeout 1d;
ssl_session_cache shared:MozSSL:10m;
ssl_session_tickets off;
ssl_dhparam /etc/nginx/dhparam.pem;

# OCSP Stapling
ssl_stapling on;
ssl_stapling_verify on;
resolver 8.8.8.8 8.8.4.4 valid=300s;
resolver_timeout 5s;
```

### 3.4 Security Headers HTTP

Di dalam blok `http { }` pada `nginx.conf`:

```nginx
# Security Headers
add_header X-Frame-Options "SAMEORIGIN" always;
add_header X-Content-Type-Options "nosniff" always;
add_header X-XSS-Protection "1; mode=block" always;
add_header Strict-Transport-Security "max-age=31536000; includeSubDomains; preload" always;
add_header Referrer-Policy "strict-origin-when-cross-origin" always;
add_header Permissions-Policy "geolocation=(), microphone=(), camera=()" always;
add_header Content-Security-Policy "default-src 'self'; script-src 'self'; object-src 'none';" always;

# Hapus header yang mengekspos informasi
more_clear_headers Server;
more_clear_headers X-Powered-By;
```

> **Catatan:** `more_clear_headers` memerlukan modul `ngx_headers_more`. Alternatif tanpa modul tambahan: gunakan `proxy_hide_header` jika Nginx sebagai reverse proxy.

Install modul `ngx_headers_more`:

```bash
sudo apt install libnginx-mod-http-headers-more-filter
```

### 3.5 Batasi Metode HTTP

Di dalam file `/etc/nginx/sites-available/default` di blok `server { }`:

```nginx
# Izinkan hanya GET, POST, HEAD
if ($request_method !~ ^(GET|POST|HEAD)$) {
    return 405;
}
```

### 3.6 Proteksi File Sensitif

Di dalam file `/etc/nginx/sites-available/default` di blok `server { }`:

```nginx
# Blokir akses ke file tersembunyi (.git, .env, dll.)
location ~ /\. {
    deny all;
    access_log off;
    log_not_found off;
}

# Blokir file konfigurasi dan backup
location ~* \.(env|log|bak|sql|config|conf|ini|sh|git)$ {
    deny all;
}

# Blokir akses ke wp-config.php (jika WordPress)
location = /wp-config.php {
    deny all;
}
```

### 3.7 Batasi Ukuran Request dan Timeout

Di dalam file `nginx.conf` di blok `http { }`:

```nginx
# Batas ukuran body upload (10MB)
client_max_body_size 10m;

# Timeout konfigurasi
client_body_timeout 12;
client_header_timeout 12;
keepalive_timeout 15;
send_timeout 10;

# Buffer sizes (cegah buffer overflow attacks)
client_body_buffer_size 1k;
client_header_buffer_size 1k;
large_client_header_buffers 2 1k;
```

### 3.8 Rate Limiting

Di dalam blok `http { }`, definisikan zone:

```nginx
# Definisikan zone rate limiting
limit_req_zone $binary_remote_addr zone=general:10m rate=10r/s;
limit_req_zone $binary_remote_addr zone=login:10m rate=1r/s;
limit_conn_zone $binary_remote_addr zone=conn_limit:10m;
```

Terapkan di blok `server { }` atau `location { }`:

```nginx
server {
    # Rate limiting umum
    limit_req zone=general burst=20 nodelay;
    limit_conn conn_limit 10;

    # Rate limiting ketat untuk endpoint login
    location /login {
        limit_req zone=login burst=5 nodelay;
        limit_req_status 429;
        # ... proxy_pass atau root
    }
}
```

### 3.9 Konfigurasi Logging Nginx

Di dalam blok `http { }`:

```nginx
# Format log yang informatif
log_format main '$remote_addr - $remote_user [$time_local] "$request" '
                '$status $body_bytes_sent "$http_referer" '
                '"$http_user_agent" "$http_x_forwarded_for"';

access_log /var/log/nginx/access.log main;
error_log  /var/log/nginx/error.log warn;
```

Log rotation:

```bash
sudo nano /etc/logrotate.d/nginx
```

```
/var/log/nginx/*.log {
    daily
    missingok
    rotate 14
    compress
    delaycompress
    notifempty
    create 640 nginx adm
    sharedscripts
    postrotate
        nginx -s reopen
    endscript
}
```

### 3.10 Nonaktifkan Autoindex

Di dalam blok `server { }` atau `location { }`:

```nginx
autoindex off;
```

### 3.11 Konfigurasi Nginx sebagai Reverse Proxy (Tambahan)

Jika Nginx digunakan sebagai reverse proxy:

```nginx
location / {
    proxy_pass http://ip-backend:port;
    proxy_set_header Host $host;
    proxy_set_header X-Real-IP $remote_addr;
    proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
    proxy_set_header X-Forwarded-Proto $scheme;

    # Sembunyikan header backend
    proxy_hide_header X-Powered-By;
    proxy_hide_header Server;

    # Timeout proxy
    proxy_connect_timeout 60s;
    proxy_send_timeout 60s;
    proxy_read_timeout 60s;
}
```

### 3.12 Permission File Konfigurasi Nginx

```bash
# Konfigurasi hanya dapat dibaca oleh root
sudo chown -R root:root /etc/nginx/
sudo chmod -R 644 /etc/nginx/*.conf
sudo chmod 755 /etc/nginx/ /etc/nginx/conf.d/ /etc/nginx/sites-available/ /etc/nginx/sites-enabled/

# Root document web
sudo chmod -R 755 /var/www/html/
sudo find /var/www/html/ -type f -exec chmod 644 {} \;
```

### 3.13 Verifikasi Konfigurasi Nginx

```bash
sudo nginx -t
sudo systemctl reload nginx

# Uji security headers
curl -I https://your-domain.com
```

---

## Bagian 4: Verifikasi dan Pengujian

### 4.1 Uji dengan curl

```bash
# Cek headers keamanan
curl -sI https://your-domain.com | grep -E "(X-Frame|X-Content|Strict-Transport|X-XSS|Server:|X-Powered)"

# Uji TRACE method (harus ditolak)
curl -X TRACE https://your-domain.com

# Uji akses file sensitif (harus 403)
curl https://your-domain.com/.env
curl https://your-domain.com/.git/config
```

### 4.2 Alat Pengujian Eksternal

Berikut alat yang disarankan untuk audit setelah hardening:

| Alat | Fungsi | URL |
|------|--------|-----|
| Mozilla Observatory | Uji security headers | https://observatory.mozilla.org |
| SSL Labs | Uji kekuatan SSL/TLS | https://ssllabs.com/ssltest |
| Security Headers | Validasi HTTP headers | https://securityheaders.com |
| Nikto | Web server scanner | `nikto -h https://your-domain.com` |

### 4.3 Audit Dengan Nikto

```bash
sudo apt install nikto -y
nikto -h https://your-domain.com -ssl
```

---

## Bagian 5: Anti PHP Shell & Anti bypass disable function

Edit file `php.ini`:

```
# Sesuaikan versi php
sudo nano /etc/php/8.1/apache2/php.ini
```

Tambahkan / edit baris berikut:

```php
disable_functions = curl_multi_exec, popen, passthru, exec, popen, symlink, proc_open, shell_exec, show_source, allow_url_fopen, system, passthru, parse_ini_file, show_source, exec, proc_open, php_uname, posix_getpwuid, setenv, main, apache_setenv, putenv, mail, link, mb_send_mail,pcntl_alarm,pcntl_fork,pcntl_waitpid,pcntl_wait,pcntl_wifexited,pcntl_wifstopped,pcntl_wifsignaled,pcntl_wifcontinued,pcntl_wexitstatus,pcntl_wtermsig,pcntl_wstopsig,pcntl_signal,pcntl_signal_get_handler,pcntl_signal_dispatch,pcntl_get_last_error,pcntl_strerror,pcntl_sigprocmask,pcntl_sigwaitinfo,pcntl_sigtimedwait,pcntl_exec,pcntl_getpriority,pcntl_setpriority,pcntl_async_signals,pcntl_unshare,phpinfo

open_basedir = /var/www/html

# Atau jika menggunakan phpMyAdmin
open_basedir = "/var/www/html:/usr/share/phpmyadmin:/usr/share/php"
```

Jika menggunakan virtualhost, tambahkan seperti berikut:

```
<VirtualHost *:443>
        #Konfigurasi lainnya ..

        <Directory /home/blog>
            php_admin_value open_basedir /home/blog
        </Directory>
        <Directory /home/sample>
            php_admin_value upload_tmp_dir /home/blog
        </Directory>
</VirtualHost>
```

---

## Bagian 6: Checklist Akhir

Setelah semua konfigurasi diterapkan, verifikasi poin-poin berikut:

```
[ ] Versi server tidak terlihat di response header
[ ] Directory listing dinonaktifkan
[ ] Modul tidak perlu sudah di-disable
[ ] SSL menggunakan TLS 1.2+ dengan cipher kuat
[ ] Semua security headers terpasang
[ ] Metode TRACE/TRACK diblokir
[ ] Akses ke .env, .git, .htaccess ditolak (HTTP 403)
[ ] mod_security / WAF aktif dan berjalan
[ ] Rate limiting dikonfigurasi
[ ] Log rotation berjalan
[ ] Permission file konfigurasi sudah dibatasi
[ ] Uji SSL Labs mendapat grade A atau A+
[ ] Mozilla Observatory mendapat grade B+ atau lebih
[ ] Nikto tidak menemukan kerentanan kritis
[ ] Uji dengan upload shell PHP apakah masih di eksekusi/tidak
```

---

# Hardening Wordpress dan phpMyAdmin

## Bagian 1: phpMyAdmin

Ubah URI (*Uniform Resource Identifier*) default di `/etc/phpmyadmin/apache.conf`. Ganti dengan kata yang sekiranya tidak ada di Wordlists manapun:

```
# phpMyAdmin default Apache configuration

Alias /ini-halaman-phpmyadmin-rahasia-00 /usr/share/phpmyadmin

# Konfigurasi lain
```

```bash
sudo systemctl restart apache2
```

---

## Bagian 2: Wordpress

### 2.1 Aktifkan update otomatis

### 2.2 Install dan aktifkan plugin Disable XML-RPC

Masuk ke menu tambahkan plugin cari plugin **Disable XML-RPC-API**. Yang saya gunakan dibuat oleh *Amin Nazemi* dengan logo warna merah.

### 2.3 Ubah URI login wp-admin

Install dan aktifkan plugin **Easy Hide Login** oleh *WebFactory*. Settings dan isi bagian **Slug Text** dengan kata rahasia. Contoh: `ini-halaman-login-rahasia-00`.

Setelah diaktifkan, logout dan login ulang dengan menambahkan Slug tadi di akhir URL. Contoh: `https://your-domain.com/?ini-halaman-login-rahasia-00`

**Cek:** Jika mengakses halaman login menggunakan `/wp-admin` harus 404
