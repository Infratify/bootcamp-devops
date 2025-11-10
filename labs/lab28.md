## LAB 28: SETUP CLOUDFLARE TUNNEL UNTUK CLOUDPANEL ADMIN PANEL

### Prasyarat

Sebelum memulakan lab ini, pastikan anda sudah:
- ✅ EC2 instance baru (Ubuntu 24.04)
- ✅ CloudPanel installed di server (port 8443 dibuka)
- ✅ Domain aktif di Cloudflare
- ✅ Cloudflare account dengan domain verified
- ✅ Boleh SSH/SSM ke EC2 instance

> **Nota**: Lab ini memerlukan EC2 instance yang BERBEZA dari lab-lab sebelumnya untuk avoid conflicts dengan Nginx configurations.

### Objektif

Dalam lab ini, anda akan belajar:
1. Memahami apa itu Cloudflare Tunnel dan kegunaannya
2. Install CloudPanel di EC2 instance
3. Install Cloudflare Tunnel (cloudflared) di server
4. Authenticate dengan Cloudflare account
5. Buat tunnel untuk expose CloudPanel admin panel
6. Configure DNS untuk subdomain cp.domain.com
7. Access CloudPanel securely melalui tunnel
8. Understand security benefits dari Cloudflare Tunnel

---

## Bahagian 1: Memahami Cloudflare Tunnel

### Apa itu Cloudflare Tunnel?

Cloudflare Tunnel (dahulunya Argo Tunnel) membenarkan anda expose aplikasi web secara selamat tanpa perlu:
- ❌ Buka ports di firewall (contoh: port 8443)
- ❌ Public IP address accessible
- ❌ Configure complex security rules

### Traditional Setup vs Cloudflare Tunnel

**Traditional Setup:**
```
Internet → Public IP:8443 → Security Group (port 8443 open) → CloudPanel
```
**Risiko:**
- ⚠️ Port 8443 terdedah ke internet
- ⚠️ Potential brute force attacks
- ⚠️ Need manage firewall rules
- ⚠️ Server IP exposed

**Cloudflare Tunnel:**
```
Internet → Cloudflare Edge → Encrypted Tunnel → CloudPanel (localhost:8443)
```
**Kelebihan:**
- ✅ Tiada inbound ports dibuka
- ✅ Zero-trust security
- ✅ DDoS protection at edge
- ✅ Server IP hidden
- ✅ Automatic SSL/TLS
- ✅ Access controls built-in

### How Cloudflare Tunnel Works

```
1. Server runs cloudflared daemon
   ↓
2. cloudflared buat outbound connection ke Cloudflare
   ↓
3. Cloudflare create secure tunnel
   ↓
4. User access: cp.domain.com
   ↓
5. Request route melalui Cloudflare Edge
   ↓
6. Cloudflare forward through tunnel
   ↓
7. cloudflared proxy ke localhost:8443 (CloudPanel)
   ↓
8. Response balik through tunnel
   ↓
9. User dapat content (selamat!)
```

**Key point**: Tiada inbound connections ke server. Semua melalui outbound tunnel yang server initiate!

---

## Bahagian 2: Install CloudPanel

### Langkah 1: Sambung ke EC2 Instance Baru

```bash
# Melalui SSM (disyorkan)
aws ssm start-session --target i-xxxxxxxxxxxxx

# Atau melalui SSH
ssh -i your-key.pem ubuntu@your-ec2-ip
```

### Langkah 2: Update System

```bash
# Update package list
sudo apt update

# Upgrade installed packages
sudo apt upgrade -y
```

### Langkah 3: Install CloudPanel
Ikuti langkah pemasangan di : https://www.cloudpanel.io/docs/v2/getting-started/other/

**Proses installation akan mengambil masa 5-10 minit.**

**Expected output:**
```
Installing CloudPanel...
[Progress bars...]
Installation completed successfully!

CloudPanel Admin URL: https://YOUR_SERVER_IP:8443
```

### Langkah 4: Buka port 8443 di Security Group


### Langkah 5: Setup CloudPanel login credentials

---

## Bahagian 3: Buat Cloudflare Tunnel melalui Dashboard

### Langkah 1: Login ke Cloudflare Dashboard

1. Pergi ke https://dash.cloudflare.com
2. Login dengan account anda
3. Pilih domain yang anda nak guna

### Langkah 2: Navigate ke Cloudflare Zero Trust

