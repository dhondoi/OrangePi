Pilihan mantap! Kombinasi **Prometheus + Grafana + Node Exporter** ini adalah standar industri untuk *monitoring*. Prometheus bertugas mengumpulkan data, dan Grafana bertugas mengubah data tersebut jadi *dashboard* visual yang keren.

Karena Orange Pi Anda sudah menjalankan Prometheus dan Node Exporter via Docker, kita tinggal menambahkan **Grafana** ke dalam ekosistem Docker yang sama.

---

### Langkah 1: Jalankan Grafana via Docker

Masukkan perintah ini di terminal Orange Pi Anda (pastikan pakai network Docker yang sama jika sebelumnya buat manual, atau jalankan perintah biasa ini):

```bash
docker run -d \
  --name=grafana \
  -p 3000:3000 \
  --network monitoring \
  restart: always \
  grafana/grafana

```

*(Catatan: Sesuaikan `--network monitoring` dengan nama Docker network yang Anda pakai sebelumnya).*

---

### Langkah 2: Login ke Grafana

1. Buka browser dan ketik: `http://IP-OrangePi-Anda:3000`
2. Login default Grafana:
* **Username:** `admin`
* **Password:** `admin`


3. Sistem akan meminta Anda mengganti password baru (bisa di-*skip* atau diganti).

---

### Langkah 3: Hubungkan Grafana ke Prometheus

1. Di menu sebelah kiri Grafana, klik ikon **Connections** (atau **Configuration**) -> **Data Sources**.
2. Klik tombol **Add data source**, lalu pilih **Prometheus**.
3. Di kolom **Prometheus server URL**, masukkan:
```text
http://prometheus:9090

```


*(Karena berada di satu Docker network, cukup panggil nama kontainernya).*
4. Scroll ke paling bawah, lalu klik **Save & test**.
5. Jika muncul centang hijau **"Data source is working"**, artinya Grafana sudah terhubung!

---

### Langkah 4: Impor Dashboard Siap Pakai (Tanpa Perlu Bikin Manual!)

Anda tidak perlu membuat grafik dari nol. Komunitas sudah membuatkan *dashboard* khusus Node Exporter yang sangat lengkap.

1. Buka menu **Dashboards** di panel sebelah kiri -> Klik **New** -> **Import**.
2. Di kolom **Import via grafana.com**, ketik ID Dashboard: **`1860`** (ini ID resmi untuk *Node Exporter Full*).
3. Klik tombol **Load**.
4. Pada opsi **Prometheus** di bagian bawah, pilih nama Data Source yang tadi Anda tambahkan.
5. Klik **Import**.

---

### Hasil Akhir 🚀

Sekarang Anda punya *dashboard* interaktif yang menampilkan:

* Persentase beban CPU real-time
* Penggunaan RAM & Swap memory
* Temperatur Orange Pi (jika sensor terbaca)
* Lalu lintas jaringan (*Network Traffic*)
* Sisa kapasitas ruang SD Card
