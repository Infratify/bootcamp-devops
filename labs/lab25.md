## LAB 25: UBAH SSL SETTINGS DAN LIHAT PERBEZAANNYA

### Prasyarat

Sebelum memulakan lab ini, pastikan anda sudah:
- ✅ Selesai Lab 24 (Cloudflare proxy sudah enabled - orange cloud)
- ✅ Domain anda boleh diakses melalui HTTP
- ✅ EC2 instance running dengan Nginx


### Objektif

Dalam lab ini, anda akan belajar:
1. Memahami 4 mod SSL yang berbeza di Cloudflare
2. Menguji setiap mod SSL dan lihat perbezaannya
3. Memahami encryption flow untuk setiap mod
4. Troubleshoot SSL issues
5. Pilih mod SSL yang sesuai untuk production

---

## Bahagian 1: Memahami SSL/TLS Modes

### 4 Mod SSL di Cloudflare

#### 1. Off (Tidak Disyorkan)

```
Browser ----HTTP----> Cloudflare ----HTTP----> Server
        (Tiada SSL)              (Tiada SSL)
```

**Ciri-ciri:**
- ❌ Tiada encryption langsung
- ❌ Data boleh dibaca oleh sesiapa sahaja
- ❌ Browser akan tunjuk "Not Secure"
- ❌ Sangat tidak selamat

**Bila guna:** Jangan guna!

---

#### 2. Flexible (Mudah, Kurang Selamat)

```
Browser ----HTTPS----> Cloudflare ----HTTP----> Server
        (Encrypted)              (Not encrypted)
```

**Ciri-ciri:**
- ✅ Connection antara browser dan Cloudflare encrypted
- ❌ Connection antara Cloudflare dan server TIDAK encrypted
- ✅ Browser tunjuk padlock icon (walaupun tidak sepenuhnya selamat)
- ✅ Senang setup - tidak perlu SSL certificate di server
- ⚠️ Data antara Cloudflare dan server boleh dibaca (man-in-the-middle risk)

**Bila guna:** Testing, temporary setup, legacy systems

---

#### 3. Full (Sederhana Selamat)

```
Browser ----HTTPS----> Cloudflare ----HTTPS----> Server
        (Encrypted)              (Encrypted, self-signed OK)
```

**Ciri-ciri:**
- ✅ End-to-end encryption
- ⚠️ Server certificate tidak perlu valid (boleh self-signed)
- ✅ Lebih selamat dari Flexible
- ❌ Tidak verify authenticity server certificate
- ⚠️ Vulnerable to man-in-the-middle jika attacker ada fake certificate

**Bila guna:** Development, internal applications

---

#### 4. Full (Strict) (Paling Selamat - Disyorkan)

```
Browser ----HTTPS----> Cloudflare ----HTTPS----> Server
        (Encrypted)              (Encrypted, valid cert required)
```

**Ciri-ciri:**
- ✅ End-to-end encryption
- ✅ Server mesti ada valid SSL certificate
- ✅ Certificate mesti dari trusted CA atau Cloudflare Origin CA
- ✅ Paling selamat
- ✅ Best practice untuk production

**Bila guna:** Production, e-commerce, sensitive data

---

### Comparison Table

| Feature | Off | Flexible | Full | Full (Strict) |
|---------|-----|----------|------|---------------|
| **Browser → CF** | HTTP | HTTPS ✅ | HTTPS ✅ | HTTPS ✅ |
| **CF → Server** | HTTP | HTTP ❌ | HTTPS ✅ | HTTPS ✅ |
| **Server Cert Required** | No | No | Yes (self-signed OK) | Yes (valid only) ✅ |
| **Padlock Icon** | ❌ | ✅ | ✅ | ✅ |
| **Security Level** | ⭐ (Very Low) | ⭐⭐ (Low) | ⭐⭐⭐ (Medium) | ⭐⭐⭐⭐⭐ (High) |
| **Setup Difficulty** | Easy | Easy | Medium | Medium-Hard |
| **Production Ready** | ❌ No | ⚠️ Not ideal | ⚠️ OK | ✅ Yes |

---

## Bahagian 2: Check Current SSL Status

### Langkah 1: Login ke Cloudflare

1. Pergi ke https://dash.cloudflare.com
2. Login dan pilih domain anda
3. Click **SSL/TLS** tab di menu sebelah kiri
4. Click **Overview** 

