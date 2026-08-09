# A. Konsep Singkat Arsitektur Prometheus
Prometheus bekerja dengan sistem Pull (menarik data). Jadi alurnya seperti ini:

Exporter (Aplikasi kecil) dipasang di Orange Pi untuk mengumpulkan data (metrics).

Prometheus bertugas mengambil/menarik data dari Exporter tersebut secara berkala dan menyimpannya.

(Opsional) Grafana digunakan untuk menampilkan data dari Prometheus dalam bentuk grafik dashboard yang keren.

## Cara Pasang Exporter di Orange Pi (Gunakan Perintah Docker)
Untuk memantau Orange Pi dan Nginx kamu, kita butuh 2 Exporter:

1. Node Exporter (Untuk Memantau Hardware Orange Pi: CPU, RAM, Disk, Suhu)
Jalankan perintah ini di terminal Orange Pi kamu:

```Bash
docker run -d \
  --name=node-exporter \
  --restart=always \
  --net="host" \
  --pid="host" \
  -v "/:/host:ro,rslave" \
  quay.io/prometheus/node-exporter:latest \
  --path.rootfs=/host
Port bawaan: 9100
```
2. Prometheus Engine (Aplikasi Utama Prometheus)
Sebelum menjalankan Prometheus, kita perlu membuat file konfigurasi kecil dulu bernama prometheus.yml.

Langkah A: Buat folder dan file konfigurasi

```Bash
mkdir -p ~/prometheus-data
nano ~/prometheus-data/prometheus.yml
```
Langkah B: Paste isi konfigurasi ini ke dalam file prometheus.yml:

```YAML
global:
  scrape_interval: 15s

scrape_configs:
  - job_name: 'orangepi-hardware'
    static_configs:
      - targets: ['127.0.0.1:9100']
```
(Simpan file dengan menekan Ctrl+O, Enter, lalu keluar dengan Ctrl+X)

Langkah C: Jalankan Prometheus dengan Docker

```Bash
docker run -d \
  --name prometheus \
  --restart always \
  -p 9090:9090 \
  -v ~/prometheus-data/prometheus.yml:/etc/prometheus/prometheus.yml \
  prom/prometheus
```
Cara Cek Apakah Prometheus Sudah Jalan
Buka browser dan akses: http://IP-OrangePi-Kamu:9090

Di menu atas, masuk ke Status > Targets.

# B. Promotheus NGINX
## Langkah 1: Aktifkan `stub_status` di Nginx

Buka file konfigurasi Nginx milikmu (biasanya ada di `/etc/nginx/sites-available/default` atau `/etc/nginx/nginx.conf`):

```bash
sudo nano /etc/nginx/sites-available/default

```

Tambahkan blok `location /nginx_status` di dalam blok `server` kamu:

```nginx
server {
    listen 80;
    server_name localhost;

    # Halaman web statismu
    location / {
        root /var/www/html;
        index index.html;
    }

    # DATAPOINT UNTUK PROMETHEUS (Tambahkan bagian ini)
    location /nginx_status {
        stub_status on;
        allow 127.0.0.1;        # Hanya izinkan akses dari internal Orange Pi
        deny all;              # Tolak akses dari luar demi keamanan
    }
}

```

Setelah disimpan, tes dan *reload* Nginx:

```bash
sudo nginx -t
sudo systemctl reload nginx

```

> **Tes manual:** Ketik `curl http://127.0.0.1/nginx_status` di terminal. Jika keluar tulisan seperti `Active connections: 1 ...`, berarti Nginx sudah siap!

---

## Langkah 2: Jalankan Nginx Exporter via Docker

Jalankan kontainer **Nginx Exporter** untuk mengubah data status Nginx tadi menjadi format yang dipahami Prometheus:

```bash
docker run -d \
  --name nginx-exporter \
  --restart always \
  --net="host" \
  nginx/nginx-prometheus-exporter:latest \
  -nginx.scrape-uri="http://127.0.0.1/nginx_status"

```

*Exporter ini akan berjalan di port `9113`.*

---

## Langkah 3: Daftarkan Nginx ke `prometheus.yml`

