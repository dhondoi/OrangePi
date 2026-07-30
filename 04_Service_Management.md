# Cek Service Berjalan

```cmd
systemctl list-units --type=service --state=running
```
# Matikan Service

Saat reboot nanti
```cmd
sudo systemctl disable bluetooth.service
```
matikan langsung
```cmd
sudo systemctl stop bluetooth.service
```