### Langkah 2: Check Current Mode

Di bahagian atas page, anda akan nampak **SSL/TLS encryption mode**

**Current mode (dari Lab 24)**: Kemungkinan **Flexible** atau **Off**

### Langkah 3: Note Current Settings

Catat setting semasa:
- SSL/TLS Mode: ______________
- Always Use HTTPS: ON / OFF
- Minimum TLS Version: ______________

---

## Bahagian 3: Test Mod "Off" (Warning!)

> **AMARAN**: Mod ini sangat tidak selamat! Hanya untuk tujuan pembelajaran sahaja. Jangan guna untuk production!

### Langkah 1: Set ke Mode "Off"

1. Di SSL/TLS tab
2. Section **SSL/TLS encryption mode**
3. Click pada **Off**
4. Confirmation popup akan muncul - click **Confirm**
5. Tunggu 10-30 saat untuk changes propagate

### Langkah 2: Disable "Always Use HTTPS"

1. Scroll ke bawah atau pergi ke **Edge Certificates**
2. Cari **Always Use HTTPS**
3. Toggle ke **OFF** (jika ON)

### Langkah 3: Test dari Browser

1. Buka browser
2. Clear cache atau guna Incognito mode
3. Pergi ke: `http://example.com` (HTTP, bukan HTTPS)

**Expected result:**
```
Welcome to nginx!
```

4. Perhatikan address bar:
   - ❌ Tiada padlock icon
   - ⚠️ Tulisan "Not secure" atau ikon warning
   - URL: `http://example.com` (bukan https)

### Langkah 4: Cuba Access dengan HTTPS

Pergi ke: `https://example.com`

**Expected result:**
- Error: "This site can't provide a secure connection"
- Atau redirect ke HTTP version

### Langkah 5: Check Headers

```bash
# Check HTTP headers
curl -I http://example.com
```

**Expected output:**
```
HTTP/1.1 200 OK
Server: cloudflare
CF-RAY: 8b1234567890-KUL
```

*Tiada HTTPS, tiada encryption*

---

## Bahagian 4: Test Mod "Flexible"

### Langkah 1: Set ke Mode "Flexible"

1. Balik ke SSL/TLS tab
2. Click pada **Flexible**
3. Tunggu 10-30 saat

### Langkah 2: Enable "Always Use HTTPS"

1. Pergi ke **Edge Certificates**
2. Toggle **Always Use HTTPS** ke **ON**

### Langkah 3: Test dari Browser

1. Clear browser cache atau guna Incognito
2. Pergi ke: `http://example.com`

**Expected result:**
- ✅ Auto redirect ke `https://example.com`
- ✅ Padlock icon muncul 🔒
- ✅ URL: `https://example.com`

3. Click pada padlock icon
4. Click "Certificate is valid" atau "Connection is secure"
5. Anda akan nampak certificate dari Cloudflare

### Langkah 4: Check Certificate Details

```bash
# Check SSL certificate
openssl s_client -connect example.com:443 -servername example.com < /dev/null | grep issuer
```

**Example Expected output:**
```
issuer=C=US, O=Google Trust Services, CN=WE1
```

*Certificate dari Cloudflare (*Google)*


### Langkah 5: Verify Server Connection

Di server, check Nginx access log:

```bash
# Connect ke EC2 via SSM/SSH


# Check Nginx logs
sudo tail -f /var/log/nginx/access.log
```

Buat request dari browser, anda akan nampak:
```
104.21.x.x - - [10/Nov/2025:10:00:00 +0000] "GET / HTTP/1.1" 200
```

*Request dari Cloudflare dalam HTTP biasa*

---

## Bahagian 5: Test Mod "Full" (Perlu Setup Certificate)

### Langkah 1: Generate Self-Signed Certificate

Connect ke EC2 instance:

```bash
# Generate self-signed certificate (valid 365 hari)
sudo openssl req -x509 -nodes -days 365 -newkey rsa:2048 \
  -keyout /etc/ssl/private/nginx-selfsigned.key \
  -out /etc/ssl/certs/nginx-selfsigned.crt \
  -subj "/C=MY/ST=Selangor/L=KualaLumpur/O=MyCompany/CN=example.com"
```

