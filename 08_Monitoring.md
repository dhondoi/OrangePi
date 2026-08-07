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
