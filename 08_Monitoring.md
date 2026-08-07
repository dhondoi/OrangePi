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