**Penjelasan:**
- `-x509`: Create self-signed certificate
- `-nodes`: No password
- `-days 365`: Valid for 1 year
- `-newkey rsa:2048`: 2048-bit RSA key
- `-subj`: Certificate details (ganti dengan maklumat anda)

### Langkah 2: Configure Nginx untuk SSL

Edit Nginx configuration:

```bash
# Edit default config
sudo nano /etc/nginx/sites-available/default
```

Tambah SSL configuration:

```nginx
server {
    listen 80;
    listen 443 ssl;
    server_name example.com www.example.com;

    # SSL Certificate
    ssl_certificate /etc/ssl/certs/nginx-selfsigned.crt;
    ssl_certificate_key /etc/ssl/private/nginx-selfsigned.key;

    # SSL Settings
    ssl_protocols TLSv1.2 TLSv1.3;

    root /var/www/html;
    index index.html index.htm index.nginx-debian.html;

    location / {
        try_files $uri $uri/ =404;
    }
}
```

**Ganti `example.com` dengan domain anda!**

### Langkah 3: Test Nginx Configuration

```bash
# Test syntax
sudo nginx -t
```

**Expected output:**
```
nginx: configuration file /etc/nginx/nginx.conf syntax is ok
nginx: configuration file /etc/nginx/nginx.conf test is successful
```

### Langkah 4: Reload Nginx

```bash
# Reload Nginx
sudo systemctl reload nginx

# Check status
sudo systemctl status nginx
```

### Langkah 5: Update Security Group

Pastikan Security Group allow port 443:

1. Pergi ke EC2 → Security Groups
2. Pilih security group untuk instance anda
3. Edit Inbound rules
4. Pastikan ada rule:
   - **Type**: HTTPS
   - **Port**: 443
   - **Source**: 0.0.0.0/0

### Langkah 6: Test SSL di Server

```bash
# Test SSL locally
curl -Ik https://localhost
```

**Expected output:**
```
HTTP/1.1 200 OK
Server: nginx/1.18.0
```

### Langkah 7: Set Cloudflare ke Mode "Full"

1. Pergi ke Cloudflare Dashboard → SSL/TLS → Overview → Configure
2. Click pada **Full**
3. Tunggu 10-30 saat

### Langkah 8: Test dari Browser

1. Clear cache atau guna Incognito
2. Pergi ke: `https://example.com`

**Expected result:**
- ✅ Website boleh diakses
- ✅ Padlock icon 🔒
- ✅ End-to-end encryption


---

## Bahagian 6: Test Mod "Full (Strict)" (Recommended)

### Langkah 1: Generate Cloudflare Origin Certificate

Untuk Full (Strict), kita perlu valid certificate. Guna Cloudflare Origin Certificate (free!):

1. Di Cloudflare Dashboard → SSL/TLS
2. Click tab **Origin Server**
3. Click **Create Certificate**

### Langkah 2: Configure Certificate

1. **Private key type**: RSA (2048)
2. **Hostnames**:
   - `example.com`
   - `*.example.com` (untuk subdomain)
3. **Certificate Validity**: 15 years (pilih paling lama)
4. Click **Create**

### Langkah 3: Copy Certificate dan Key

Anda akan nampak 2 box:

**Origin Certificate:**
```
-----BEGIN CERTIFICATE-----
MIIEpDCCA4ygAwIBAgIUXxxx...
-----END CERTIFICATE-----
```

**Private Key:**
```
-----BEGIN RSA PRIVATE KEY-----
MIIEowIBAAKCAQEA...
-----END RSA PRIVATE KEY-----
```

**PENTING**: Copy both! Page ini hanya show once.

### Langkah 4: Install Certificate di Server

Connect ke EC2:

```bash
# Create certificate file
sudo nano /etc/ssl/certs/cloudflare-origin.crt
```

Paste **Origin Certificate**, kemudian save (Ctrl+X, Y, Enter)

```bash
# Create private key file
sudo nano /etc/ssl/private/cloudflare-origin.key
```

Paste **Private Key**, kemudian save

```bash
# Set proper permissions
sudo chmod 644 /etc/ssl/certs/cloudflare-origin.crt
sudo chmod 600 /etc/ssl/private/cloudflare-origin.key
```

### Langkah 5: Update Nginx Configuration

