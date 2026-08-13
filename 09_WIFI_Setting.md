- cd `/etc/NetworkManager/system-connections`
- template 1
```nmconnection
[connection]
id=<YOUR_ID_1>
uuid=<YOUR_UUID_1>
type=wifi
autoconnect=true
autoconnect-priority=10

[wifi]
mode=infrastructure
ssid=<YOUR_WIFI_NAME>

[wifi-security]
key-mgmt=wpa-psk
psk=<PASSWORD_1>

[ipv4]
address1=192.168.1.111/24,192.168.1.1
dns=8.8.8.8;1.1.1.1;
method=manual

[ipv6]
method=ignore

[proxy]
```
- template 2
```nmconnection
[connection]
id=<YOUR_ID_2>
uuid=<YOUR_UUID_2>
type=wifi
autoconnect=true
autoconnect-priority=9

[wifi]
mode=infrastructure
ssid=<YOUR_WIFI_NAME>

[wifi-security]
key-mgmt=wpa-psk
psk=<PASSWORD_2>

[ipv4]
address1=192.168.1.111/24,192.168.1.1
dns=8.8.8.8;1.1.1.1;
method=manual

[ipv6]
method=ignore

[proxy]
```
- tes
```bash
 sudo nmcli connection load /etc/NetworkManager/system-connections/
```
- reload
```bash
 nmcli connection reload
```
-show
```bash
 nmcli connection show
```
