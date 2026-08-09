# KUMA
## Langkah 1: Install Uptime Kuma (via Docker Compose)

Buat folder khusus untuk Uptime Kuma di Orange Pi kamu:

```bash
mkdir ~/uptime-kuma && cd ~/uptime-kuma
nano docker-compose.yml

```

Tempel (*paste*) konfigurasi berikut:

```yaml
services:
  uptime-kuma:
    image: louislam/uptime-kuma:1
    container_name: uptime-kuma
    restart: always
    ports:
      - '3001:3001'
    volumes:
      - ./uptime-kuma-data:/app/data

```

Jalankan container-nya:

```bash
sudo docker compose up -d

```
Berikut perintah alternatifnya yang setara dengan file YAML kamu:

```Bash
docker run -d \
  --name uptime-kuma \
  --restart always \
  -p 3001:3001 \
  -v $(pwd)/uptime-kuma-data:/app/data \
  louislam/uptime-kuma:1
```  
> **Cek Akses Lokal:** > Buka browser dan akses `http://IP_ORANGE_PI_KAMU:3001`. Kamu akan diminta untuk membuat akun **Admin** pertama kali.

---

## Langkah 2: Hubungkan ke Cloudflare Tunnel (Supaya BISA DIPAMERKAN)

Supaya status page kamu bisa diakses siapa saja di internet tanpa perlu *login*:

1. Buka dashboard **Cloudflare Zero Trust** (`dash.teams.cloudflare.com`).
2. Masuk ke menu **Networks** > **Tunnels**.
3. Pilih Tunnel kamu, lalu klik **Edit**.
4. Tambahkan **Public Hostname** baru:
* **Subdomain:** `status`
* **Domain:** `dhondoi.online`
* **Service Type:** `HTTP`
* **URL:** `localhost:3001` (atau IP Lokal Orange Pi kamu `:3001`)


5. Klik **Save hostname**.

---

## Langkah 3: Membuat Public Status Page (Halaman Pamer)

Ini fitur utamanya! Kamu bisa membuat halaman status cantik tanpa perlu memberikan akses ke dashboard admin kamu:

1. Buka dashboard Uptime Kuma kamu (`http://IP_ORANGE_PI_KAMU:3001`).
2. Tambahkan dulu monitor yang mau dipamerkan (misal: Website Laravel kamu, Ping Orange Pi, Nginx, Port MySQL, dll) lewat tombol **Add New Monitor**.
3. Setelah monitor dibuat, klik menu **Status Pages** di bagian atas menu.
4. Klik **New Status Page**.
5. Isi data:
* **Title:** *OrangePi Server Status* (atau bebas sesuai selera).
* **URL Slug:** `status` atau biarkan default.


6. Pilih monitor mana saja yang ingin kamu tampilkan ke publik.
7. Kamu juga bisa ganti Theme (Dark Mode keren banget!), masukkan custom CSS, atau upload logo pribadi/profil.
8. Klik **Save**.

---

## Hasil Akhir 🎉

Sekarang siapa saja yang membuka URL domain kamu:
👉 **`https://status.dhondoi.online`**

Akan langsung melihat **halaman indikator hijau menyala**, grafik *uptime* 30 hari, *response time* (ping ms), dan status *real-time* dari Orange Pi kamu **secara publik tanpa perlu login**!

Cocok banget buat dipasang di link bio Instagram, GitHub profile, atau dibagikan ke teman-teman komunitas!
---


# Beszel menggunakan arsitektur **Hub & Agent**:

1. **Beszel Hub**: Dashboard utama tempat kamu melihat grafik.
2. **Beszel Agent**: Layanan kecil yang berjalan di Orange Pi untuk mengirim data ke Hub (Hub dan Agent bisa dipasang di satu Orange Pi yang sama).

Cara paling mudah dan direkomendasikan untuk memasang Beszel adalah menggunakan **Docker & Docker Compose**.

---

## Langkah 1: Install Docker di Orange Pi (Jika Belum Ada)

Jalankan perintah ini di terminal SSH Orange Pi kamu untuk menginstal Docker dengan cepat:

```bash
curl -fsSL https://get.docker.com -o get-docker.sh
sudo sh get-docker.sh

```

---

## Langkah 2: Menyiapkan File Configuration (Docker Compose)

Buat folder khusus untuk Beszel dan masuk ke foldernya:

```bash
mkdir ~/beszel && cd ~/beszel
nano docker-compose.yml

```

Salin (*copy*) dan tempel (*paste*) konfigurasi `docker-compose.yml` berikut:

```yaml
services:
  beszel:
    image: 'henrygd/beszel:latest'
    container_name: 'beszel'
    restart: unless-stopped
    ports:
      - '8090:8090'
    volumes:
      - ./beszel_data:/beszel_data

  beszel-agent:
    image: 'henrygd/beszel-agent:latest'
    container_name: 'beszel-agent'
    restart: unless-stopped
    network_mode: host
    volumes:
      - /var/run/docker.sock:/var/run/docker.sock:ro
    environment:
      PORT: 45876
      KEY: 'PASTE_PUBLIC_KEY_DI_SINI'

```

> ⚠️ **PENTING:** Jangan jalankan dulu! Kita butuh mengisi `KEY` (Public Key) pada variabel `KEY` di atas terlebih dahulu melalui dashboard Beszel.

---

## Langkah 3: Jalankan Beszel Hub Terlebih Dahulu