```bash
sudo nano /etc/nginx/sites-available/default
```

Update SSL certificate paths:

```nginx
server {
    listen 80;
    listen 443 ssl;
    server_name example.com www.example.com;

    # Cloudflare Origin Certificate
    ssl_certificate /etc/ssl/certs/cloudflare-origin.crt;
    ssl_certificate_key /etc/ssl/private/cloudflare-origin.key;

    # SSL Settings
    ssl_protocols TLSv1.2 TLSv1.3;
    ssl_ciphers HIGH:!aNULL:!MD5;
    ssl_prefer_server_ciphers on;

    root /var/www/html;
    index index.html index.htm index.nginx-debian.html;

    location / {
        try_files $uri $uri/ =404;
    }
}
```

### Langkah 6: Test dan Reload Nginx

```bash
# Test configuration
sudo nginx -t

# Reload Nginx
sudo systemctl reload nginx

# Check status
sudo systemctl status nginx
```

### Langkah 7: Set Cloudflare ke Mode "Full (Strict)"

1. Pergi ke Cloudflare Dashboard → SSL/TLS → Overview → Configure
2. Click pada **Full (strict)**
3. Tunggu 10-30 saat

### Langkah 8: Test dari Browser

1. Clear cache atau guna Incognito mode
2. Pergi ke: `https://example.com`

**Expected result:**
- ✅ Website accessible
- ✅ Padlock icon 🔒
- ✅ End-to-end encryption dengan valid certificate
- ✅ Production-ready security!

### Langkah 9: Verify Certificate Chain

Click padlock icon → Certificate details:
- **Issued to**: example.com
- **Issued by**: Cloudflare Inc ECC CA-3
- **Valid from**: [date]
- **Valid until**: [date]

```bash
# Check certificate via command line
openssl s_client -connect example.com:443 -servername example.com < /dev/null 2>/dev/null | openssl x509 -noout -dates
```

**Output:**
```
notBefore=Nov 10 00:00:00 2025 GMT
notAfter=Nov 10 00:00:00 2040 GMT
```

---

## Bahagian 7: Comparison Testing

### Test 1: Security Check

Pergi ke: https://www.ssllabs.com/ssltest/

1. Masukkan domain: `example.com`
2. Click **Submit**
3. Tunggu 2-5 minit untuk analysis complete

**Expected grade:**
- Off: F (Fail)
- Flexible: T (Certificate not trusted by browsers directly)
- Full: A or A- (depends on configuration)
- Full (Strict): A or A+ (dengan proper configuration)



---

## Bahagian 8: Troubleshooting

### ❌ Problem 1: Error 525 - SSL Handshake Failed

**Sebab:**
- Nginx tidak listen di port 443
- Certificate tidak configured betul
- SSL module tidak enabled

**Solusi:**

```bash
# Check Nginx listening ports
sudo netstat -tlnp | grep nginx

# Expected output should show:
# tcp  0  0  0.0.0.0:80    0.0.0.0:*  LISTEN
# tcp  0  0  0.0.0.0:443   0.0.0.0:*  LISTEN

# Check Nginx error logs
sudo tail -50 /var/log/nginx/error.log

# Test SSL configuration
sudo nginx -t

# Restart Nginx
sudo systemctl restart nginx
```

### ❌ Problem 2: Error 526 - Invalid SSL Certificate

**Sebab:**
- Mode = Full (Strict) tetapi certificate tidak valid
- Certificate expired
- Certificate hostname tidak match

**Solusi:**

1. Check certificate validity:
```bash
openssl x509 -in /etc/ssl/certs/cloudflare-origin.crt -noout -dates
```

2. Check certificate hostname:
```bash
openssl x509 -in /etc/ssl/certs/cloudflare-origin.crt -noout -text | grep DNS
```

3. Generate new certificate jika perlu (refer Langkah 1-6 di Bahagian 6)

### ❌ Problem 3: ERR_TOO_MANY_REDIRECTS

**Sebab:**
- Cloudflare mode = Full/Full (Strict) tetapi Nginx force redirect HTTP → HTTPS
- Infinite redirect loop

**Solusi:**

Remove redirect rules dari Nginx config:

```bash
sudo nano /etc/nginx/sites-available/default
```

