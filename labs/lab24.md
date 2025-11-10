## LAB 24: ENABLE CLOUDFLARE PROXY DAN LIHAT PERBEZAANNYA

### Prasyarat

Sebelum memulakan lab ini, pastikan anda sudah:
- ✅ Selesai Lab 23 (DNS record sudah setup dengan DNS only mode)
- ✅ Domain anda sudah boleh diakses dari browser
- ✅ EC2 instance masih running dengan Nginx

### Objektif

Dalam lab ini, anda akan belajar:
1. Perbezaan antara DNS only (gray cloud) vs Proxied (orange cloud)
2. Enable Cloudflare proxy untuk domain anda
3. Verify perubahan IP address
4. Lihat benefits dari Cloudflare proxy
5. Troubleshoot issues jika ada

---

## Bahagian 1: Memahami Mod Proxy

### DNS Only Mode (Gray Cloud 🔴)

**Status sekarang**: Dari Lab 23, DNS record anda dalam mode **DNS only**

Cara kerja:
```
User Browser → (DNS lookup) → Cloudflare DNS → Return real IP
User Browser → (Direct connection) → Your AWS Server
```

**Ciri-ciri:**
- Traffic direct ke server anda
- Real IP address visible kepada public
- Tiada CDN caching
- Tiada extra security layer
- Simple setup

### Proxied Mode (Orange Cloud ☁️)

**Mode yang akan kita enable**: **Proxied through Cloudflare**

Cara kerja:
```
User Browser → (DNS lookup) → Cloudflare DNS → Return Cloudflare IP
User Browser → Cloudflare Edge Server → Your AWS Server
```

**Ciri-ciri:**
- Traffic melalui Cloudflare network
- Real IP hidden (hanya Cloudflare IP visible)
- CDN caching enabled
- Extra security (DDoS protection, WAF)
- Free SSL certificate
- Slightly more complex setup

---

## Bahagian 2: Semak Current Setup

### Langkah 1: Check IP Address Sekarang

Buka terminal dan check IP address yang DNS return:

```bash
# Check IP untuk domain anda
dig +short example.com
```

**Expected output (DNS only mode):**
```
54.123.45.67
```
*Ini adalah real IP dari EC2 instance anda*

### Langkah 2: Check dari Browser

1. Buka browser
2. Pergi ke: https://www.whatsmydns.net/
3. Masukkan domain anda: `example.com`
4. Click **Search**
5. Perhatikan pada IP yang dipaparkan - ini adalah real server IP anda

### Langkah 3: Check Response Headers

```bash
# Check HTTP headers
curl -I http://example.com
```

**Expected output:**
```
HTTP/1.1 200 OK
Server: nginx/1.18.0
Date: Mon, 10 Nov 2025 10:00:00 GMT
Content-Type: text/html
```

*Note: Nampak server details (nginx version)*

---

## Bahagian 3: Enable Cloudflare Proxy

### Langkah 1: Login ke Cloudflare

1. Pergi ke https://dash.cloudflare.com
2. Login dan pilih domain anda
3. Click **DNS** tab

### Langkah 2: Enable Proxy untuk Root Domain

1. Cari A record untuk root domain (`example.com` atau `@`)
2. Anda akan nampak icon **gray cloud** 🔴 (DNS only)
3. Click pada **gray cloud** icon
4. Icon akan bertukar ke **orange cloud** ☁️ (Proxied)
5. Record akan automatically save

**Before:**
```
Type: A
Name: @
Content: 54.123.45.67
Proxy status: 🔴 DNS only
```

**After:**
```
Type: A
Name: @
Content: 54.123.45.67
Proxy status: ☁️ Proxied
```

### Langkah 3: Enable Proxy untuk WWW (Jika Ada)

1. Cari record untuk `www` subdomain
2. Click gray cloud icon
3. Icon bertukar ke orange cloud
4. Save

---

## Bahagian 4: Verify Perubahan

