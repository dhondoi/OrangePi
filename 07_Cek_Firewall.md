# Cek Firewall Bawaan Kernel (iptables)
Karena UFW hanyalah interface pemanis, firewall asli Linux sebenarnya bernama iptables. Cek aturan iptables dengan:
```Bash
sudo iptables -L -n -v
```
Jika output pada baris Chain INPUT menunjukkan target ACCEPT dan tidak ada aturan pembatasan khusus, artinya tidak ada pemblokiran port di dalam Orange Pi Anda.

# Opsi: Install UFW Secara Manual (Jika Ingin Pakai)
Jika Anda ingin kemudahan pengelolaan firewall khas UFW di Orange Pi, Anda bisa menginstalnya dengan mudah:

```Bash
sudo apt update
sudo apt install ufw -y
# Izinkan SSH dulu agar tidak terkunci dari server!
sudo ufw allow 22/tcp
# Izinkan HTTP dan HTTPS untuk Nginx
sudo ufw allow 80/tcp
sudo ufw allow 443/tcp
# Aktifkan UFW
sudo ufw enable
```
