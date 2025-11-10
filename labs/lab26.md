## LAB 26: SETUP NGINX VIRTUAL HOST UNTUK PRODUCTION DAN STAGING


### Objektif

Dalam lab ini, anda akan belajar:
1. Fork dan clone repository HTML template
2. Setup 2 branches (main untuk production, staging untuk staging)
3. Ubah content untuk setiap environment menggunakan VSCode
4. Deploy code ke server dalam directories berbeza
5. Configure Nginx virtual hosts untuk 2 domains
6. Setup DNS records untuk subdomain
7. Test kedua-dua environment

---

## Bahagian 1: Setup Repository di Local

### Langkah 1: Fork Repository

1. Pergi ke: https://github.com/Infratify/html5-aerial
2. Click butang **Fork** (di kanan atas)
3. Pilih account GitHub anda
4. Click **Create fork**
5. Repository akan di-fork ke account anda: `https://github.com/[username]/html5-aerial`

### Langkah 2: Clone Repository ke Local

Buka terminal di komputer local anda:

```bash
# Clone repository yang baru di-fork
git clone https://github.com/[username]/html5-aerial.git

# Ganti [username] dengan GitHub username anda!
```

**Contoh:**
```bash
git clone https://github.com/ahmad123/html5-aerial.git
```

### Langkah 3: Buka Project di VSCode

```bash
# Masuk ke directory
cd html5-aerial

# Buka project di VSCode
code .
```

VSCode akan buka dengan struktur project:
```
html5-aerial/
├── index.html
├── README.md
└── assets/
    ├── css/
    ├── js/
    └── (fail lain)
```

### Langkah 4: Semak Current Branch

Di VSCode:
1. Lihat sudut kiri bawah - anda akan nampak current branch (main)
2. Atau semak melalui terminal di VSCode (View → Terminal):

```bash
git branch
# Output: * main
```

---

## Bahagian 2: Setup Production Branch (Main)

### Langkah 1: Semak Content Asal

Di VSCode:
1. Click pada `index.html` di panel Explorer (kiri)
2. Fail akan buka di editor
3. Cari line yang ada `<h1>This is my website</h1>` (sekitar line 35-40)

### Langkah 2: Edit untuk Production Environment

Di VSCode editor, cari line:

**Sebelum:**
```html
<h1>This is my website</h1>
```

**Selepas:**
```html
<h1>This is PRODUCTION website</h1>
```

Save fail (Ctrl+S atau Cmd+S)

### Langkah 3: Preview Perubahan

Di VSCode:
1. Klik kanan pada `index.html` di Explorer
2. Pilih **Open with Live Server** (jika ada extension)
3. Atau double-click fail `index.html` untuk buka di browser

**Expected**: Anda akan nampak heading "This is PRODUCTION website"


### Langkah 4: Commit Perubahan

Di VSCode Source Control panel:
1. Hover pada `index.html` dan click icon `+` (Stage Changes)
2. Taip commit message di kotak teks atas: `docs: update heading to PRODUCTION for main branch`
3. Click butang ✓ (Commit)

Atau melalui terminal:

```bash
# Tambah perubahan
git add index.html

# Commit
git commit -m "docs: update heading to PRODUCTION for main branch"
```

### Langkah 5: Push ke GitHub

Di VSCode:
1. Click `...` (More Actions) dalam Source Control panel
2. Click **Push**

Atau melalui terminal:

```bash
git push origin main
```

---

## Bahagian 3: Setup Staging Branch

### Langkah 1: Buat Staging Branch

Di VSCode:
1. Click pada nama branch di sudut kiri bawah (main)
2. Click **Create new branch...**
3. Taip: `staging`
4. Tekan Enter

Atau melalui terminal:

```bash
# Buat dan tukar ke staging branch
git checkout -b staging

# Verify
git branch
# Output:
# * staging
#   main
```

Sudut kiri bawah VSCode sekarang tunjuk: `staging`

### Langkah 2: Edit untuk Staging Environment

Di VSCode editor, dalam `index.html`, ubah line:

**Sebelum:**
```html
<h1>This is PRODUCTION website</h1>
```

**Selepas:**
```html
<h1>This is STAGING website</h1>
```

Save fail (Ctrl+S atau Cmd+S)

### Langkah 3: Preview Perubahan

Refresh browser atau buka `index.html` semula:

**Expected**: Anda akan nampak heading "This is STAGING website"


### Langkah 4: Commit dan Push Staging Branch

Di VSCode Source Control:
1. Stage perubahan (icon `+`)
2. Commit message: `docs: update heading to STAGING for staging branch`
3. Click ✓ (Commit)
4. Click `...` → **Push**
5. Jika ada prompt "staging has no upstream branch", click **OK** untuk push

Atau melalui terminal:

```bash
# Tambah dan commit
git add index.html
git commit -m "docs: update heading to STAGING for staging branch"

# Push staging branch
git push origin staging
```

### Langkah 5: Verify di GitHub

1. Pergi ke repository anda di GitHub
2. Click dropdown branch (default: main)
3. Anda sepatutnya nampak 2 branches:
   - `main`
   - `staging`
4. Tukar antara branches di GitHub
5. Click `index.html` untuk setiap branch
6. Verify content berbeza:
   - main: "This is PRODUCTION website"
   - staging: "This is STAGING website"


Atau di VSCode, click nama branch di sudut kiri bawah dan pilih branch.

---

## Bahagian 4: Deploy Code ke Server

### Langkah 1: Sambung ke EC2 Instance

```bash
# Melalui SSM (disyorkan)
aws ssm start-session --target i-xxxxxxxxxxxxx

# Atau melalui SSH
ssh -i your-key.pem ubuntu@your-ec2-ip
```

### Langkah 2: Install Git di Server (Jika Belum Ada)

```bash
# Semak jika git sudah installed
git --version

# Jika belum, install git
sudo apt update
sudo apt install git -y

# Verify installation
git --version
```

### Langkah 3: Buat Directories untuk Production

```bash
# Buat directory untuk production
sudo mkdir -p /var/www/www.domain.com

# Ganti 'domain.com' dengan domain anda!
# Contoh: sudo mkdir -p /var/www/www.example.com
```

### Langkah 4: Clone Main Branch (Production)

```bash
# Clone main branch ke production directory
sudo git clone -b main https://github.com/[username]/html5-aerial.git /var/www/www.domain.com

# Ganti:
# - [username] dengan GitHub username anda
# - domain.com dengan domain anda
```

**Contoh:**
```bash
sudo git clone -b main https://github.com/ahmad123/html5-aerial.git /var/www/www.example.com
```

### Langkah 5: Buat Directories untuk Staging

```bash
# Buat directory untuk staging
sudo mkdir -p /var/www/stg.domain.com

# Ganti 'domain.com' dengan domain anda!
```

### Langkah 6: Clone Staging Branch

```bash
# Clone staging branch ke staging directory
sudo git clone -b staging https://github.com/[username]/html5-aerial.git /var/www/stg.domain.com
```

**Contoh:**
```bash
sudo git clone -b staging https://github.com/ahmad123/html5-aerial.git /var/www/stg.example.com
```

### Langkah 7: Set Permissions yang Betul

```bash
# Set ownership ke www-data (Nginx user)
sudo chown -R www-data:www-data /var/www/www.domain.com
sudo chown -R www-data:www-data /var/www/stg.domain.com

# Set permissions yang sesuai
sudo chmod -R 755 /var/www/www.domain.com
sudo chmod -R 755 /var/www/stg.domain.com
```

**Ganti `domain.com` dengan domain anda!**

### Langkah 8: Verify Fail dan Content

```bash
# Semak fail production
ls -la /var/www/www.domain.com/

# Expected output:
# index.html
# README.md
# assets/

# Semak fail staging
ls -la /var/www/stg.domain.com/

# Verify heading dalam production
grep "<h1>" /var/www/www.domain.com/index.html

# Expected output: <h1>This is PRODUCTION website</h1>

# Verify heading dalam staging
grep "<h1>" /var/www/stg.domain.com/index.html

# Expected output: <h1>This is STAGING website</h1>
```

✅ Jika output betul, bermakna kedua-dua branch sudah deployed dengan content yang berbeza!

---

## Bahagian 5: Configure Nginx Virtual Hosts

### Langkah 1: Buat Configuration untuk Production

```bash
# Buat config file baru
sudo nano /etc/nginx/sites-available/www.domain.com
```

**Ganti `domain.com` dengan domain anda!**

Masukkan configuration ini:

```nginx
server {
    listen 80;
    listen 443 ssl;
    server_name www.domain.com domain.com;

    # SSL Certificate (jika sudah setup dari Lab 25)
    ssl_certificate /etc/ssl/certs/cloudflare-origin.crt;
    ssl_certificate_key /etc/ssl/private/cloudflare-origin.key;

    # SSL Settings
    ssl_protocols TLSv1.2 TLSv1.3;

    # Document root
    root /var/www/www.domain.com;
    index index.html index.htm;

    # Logging
    access_log /var/log/nginx/www.domain.com-access.log;
    error_log /var/log/nginx/www.domain.com-error.log;

    location / {
        try_files $uri $uri/ =404;
    }

}
```

**PENTING**: Ganti semua `domain.com` dengan domain anda!

Save fail (Ctrl+X, Y, Enter)

### Langkah 2: Buat Configuration untuk Staging

```bash
# Buat staging config
sudo nano /etc/nginx/sites-available/stg.domain.com
```