1. Hapus dulu bagian `beszel-agent` sementara atau cukup jalankan container `beszel` saja dengan perintah:
```bash
sudo docker run -d \
  --name beszel \
  --restart unless-stopped \
  -p 8090:8090 \
  -v ./beszel_data:/beszel_data \
  henrygd/beszel:latest

```


2. Buka browser di Laptop/HP kamu, lalu akses:
`http://IP_ORANGE_PI_KAMU:8090`
*(Contoh: `http://192.168.1.50:8090`)*
3. **Buat akun administrator** pertama kamu (Username/Email & Password).
4. Di dashboard Beszel:
* Klik tombol **Add System** / **+**.
* Masukkan nama Orange Pi kamu (misal: `Orange-Pi`).
* Masukkan Host/IP Orange Pi kamu (misal: `localhost` atau IP lokalnya).
* Kamu akan melihat baris **Public Key** yang dimunculkan di layar. **Salin (copy) kunci tersebut!**



---

## Langkah 4: Jalankan Agent Menggunakan Public Key

Setelah mendapatkan Public Key dari dashboard:

1. Jalankan perintah agent di bawah ini di terminal Orange Pi (ganti `PASTE_PUBLIC_KEY_DI_SINI` dengan kunci yang kamu salin tadi):
```bash
sudo docker run -d \
  --name beszel-agent \
  --restart unless-stopped \
  --network host \
  -v /var/run/docker.sock:/var/run/docker.sock:ro \
  -e PORT=45876 \
  -e KEY="PASTE_PUBLIC_KEY_DI_SINI" \
  henrygd/beszel-agent:latest

```


2. Kembali ke browser dan simpan pengaturan di dashboard Beszel.

---

## Selesai! 🎉

Dalam beberapa detik, status Orange Pi di dashboard Beszel kamu akan berubah menjadi **Connected** (Hijau), dan kamu bisa langsung memantau suhu CPU, RAM, Disk, hingga penggunaan Docker container secara *real-time*!

Apakah kamu mengalami kendala saat install Docker atau saat membuat containernya?

Tentu, ini langkah-langkah lengkap dan paling praktis untuk meng-install **Netdata** di Orange Pi milikmu, sampai bisa diakses publik tanpa perlu login.

---
# NETDATA

## Langkah 1: Update Sistem Orange Pi

Buka Terminal/SSH Orange Pi kamu, lalu jalankan perintah update agar paket sistem berada di versi terbaru:

```bash
sudo apt update && sudo apt upgrade -y

```

---

## Langkah 2: Install Netdata

Netdata menyediakan skrip *installer* resmi satu baris yang otomatis mendeteksi OS dan arsitektur CPU Orange Pi (ARM).

Jalankan perintah berikut:

```bash
wget -O /tmp/netdata-kickstart.sh https://get.netdata.cloud/kickstart.sh && sh /tmp/netdata-kickstart.sh --disable-telemetry

```

> **Keterangan:** Opsi `--disable-telemetry` bersifat opsional, pengunaannya agar Netdata tidak mengirim statistik penggunaan anonim ke server mereka.

Proses instalasi butuh waktu sekitar **2–5 menit** tergantung kecepatan internet dan spek Orange Pi kamu. Tunggu sampai muncul pesan sukses.

---

## Langkah 3: Tes Akses di Jaringan Lokal

Setiap kali selesai di-install, Netdata otomatis berjalan di background menggunakan port `19999`.

1. Cek IP lokal Orange Pi kamu (misal: `192.168.1.50`) dengan perintah:
```bash
hostname -I

```


2. Buka browser di HP/Laptop (yang terhubung ke WiFi/jaringan yang sama).
3. Masukkan URL:
```text
http://IP_ORANGE_PI_KAMU:19999

```


*(Contoh: `http://192.168.1.50:19999`)*

Kamu akan langsung melihat *dashboard* animasi grafik Netdata yang sangat responsif **tanpa minta login sama sekali**.

---

## Langkah 4: Aktifkan Sensor Suhu CPU (Penting untuk Orange Pi)

Agar widget suhu CPU Orange Pi kamu muncul dan bisa dipamerkan ke orang lain, install paket pembaca sensor berikut:

```bash
sudo apt install lm-sensors -y

```

Setelah terinstall, restart service Netdata agar mendeteksi sensor barunya:

```bash
sudo systemctl restart netdata

```

---

## Langkah 5: Bikin URL Publik (Bisa Diakses dari Mana Saja)

Agar orang luar (tidak di WiFi rumahmu) bisa klik link dan langsung melihat dashboard Netdata tanpa login, cara paling aman dan gratis adalah menggunakan **Cloudflare Tunnel**:

1. Download `cloudflared` di Orange Pi:
```bash
curl -L --output cloudflared.deb https://github.com/cloudflare/cloudflared/releases/latest/download/cloudflared-linux-arm64.deb
sudo dpkg -i cloudflared.deb

```


*(Catatan: Jika Orange Pi kamu pakai OS 32-bit, ganti `arm64` menjadi `arm`)*.
2. Bikin URL acak publik secara instant (*Quick Tunnel*):
```bash
cloudflared tunnel --url http://localhost:19999

```


3. Di terminal akan muncul link publik otomatis seperti:
`https://random-words-123.trycloudflare.com`

Link tersebut tinggal kamu salin dan bagikan ke teman-temanmu. Siapapun yang mengklik link itu bisa langsung melihat statistik Orange Pi kamu secara *real-time* tanpa perlu login!

cara uninstall :
```bash
sudo systemctl stop debian-keyring netdata-repo-edge

sudo apt purge netdata-repo-edge -y

sudo apt autoremove -y
```