### Langkah 1: Check New IP Address

Tunggu beberapa saat (30 saat - 2 minit), kemudian:

```bash
# Check IP address
dig +short example.com
```

**Expected output (Proxied mode):**
```
104.21.45.123
172.67.89.234
```
*Ini adalah Cloudflare IP addresses (bukan real server IP anda!)*

> **Nota**: Anda akan dapat 2 IP addresses - ini adalah Cloudflare edge servers yang akan proxy request ke server anda.

### Langkah 2: Verify Real IP Hidden

1. Pergi ke: https://www.whatsmydns.net/
2. Masukkan domain: `example.com`
3. Click **Search**
4. IP address sekarang adalah Cloudflare IP (104.x.x.x atau 172.x.x.x)

### Langkah 3: Check Response Headers

```bash
# Check HTTP headers dengan proxy enabled
curl -I http://example.com
```

**Expected output:**
```
HTTP/1.1 200 OK
Date: Mon, 10 Nov 2025 10:05:00 GMT
Content-Type: text/html
Server: cloudflare
CF-RAY: 8b1234567890abcd-KUL
CF-Cache-Status: DYNAMIC
```

**Perhatikan perubahan:**
- ✅ `Server: cloudflare` (bukan nginx lagi)
- ✅ Ada `CF-RAY` header (Cloudflare tracking ID)
- ✅ Ada `CF-Cache-Status` header
- ✅ Nginx version tidak visible (more secure)


---

## Bahagian 5: Lihat Perbezaan

### Comparison Table

| Feature | DNS Only (Gray Cloud) | Proxied (Orange Cloud) |
|---------|----------------------|------------------------|
| **IP Visible** | Real server IP | Cloudflare IP sahaja |
| **DNS Response** | 1 IP (server anda) | 2 IPs (Cloudflare edges) |
| **Traffic Route** | Direct ke server | Melalui Cloudflare |
| **CDN Caching** | ❌ Tidak | ✅ Ya |
| **DDoS Protection** | ❌ Basic AWS only | ✅ Yes (Cloudflare) |
| **SSL Certificate** | ❌ Perlu setup manual | ✅ Free automatic |
| **Server Header** | Visible (nginx/1.18.0) | Hidden (cloudflare) |
| **Response Time** | Direct connection | Potentially faster (CDN) |
| **Security** | Basic | Enhanced (WAF, bot protection) |
| **Analytics** | ❌ Limited | ✅ Detailed di Cloudflare |

### Speed Test Comparison

**Before (DNS only):**
```bash
# Test response time
time curl -I http://example.com
```

**After (Proxied):**
```bash
# Test response time with Cloudflare
time curl -I http://example.com
```

*Hasil mungkin berbeza bergantung lokasi anda dan Cloudflare edge server terdekat*


## Pengetahuan Tambahan

### Cloudflare Global Network

Cloudflare mempunyai 300+ data centers di seluruh dunia:
- Asia: Singapore, Kuala Lumpur, Bangkok, Jakarta, etc.
- Europe: London, Frankfurt, Paris, Amsterdam, etc.
- Americas: New York, Los Angeles, São Paulo, etc.
- Oceania: Sydney, Auckland, etc.

Bila enable proxy, request akan route ke edge server terdekat.


### Cloudflare Headers

Bila proxied, Cloudflare add extra headers:

```
CF-RAY: 8b1234567890abcd-KUL
  → Unique request ID + edge location (KUL = Kuala Lumpur)

CF-Cache-Status: HIT | MISS | DYNAMIC | BYPASS
  → HIT = Served from cache
  → MISS = Not in cache, fetched from origin
  → DYNAMIC = Dynamic content, not cacheable
  → BYPASS = Cache bypassed

CF-Connecting-IP: 1.2.3.4
  → Real visitor IP (use this for logging!)

CF-Visitor: {"scheme":"https"}
  → Original protocol used by visitor
```