Buka kembali file konfigurasi Prometheus yang kita buat sebelumnya:

```bash
nano ~/prometheus-data/prometheus.yml

```

Tambahkan *job* baru untuk Nginx di bawah `scrape_configs`:

```yaml
global:
  scrape_interval: 15s

scrape_configs:
  - job_name: 'orangepi-hardware'
    static_configs:
      - targets: ['127.0.0.1:9100']

  # TAMBAHKAN JOB NGINX DI BAWAH INI
  - job_name: 'nginx-web'
    static_configs:
      - targets: ['127.0.0.1:9113']

```

Restart kontainer Prometheus agar membaca konfigurasi baru:

```bash
docker restart prometheus

```

---

## Metrik Apa Saja yang Sekarang Bisa Kamu Lihat?

Buka Prometheus di browser (`http://IP-OrangePi-Kamu:9090`), lalu kamu bisa mencari metrik-metrik ini:

* **`nginx_connections_active`**: Jumlah koneksi/pengunjung yang sedang aktif detik ini.
* **`nginx_http_requests_total`**: Total seluruh *request* HTTP yang masuk ke web statismu.
* **`nginx_connections_reading / writing / waiting`**: Detail proses penanganan *traffic* oleh Nginx.

# C. **Grafana** adalah aplikasi pemvisualisasi data (*dashboarding tool*).

Jika disederhanakan perannya dalam sistem *monitoring* kamu:

* **Node Exporter & Nginx Exporter** = Bertugas **mengumpulkan data** (suhu Orange Pi, CPU, jumlah pengunjung web).
* **Prometheus** = Bertugas **menyimpan dan mengolah data** mentah tersebut (seperti *database*).
* **Grafana** = Bertugas **menampilkan data** dari Prometheus agar menjadi grafik, chart, dan indikator warna-warni yang indah, rapi, dan mudah dibaca.

Tanpa Grafana, data di Prometheus hanya berbentuk teks mentah dan angka-angka yang membingungkan. Grafana-lah yang mengubahnya jadi *dashboard* ala pusat kontrol (*command center*).

---

## Contoh Tampilan Dashboard Grafana

Dengan menghubungkan Grafana ke Prometheus di Orange Pi kamu, kamu bisa membuat grafik seperti:

1. **Grafik Suhu CPU Orange Pi** (Lengkap dengan warna merah jika terlalu panas).
2. **Grafik Penggunaan RAM & Storage**.
3. **Grafik Traffic Web Nginx** (Berapa *request* per detik yang masuk ke web statismu).
4. **Indikator Status Online/Offline**.

---

## Cara Pasang Grafana Menggunakan Docker

Kamu bisa langsung menjalankannya di Orange Pi kamu dengan satu perintah ini:

```bash
docker run -d \
  --name=grafana \
  --restart=always \
  -p 3000:3000 \
  grafana/grafana-oss

```

### Cara Akses:

1. Buka browser dan ketik: `http://IP-OrangePi-Kamu:3000`
2. **Username bawaan**: `admin`
3. **Password bawaan**: `admin` (kamu akan diminta mengganti password saat pertama kali *login*).

---

## Cara Menghubungkan Grafana ke Prometheus

1. Setelah masuk ke Grafana, buka menu **Connections** > **Data sources**.
2. Klik **Add data source**, lalu pilih **Prometheus**.
3. Pada kolom **Prometheus server URL**, masukkan IP Orange Pi kamu beserta port Prometheus:
`http://192.168.x.x:9090` *(Ganti dengan IP Orange Pi kamu)*
4. Scroll ke bawah dan klik **Save & test**. Jika keluar warna hijau, artinya Grafana dan Prometheus sudah terhubung!

> 💡 **Tips Praktis:** Kamu tidak perlu membuat grafik dari nol! Di Grafana, kamu bisa mengimpor *dashboard* siap pakai yang dibuat oleh komunitas. Cukup masukkan ID Dashboard (misalnya ID `1860` untuk Node Exporter / Orange Pi), dan *dashboard* keren akan otomatis terbentuk!
