# Cara menghubungkan orangepi ke pc

1. siapkan kabel lan straight
2. buka menu network connection di pc
3. pilih wifi (klik kanan) > properties > tab sharing, centang allow
4. cek pada cmd `arp -a`
5. jika ada ip lain pada pada interface ip pc, coba ping.
6. atau ada interface network lain, cek list ip kemungkinan itu.
7. ketik cmd `ssh root@<ip_yang_ketemu>` untuk pass `orangepi`

# membuat wifi menjadi ip statis

1. ketik `nmtui`
2. masukkan pass wifi
3. edit ipv4 jadi manual
4. `systemctl restart NetworkManager`
5. coba `curl https://google.com` klo sukses berarti lancar
