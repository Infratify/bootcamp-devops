## LAB 23: POINT ROOT DNS RECORD KE AWS SERVER (CLOUDFLARE)

### Prasyarat

Sebelum memulakan lab ini, pastikan anda sudah ada:
- ✅ EC2 instance yang berjalan dengan Nginx (dari Lab 6-8)
- ✅ Domain name sendiri yang sudah add ke Cloudflare
- ✅ Akaun Cloudflare (free tier sudah memadai)
- ✅ Public IP address dari EC2 instance anda

### Objektif

Dalam lab ini, anda akan belajar:
1. Mencari Public IP address EC2 instance
2. Login ke Cloudflare Dashboard
3. Menambah A record untuk point root domain ke server AWS
4. Configure Cloudflare settings untuk optimal performance
5. Verify domain sudah berfungsi

---

### Langkah 1: Dapatkan Public IP Address

1. Pergi ke AWS Console → EC2 Dashboard
2. Click pada instance anda
3. Dalam bahagian **Details**, cari dan copy **Public IPv4 address**

   **Contoh**: `54.123.45.67`

> **Nota**: Jika instance anda restart, Public IP akan berubah! Untuk production, guna **Elastic IP** (IP statik).

---

### Langkah 2: Login ke Cloudflare Dashboard

1. Pergi ke https://dash.cloudflare.com
2. Login dengan email dan password anda
3. Anda akan nampak list domain anda
4. Click pada domain yang ingin anda configure

---

### Langkah 3: Pergi ke DNS Settings

1. Selepas pilih domain, anda akan berada di overview page
2. Di sebelah kiri menu, click **DNS**
3. Atau click tab **DNS** di bahagian atas
4. Anda akan nampak **DNS Records** section

---

### Langkah 4: Delete Existing Records (Jika Ada)

Jika ada A record atau CNAME record untuk root domain (`@` atau `example.com`), delete dahulu:

1. Cari record dengan **Name** = `example.com` atau `@`
2. Click butang **Edit** atau click pada record
3. Click **Delete**
4. Confirm deletion

---

### Langkah 5: Tambah A Record untuk Root Domain

1. Click butang **Add record** (biasanya berwarna biru)

2. Isikan maklumat berikut:

   | Field | Value | Penjelasan |
   |-------|-------|------------|
   | **Type** | `A` | A record untuk point ke IP address |
   | **Name** | `@` | Simbol `@` bermaksud root domain |
   | **IPv4 address** | `54.123.45.67` | Public IP EC2 anda |
   | **Proxy status** | 🔴 **DNS only** | Pilih "DNS only" (gray cloud) |
   | **TTL** | `Auto` | Automatic TTL |

3. **PENTING**: Pastikan **Proxy status** adalah **DNS only** (gray cloud icon 🔴)
   - **Proxied** (orange cloud ☁️): Traffic melalui Cloudflare (gunakan jika mahu CDN)
   - **DNS only** (gray cloud): Direct connection ke server anda

4. Click **Save**

**Contoh configuration:**
```
Type: A
Name: @
IPv4 address: 54.123.45.67
Proxy status: DNS only (gray cloud)
TTL: Auto
```

> **Penjelasan**:
> - `@` = root domain (contoh: `example.com`, bukan `www.example.com`)
> - `A record` = Mapping domain ke IPv4 address
> - `DNS only` = Cloudflare hanya handle DNS, tak proxy traffic
> - `TTL Auto` = Cloudflare automatically manage TTL

---

### Langkah 6: Tambah A Record untuk WWW (Optional)

Untuk `www.example.com` juga boleh diakses:

**Option 1: Guna A Record (sama macam root)**

1. Click **Add record**
2. Isikan maklumat:

   | Field | Value |
   |-------|-------|
   | **Type** | `A` |
   | **Name** | `www` |
   | **IPv4 address** | `54.123.45.67` |
   | **Proxy status** | 🔴 **DNS only** |
   | **TTL** | `Auto` |

3. Click **Save**

**Option 2: Guna CNAME Record (lebih baik)**

1. Click **Add record**
2. Isikan maklumat:

   | Field | Value |
   |-------|-------|
   | **Type** | `CNAME` |
   | **Name** | `www` |
   | **Target** | `example.com` |
   | **Proxy status** | 🔴 **DNS only** |
   | **TTL** | `Auto` |

3. Click **Save**

> **Nota**: CNAME lebih baik sebab jika IP berubah, anda hanya perlu update satu record sahaja (root domain).

