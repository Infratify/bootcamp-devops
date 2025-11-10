## LAB 27: BUAT FORWARDING RULE MENGGUNAKAN CLOUDFLARE

### Prasyarat

Sebelum memulakan lab ini, pastikan anda sudah:
- ✅ Domain sudah configured di Cloudflare (dari Lab 23-25)
- ✅ Cloudflare account active
- ✅ Access ke Cloudflare Dashboard

### Objektif

Dalam lab ini, anda akan belajar:
1. Memahami Cloudflare Page Rules
2. Buat subdomain baru untuk forwarding
3. Configure forwarding rule di Cloudflare
4. Setup DNS record untuk subdomain
5. Test redirect berfungsi dengan betul
6. Explore limits dan best practices

---

## Bahagian 1: Memahami Cloudflare Page Rules

### Apa itu Page Rules?

Cloudflare Page Rules membenarkan anda configure behavior khusus untuk URLs tertentu.

**Keupayaan:**
- ✅ Forwarding URL (redirects)
- ✅ Custom caching
- ✅ Security settings
- ✅ SSL settings
- ✅ Performance optimization

**Untuk lab ini, kita fokus pada Forwarding URL.**

### Types of Forwarding

#### 301 - Permanent Redirect

```
User → wp.domain.com
    ↓
Cloudflare: "Permanently moved to wordpress.org/download/"
    ↓
User browser → wordpress.org/download/
    ↓
Address bar: wordpress.org/download/
```

**Ciri-ciri:**
- Permanent redirect
- Search engines update index
- Browser cache the redirect
- Best untuk permanent moves

#### 302 - Temporary Redirect

```
User → wp.domain.com
    ↓
Cloudflare: "Temporarily at wordpress.org/download/"
    ↓
User browser → wordpress.org/download/
    ↓
Address bar: wordpress.org/download/
```

**Ciri-ciri:**
- Temporary redirect
- Search engines keep original URL
- Not cached aggressively
- Best untuk temporary situations

### Comparison

| Feature | 301 (Permanent) | 302 (Temporary) |
|---------|----------------|-----------------|
| **SEO** | Transfer ranking | Keep original ranking |
| **Cache** | Heavily cached | Less cached |
| **Use Case** | Domain migration, permanent moves | Maintenance, A/B testing |
| **Reversible** | Difficult (cached) | Easy |

**Untuk lab ini, kita akan guna 301** kerana ini adalah permanent forward.

---

## Bahagian 2: Free Plan Limitations

### Cloudflare Page Rules Limits

| Plan | Page Rules Limit |
|------|-----------------|
| **Free** | 3 rules |
| **Pro** | 20 rules |
| **Business** | 50 rules |
| **Enterprise** | 125 rules |

**Important:** Free plan hanya dapat **3 Page Rules**. Guna dengan bijak!

### Tips untuk Free Plan

1. **Prioritize**: Guna untuk redirects paling penting sahaja
2. **Combine**: Guna wildcard untuk cover multiple URLs
3. **Alternative**: Guna Bulk Redirects (Beta) untuk more redirects

---

## Bahagian 3: Setup DNS Record

Sebelum buat Page Rule, kita perlu setup DNS record dahulu.

### Langkah 1: Login ke Cloudflare

1. Pergi ke https://dash.cloudflare.com
2. Login dengan account anda
3. Pilih domain anda dari dashboard

### Langkah 2: Pergi ke DNS Settings

1. Click tab **DNS** di menu sebelah kiri
2. Anda akan nampak senarai DNS records sedia ada

### Langkah 3: Tambah A Record untuk Subdomain

Click **Add record**

Masukkan maklumat:

| Field | Value | Keterangan |
|-------|-------|------------|
| **Type** | `A` | Record type |
| **Name** | `wp` | Subdomain name |
| **IPv4 address** | `192.0.2.1` | Dummy IP (akan redirect sebelum sampai server) |
| **Proxy status** | ☁️ **Proxied** (orange cloud) | PENTING: Mesti proxied! |
| **TTL** | `Auto` | Automatic |

**PENTING Notes:**
- ✅ **Proxy status MESTI Proxied (orange cloud)** - Page Rules hanya berfungsi dengan proxied records
- ✅ IP address boleh guna dummy IP seperti `192.0.2.1` kerana traffic akan di-redirect sebelum sampai server
- ✅ Atau boleh guna server IP anda (54.x.x.x) sebagai fallback

Click **Save**

### Langkah 4: Verify DNS Record

Anda sepatutnya nampak record baru:

```
Type    Name    Content        Proxy Status
A       wp      192.0.2.1      Proxied (orange cloud)
```

### Langkah 5: Tunggu DNS Propagate

Tunggu 1-5 minit untuk DNS propagate.

Semak propagation:
```bash
dig +short wp.domain.com
```

**Expected:** Cloudflare IPs (104.x.x.x atau 172.x.x.x)

Atau check online: https://dnschecker.org

---

## Bahagian 4: Buat Page Rule untuk Forwarding

### Langkah 1: Pergi ke Page Rules

1. Di Cloudflare Dashboard, click tab **Rules** di menu kiri
2. Click **Page Rules**
3. Atau pergi ke: **Rules → Page Rules**

### Langkah 2: Check Current Usage

Di bahagian atas page, anda akan nampak:
```
Page Rules (X/3)
```

**X = Berapa rules yang dah digunakan**

Jika sudah 3/3, anda perlu delete satu rule untuk tambah yang baru.

### Langkah 3: Create Page Rule

Click **Create Page Rule** button

### Langkah 4: Configure URL Pattern

Di field **If the URL matches:**

