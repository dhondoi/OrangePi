
---
# SSH TANPA PASSWORD FOR ROOT
- Buka CMD di Komputer pada direktori ssh `/Users/<nama_user>/.ssh`, untuk menghindari salah penempatan direktori saat pembuatan keygen.
- Gunakan Ed25519 untuk semua kebutuhan modern (Default pilihan utama):
```Bash
ssh-keygen -t ed25519 -C "email_anda@example.com"
```
- Gunakan RSA 4096-bit HANYA jika server tujuan tidak mendukung Ed25519:
```Bash
ssh-keygen -t rsa -b 4096 -C "email_anda@example.com"
```
- Copy SSH
  - Cara 1 : Menggunakan command
    ```bash
    ssh-copy-id -p <port> root@IP_SERVER_ANDA`
    ```
    - jika saat membuat keygen nama filenya diubah
    ```bash
    ssh-copy-id -i <direktori_file_ssh>.pub -p <port> root@IP_SERVER_ANDA`

    # contoh:
    # ssh-copy-id -i ~/.ssh/inikeygen.pub -p <2012> root@112.112.211.123`
    ```
  - Cara 2 :
    - Tampilkan isi kunci publik di terminal komputer Anda:
      ```Bash
      cat ~/.ssh/<nama_keygen>.pub
      ```
    - Salin (copy) seluruh baris teks yang muncul.
    - Login ke server Anda sebagai root, lalu tempel (paste) teks tersebut ke dalam file /root/.ssh/authorized_keys:
      ```Bash
      echo "PASTE_TEKS_KUNCI_DISINI" >> /root/.ssh/authorized_keys
      ```
    - Atur hak akses folder dan file di server agar aman:
      ```Bash
      chmod 700 /root/.ssh
      chmod 600 /root/.ssh/authorized_keys
      ```
- Tes login apakah perlu password
```bash
ssh root@<ip> -p <port>
# klo nama file dubah
ssh -i ~/.ssh/<nama_file_yang_dicopy> root@<ip> -p <port>
```
---
# KONFIGURASI SSH
- buat file di `/etc/ssh/sshd_config.d/<nama_file>.conf` atau klo mau langsung di default file `/etc/ssh/sshd_config`
- isi yang dibutuhkan, misal :
1. Ganti Port `Port <angka>`. biar ga standar.
2. Jikalau sudah membuat ssh keygen dengan client `PermitRootLogin prohibit-password`. biar hanya bisa akses via keygen file.
3. `PasswordAuthentication no` Mematikan login password untuk semua akun (root maupun user biasa). Wajib pakai SSH Key.
- restart
```bash
sshd -t
sudo systemctl restart sshd
```
---
# MENAMPILKAN YANG LOGIN KE SYSTEM
`w` atau `watch -n1 w` untuk interaktif mode
---