---

### Langkah 7: Verify DNS Records

Di Cloudflare DNS Records page, anda sepatutnya nampak:

```
Type    Name           Content         Proxy status    TTL
A       example.com    54.123.45.67    DNS only        Auto
CNAME   www            example.com     DNS only        Auto
```

---

### Langkah 8: Check DNS Propagation

Cloudflare updates DNS sangat cepat (biasanya dalam beberapa saat).

#### A. Check menggunakan terminal

```bash
# Check A record untuk root domain
dig example.com

# Check A record untuk www
dig www.example.com

# Atau guna command ni untuk result lebih simple
dig +short example.com
```

**Expected output:**
```
54.123.45.67
```

#### B. Check menggunakan online tools

Pergi ke: https://dnschecker.org

1. Masukkan domain anda: `example.com`
2. Pilih record type: `A`
3. Click **Search**
4. Verify kebanyakan location show IP address anda

---

### Langkah 9: Test dari Browser

1. Buka browser (Chrome, Firefox, Safari)
2. Type domain anda dalam address bar:
   ```
   http://example.com
   ```
   *Ganti dengan domain anda!*

3. Tekan Enter

**Expected result:**
```
Welcome to nginx!
```

✅ Domain anda berjaya point ke AWS server!

4. Test juga dengan `www`:
   ```
   http://www.example.com
   ```

---

### Troubleshooting

#### ❌ Problem 1: Domain tidak resolve

**Solusi:**
- Check DNS records di Cloudflare - pastikan A record betul
- Tunggu beberapa minit untuk DNS propagate
- Clear DNS cache di komputer anda:
  ```bash
  # macOS
  sudo dscacheutil -flushcache; sudo killall -HUP mDNSResponder

  # Windows
  ipconfig /flushdns

  # Linux
  sudo systemd-resolve --flush-caches
  ```

#### ❌ Problem 2: Shows "Error 522: Connection timed out"

**Solusi:**
- Pastikan **Proxy status** adalah **DNS only** (gray cloud), bukan Proxied (orange cloud)
- Jika mahu guna Proxied, kena configure Security Group untuk allow Cloudflare IPs

#### ❌ Problem 3: "Connection refused" atau "Site can't be reached"

**Solusi:**
- Check EC2 instance Security Group:
  - Pergi ke EC2 → Security Groups
  - Pastikan ada inbound rule untuk HTTP (port 80) dari `0.0.0.0/0`
- Check Nginx service running:
  ```bash
  sudo systemctl status nginx
  ```
- Check EC2 instance masih running di AWS Console

#### ❌ Problem 4: Public IP berubah selepas restart

**Solusi:**
- Guna **Elastic IP** (IP statik dari AWS):
  1. EC2 Dashboard → Elastic IPs
  2. Allocate Elastic IP address
  3. Associate dengan EC2 instance anda
  4. Update A record di Cloudflare dengan Elastic IP baru

---

### Tips Production

1. **Guna Elastic IP**: Supaya IP tidak berubah bila restart instance
2. **Enable Cloudflare Proxy**: Untuk CDN, security, dan free SSL
3. **Configure SSL**: Guna Cloudflare Origin Certificate di Nginx
4. **Enable Security Features**: Firewall rules, rate limiting, bot protection
5. **Monitor Analytics**: Check traffic pattern di Cloudflare dashboard


---

### Pengetahuan Tambahan

**Cloudflare Proxy Status:**
- **DNS only (gray cloud)**: Direct connection ke server
  - Pros: Simple setup, full control
  - Cons: No CDN, no extra security, real IP exposed

- **Proxied (orange cloud)**: Traffic melalui Cloudflare
  - Pros: CDN, DDoS protection, free SSL, hide real IP
  - Cons: Perlu configure security group, slightly complex

**DNS Record Types:**
- **A Record**: Maps domain ke IPv4 address
- **AAAA Record**: Maps domain ke IPv6 address
- **CNAME Record**: Maps domain ke domain lain (alias)
- **MX Record**: Mail server records
- **TXT Record**: Text records (untuk verification, SPF, DKIM)

**Root Domain vs Subdomain:**
- Root domain: `example.com` (guna `@` di Cloudflare)
- Subdomain: `www.example.com`, `blog.example.com` (guna `www`, `blog`)

---

🎉 **Tahniah!** Domain anda sudah point ke AWS server menggunakan Cloudflare dan boleh diakses dari internet!