Masukkan:
```
wp.domain.com/*
```

**Ganti `domain.com` dengan domain anda!**

**Penjelasan pattern:**
- `wp.domain.com` - Subdomain anda
- `/*` - Wildcard untuk match semua paths

**Contoh URLs yang akan match:**
- ✅ `wp.domain.com/`
- ✅ `wp.domain.com/anything`
- ✅ `wp.domain.com/path/to/page`

### Langkah 5: Configure Setting

1. Click dropdown **+ Add a Setting**
2. Pilih **Forwarding URL**

3. Anda akan nampak 2 fields:

**Status Code:**
- Pilih: **301 - Permanent Redirect**

**Destination URL:**
- Masukkan: `https://wordpress.org/download/`

**PENTING:** Pastikan URL lengkap dengan `https://`

### Langkah 6: Preview Configuration

Page Rule anda sepatutnya nampak macam ini:

```
If the URL matches:
  wp.domain.com/*

Then the settings are:
  Forwarding URL: 301 - Permanent Redirect
  Destination URL: https://wordpress.org/download/
```

### Langkah 7: Save Page Rule

1. Review semua settings
2. Click **Save and Deploy** button
3. Rule akan active dalam beberapa saat

### Langkah 8: Verify Page Rule Active

Di Page Rules list, anda akan nampak rule baru:

```
Rule 1: wp.domain.com/*
Status: Active (toggle should be ON)
Forwarding URL: 301 to https://wordpress.org/download/
```

---

## Bahagian 5: Test Redirect

### Langkah 1: Tunggu Sebentar

Tunggu 10-30 saat untuk rule propagate ke Cloudflare edge servers.

### Langkah 2: Test dari Browser

1. Buka browser
2. Clear cache atau guna Incognito mode
3. Pergi ke: `https://wp.domain.com`

**Expected result:**
- ✅ Browser automatically redirect ke `https://wordpress.org/download/`
- ✅ Address bar berubah ke `wordpress.org/download/`
- ✅ WordPress download page loaded
- ✅ Redirect sangat cepat (instant, dari Cloudflare edge)

### Langkah 3: Test Different Paths

Test dengan paths berbeza:

1. `https://wp.domain.com/` → redirects
2. `https://wp.domain.com/test` → redirects
3. `https://wp.domain.com/any/path` → redirects

**Semua sepatutnya redirect ke same destination!**

### Langkah 4: Test dengan HTTP

Pergi ke: `http://wp.domain.com`

**Expected:**
- ✅ Automatic redirect ke `https://wordpress.org/download/`
- ✅ Works untuk HTTP dan HTTPS

### Langkah 5: Test dari Command Line

```bash
# Test redirect
curl -I https://wp.domain.com
```

**Expected output:**
```
HTTP/2 301
location: https://wordpress.org/download/
server: cloudflare
cf-cache-status: DYNAMIC
```

**Perhatikan:**
- Status: `301` (Permanent Redirect)
- `location:` header tunjuk destination
- `server: cloudflare` (redirect handled at edge)

### Langkah 6: Test Follow Redirect

```bash
# Follow redirect dan lihat final destination
curl -L -I https://wp.domain.com
```

**Expected output:**
```
HTTP/2 301
location: https://wordpress.org/download/

HTTP/2 200
server: nginx
content-type: text/html
```

✅ Perfect! Redirect berfungsi dengan betul!

---

## Bahagian 6: Multiple Redirects (Optional)

Jika anda ada lebih banyak Page Rules available, anda boleh buat multiple redirects.

### Contoh: GitHub Profile Shortlink

**Langkah 1: Tambah DNS Record**

Di Cloudflare DNS:
```
Type: A
Name: github
IPv4 address: 192.0.2.1
Proxy: Proxied (orange cloud)
```

**Langkah 2: Create Page Rule**

```
URL Pattern: github.domain.com/*
Setting: Forwarding URL
Status Code: 301 - Permanent Redirect
Destination: https://github.com/your-username
```

**Langkah 3: Save dan Test**

```bash
curl -I https://github.domain.com
# Should redirect to github.com/your-username
```

### Contoh: LinkedIn Profile Shortlink

**DNS:**
```
Type: A
Name: linkedin
IPv4: 192.0.2.1
Proxy: Proxied
```

**Page Rule:**
```
URL: linkedin.domain.com/*
Forward: 301 to https://linkedin.com/in/your-profile
```

### Summary Rules Usage

```
Page Rules (3/3)

1. wp.domain.com/* → wordpress.org/download/
2. github.domain.com/* → github.com/your-username
3. linkedin.domain.com/* → linkedin.com/in/your-profile
```


## Bahagian 6: Use Cases

### 1. Social Media Profiles

Buat easy-to-remember links:

```
twitter.domain.com → twitter.com/username
fb.domain.com → facebook.com/page
ig.domain.com → instagram.com/profile
```

### 2. Documentation Shortcuts

```
docs.domain.com → gitbook.com/docs
help.domain.com → zendesk.com/support
api.domain.com → swagger.io/api-docs
```

### 3. Download Links

```
download.domain.com → github.com/releases/latest
app.domain.com → apps.apple.com/app-id
```

### 4. Campaign URLs

```
promo.domain.com → domain.com/?utm_source=promo
blackfriday.domain.com → domain.com/sale
```

### 5. Affiliate Links

```
shop.domain.com → amazon.com/shop/affiliate-id
store.domain.com → shopify-store.com?ref=code
```

### 6. Event Registration

```
register.domain.com → eventbrite.com/event-id
webinar.domain.com → zoom.us/meeting-id
```