**Remove lines like:**
```nginx
# Don't do this with Cloudflare proxy!
if ($scheme != "https") {
    return 301 https://$server_name$request_uri;
}
```

Cloudflare sudah handle redirect dengan "Always Use HTTPS" setting.

### ❌ Problem 4: Mixed Content Warning

**Sebab:**
- HTTPS page loading HTTP resources (images, scripts, CSS)

**Solusi:**

Check page source dan ensure semua resources guna HTTPS:

```html
<!-- Bad -->
<img src="http://example.com/image.jpg">

<!-- Good -->
<img src="https://example.com/image.jpg">

<!-- Better (protocol-relative) -->
<img src="//example.com/image.jpg">
```

---

## Bahagian 9: Best Practices

### Untuk Production

1. **SSL Mode**: Guna **Full (Strict)**
   - End-to-end encryption
   - Valid certificate required
   - Best security

2. **Certificate**: Guna Cloudflare Origin Certificate
   - Free
   - Valid 15 years
   - Easy renewal

3. **TLS Version**: Minimum TLS 1.2
   - Balance security dan compatibility
   - Block outdated protocols

4. **HSTS**: Enable dengan long Max Age
   - Prevent SSL stripping
   - Better SEO ranking

5. **Monitor**: Regular check SSL Labs test
   - Target: A or A+ rating
   - Fix issues promptly

### Untuk Development/Testing

1. **SSL Mode**: Full atau Flexible OK
   - Easier setup
   - Self-signed cert acceptable

2. **Certificate**: Self-signed OK
   - Quick setup
   - No cost

3. **HSTS**: Don't enable
   - Allow HTTP testing
   - Easier troubleshooting

---

## Pengetahuan Tambahan

### SSL vs TLS

**SSL (Secure Sockets Layer)**:
- Older protocol
- SSL 2.0 dan 3.0 deprecated (insecure)
- Terminology masih digunakan

**TLS (Transport Layer Security)**:
- Modern replacement for SSL
- TLS 1.2 dan 1.3 recommended
- More secure and efficient

**Note**: Bila orang kata "SSL", usually maksudnya TLS. Cloudflare guna TLS, tetapi interface kata "SSL/TLS".

### Certificate Types

1. **Self-signed Certificate**:
   - Free
   - Not trusted by browsers (warning displayed)
   - OK untuk development
   - Boleh guna dengan Cloudflare Full mode

2. **Domain Validated (DV)**:
   - Verify domain ownership only
   - Quick issuance (minutes)
   - Free (Let's Encrypt) or paid
   - Trusted by browsers

3. **Organization Validated (OV)**:
   - Verify organization identity
   - More rigorous validation
   - Show organization name in certificate
   - Paid

4. **Extended Validation (EV)**:
   - Highest level validation
   - Green address bar (older browsers)
   - Show organization name prominently
   - Expensive
   - Less common now

5. **Cloudflare Origin Certificate**:
   - Free from Cloudflare
   - Only trusted by Cloudflare (not browsers directly)
   - Perfect untuk Cloudflare proxy
   - Valid up to 15 years

### Common SSL Errors

| Error Code | Meaning | Typical Cause |
|------------|---------|---------------|
| **525** | SSL Handshake Failed | Server tidak listen port 443 |
| **526** | Invalid SSL Certificate | Cert expired, hostname mismatch |
| **ERR_CERT_AUTHORITY_INVALID** | Certificate not trusted | Self-signed cert, invalid CA |
| **ERR_CERT_COMMON_NAME_INVALID** | Hostname mismatch | Cert untuk wrong domain |
| **ERR_CERT_DATE_INVALID** | Certificate expired | Past expiry date |
| **ERR_TOO_MANY_REDIRECTS** | Redirect loop | Wrong SSL mode + Nginx redirect |
| **Mixed Content** | HTTP on HTTPS page | Loading insecure resources |

---

🎉 **Tahniah!** Anda sudah faham perbezaan 4 mod SSL di Cloudflare dan berjaya setup Full (Strict) mode untuk production-ready security!

**Recommended setup:**
- ✅ SSL Mode: Full (Strict)
- ✅ Certificate: Cloudflare Origin Certificate
- ✅ Always Use HTTPS: Enabled
- ✅ Minimum TLS: 1.2
- ✅ HSTS: Enabled (production)
- ✅ SSL Labs Grade: A or A+