1. Di dashboard, click **Zero Trust** di sidebar kiri
2. Atau pergi direct ke: https://one.dash.cloudflare.com

**Nota**: Jika first time guna Zero Trust, anda perlu setup team name (boleh guna nama sendiri atau company).

### Langkah 3: Pergi ke Tunnels Section

1. Di Zero Trust dashboard, expand **Networks** di sidebar
2. Click **Tunnels**
3. Anda akan nampak page "Create a tunnel"

### Langkah 4: Create Tunnel

1. Click butang **Create a tunnel**
2. Select tunnel type: **Cloudflared**
3. Click **Next**

### Langkah 5: Name Your Tunnel

1. Tunnel name: `cloudpanel`
2. Click **Save tunnel**

**Tunnel akan dibuat dan anda akan dapat installation instructions.**

---

## Bahagian 4: Install Connector di Server

### Langkah 1: View Installation Instructions

Selepas create tunnel, Cloudflare akan show installation steps untuk different OS.

Pilih: **Debian** (untuk Ubuntu)

### Langkah 2: Copy Installation Commands

Cloudflare akan provide commands yang dah include dengan tunnel token. Contoh:

```bash
# Example commands (your commands will be different!)
curl -L --output cloudflared.deb https://github.com/cloudflare/cloudflared/releases/latest/download/cloudflared-linux-amd64.deb

sudo dpkg -i cloudflared.deb

sudo cloudflared service install <YOUR-UNIQUE-TOKEN-HERE>
```

**PENTING**: Guna commands yang Cloudflare provide untuk anda, BUKAN commands di atas! Setiap tunnel ada unique token.

### Langkah 3: Run Commands di Server

Di server terminal, run commands satu persatu:

**Command 1: Download cloudflared**
```bash
curl -L --output cloudflared.deb https://github.com/cloudflare/cloudflared/releases/latest/download/cloudflared-linux-amd64.deb
```

**Command 2: Install package**
```bash
sudo dpkg -i cloudflared.deb
```

**Expected output:**
```
Selecting previously unselected package cloudflared.
Unpacking cloudflared...
Setting up cloudflared...
```

**Command 3: Install tunnel service dengan token**
```bash
sudo cloudflared service install <YOUR-TOKEN>
```

**Ganti `<YOUR-TOKEN>` dengan token yang Cloudflare provide!**

**Expected output:**
```
Successfully installed cloudflared service
```

### Langkah 4: Verify Installation


Balik ke Cloudflare Dashboard, anda akan nampak:

```
Connector Status: HEALTHY
● Connected
```

✅ Tunnel successfully connected!

Click **Next** untuk proceed.

---

## Bahagian 5: Configure Public Hostname

### Langkah 1: Add Public Hostname

Di Cloudflare Dashboard, section "Route tunnel":

1. **Subdomain**: `cp`
2. **Domain**: Pilih domain anda dari dropdown
3. **Path**: (leave blank)
4. **Type**: `HTTPS`
5. **URL**: `localhost:8443`

**Full hostname akan jadi**: `cp.domain.com`

### Langkah 2: Additional Settings

Click **Additional application settings** untuk expand:

1. Scroll ke **TLS Settings**
2. Enable **No TLS Verify** (toggle ON)

**PENTING**: Ini perlu kerana CloudPanel guna self-signed certificate.

### Langkah 3: Save Hostname

Click **Save tunnel**

**Expected**: Route configured successfully!

---

## Bahagian 6: Verify DNS dan Tunnel

1. Di Cloudflare Dashboard, pergi ke **DNS** tab
2. Anda akan nampak CNAME record baru:

```
Type     Name    Content                              Proxy
CNAME    cp      <TUNNEL-ID>.cfargotunnel.com         Proxied (orange cloud)
```

---

## Bahagian 7: Test Access CloudPanel

### Langkah 1: Open Browser

1. Buka browser baru
2. Pergi ke: `https://cp.domain.com`

**Ganti dengan subdomain anda!**

### Langkah 2: CloudPanel Login Page

**Expected result:**
- ✅ CloudPanel login page loaded
- ✅ Padlock icon 🔒 (HTTPS dengan Cloudflare certificate)
- ✅ Tiada security warnings
- ✅ Fast loading

### Langkah 3: Login ke CloudPanel

Masukkan credentials:
- **Username**: admin
- **Password**: (dari `sudo clpctl admin:show` tadi)

Click **Login**

**Expected**: CloudPanel dashboard loaded successfully!