Masukkan configuration ini:

```nginx
server {
    listen 80;
    listen 443 ssl;
    server_name stg.domain.com;

    # SSL Certificate (jika sudah setup dari Lab 25)
    ssl_certificate /etc/ssl/certs/cloudflare-origin.crt;
    ssl_certificate_key /etc/ssl/private/cloudflare-origin.key;

    # SSL Settings
    ssl_protocols TLSv1.2 TLSv1.3;

    # Document root
    root /var/www/stg.domain.com;
    index index.html index.htm;

    # Logging
    access_log /var/log/nginx/stg.domain.com-access.log;
    error_log /var/log/nginx/stg.domain.com-error.log;

    location / {
        try_files $uri $uri/ =404;
    }
}
```

**PENTING**: Ganti semua `domain.com` dengan domain anda!

Save fail (Ctrl+X, Y, Enter)

### Langkah 3: Enable Sites

```bash
# Buang default site (jika masih ada)
sudo mv /etc/nginx/sites-enabled/default /etc/nginx/sites-enabled/default_backup

# Enable production site
sudo ln -s /etc/nginx/sites-available/www.domain.com /etc/nginx/sites-enabled/

# Enable staging site
sudo ln -s /etc/nginx/sites-available/stg.domain.com /etc/nginx/sites-enabled/
```

**Ganti `domain.com` dengan domain anda!**

### Langkah 4: Test Nginx Configuration

```bash
# Test syntax
sudo nginx -t
```

**Expected output:**
```
nginx: the configuration file /etc/nginx/nginx.conf syntax is ok
nginx: configuration file /etc/nginx/nginx.conf test is successful
```

**Jika ada error**, semak:
- Typo dalam server_name
- Missing semicolons
- Salah path untuk fail

### Langkah 5: Reload Nginx

```bash
# Reload Nginx
sudo systemctl reload nginx

# Semak status
sudo systemctl status nginx

# Pastikan ia active (running)
```

---

## Bahagian 6: Setup DNS Records di Cloudflare

### Langkah 1: Login ke Cloudflare

1. Pergi ke https://dash.cloudflare.com
2. Login dan pilih domain anda
3. Click tab **DNS**

### Langkah 2: Dapatkan Server IP Address

Dari EC2 Console atau melalui command:

```bash
# Di server, dapatkan public IP
curl ifconfig.me

# Atau
curl ipinfo.io/ip
```

Copy IP address (contoh: `54.123.45.67`)

### Langkah 3: Tambah/Update A Record untuk Production

Pastikan anda sudah ada A record untuk root domain dan www (dari Lab 23):

**Root domain:**
- Type: `A`
- Name: `@`
- Content: `54.123.45.67` (IP server anda)
- Proxy: `Proxied` (orange cloud)
- TTL: `Auto`

**WWW subdomain:**
- Type: `CNAME`
- Name: `www`
- Content: `domain.com`
- Proxy: `Proxied` (orange cloud)
- TTL: `Auto`

### Langkah 4: Tambah A Record untuk Staging

Click **Add record**

Masukkan:
- **Type**: `A`
- **Name**: `stg`
- **IPv4 address**: `54.123.45.67` (IP server yang sama)
- **Proxy status**: `Proxied` (orange cloud ☁️)
- **TTL**: `Auto`

Click **Save**

### Langkah 5: Verify DNS Records

Anda sepatutnya ada 3 records:

```
Type    Name           Content           Proxy
A       domain.com     54.123.45.67      Proxied
CNAME   www            domain.com        Proxied
A       stg            54.123.45.67      Proxied
```

### Langkah 6: Semak DNS Propagation

```bash
# Semak production domain
dig +short www.domain.com

# Semak staging subdomain
dig +short stg.domain.com
```

**Expected output**: Cloudflare IPs (104.x.x.x atau 172.x.x.x)

Atau semak secara online: https://dnschecker.org

---

## Bahagian 7: Test Kedua-dua Environment

### Langkah 1: Test Production dari Browser

1. Buka browser
2. Clear cache atau guna Incognito mode
3. Pergi ke: `https://www.domain.com` atau `https://domain.com`

**Expected result:**
- ✅ Website loaded dengan HTML5 Aerial template
- ✅ Heading: **"This is PRODUCTION website"**
- ✅ Padlock icon 🔒 (HTTPS)
- ✅ Design yang cantik dengan background image

### Langkah 2: Test Staging dari Browser

1. Buka tab baru
2. Pergi ke: `https://stg.domain.com`

**Expected result:**
- ✅ Website loaded dengan HTML5 Aerial template (design sama)
- ✅ Heading: **"This is STAGING website"**
- ✅ Padlock icon 🔒 (HTTPS)
- ✅ Design sama tetapi heading berbeza


