- cd `/etc/NetworkManager/system-connections`
- template
```nmconnection
[connection]
id=Tecno Pova 5
uuid=<YOUR UUID>
type=wifi
interface-name=wlan0
timestamp=1785322223

[wifi]
mode=infrastructure
ssid=<YOUR SSID>

[wifi-security]
auth-alg=open
key-mgmt=wpa-psk
psk=<YOUR PASSWORD>

[ipv4]
address1=<YOUR IP WANT>/24,<DNS (AP)>
dns=8.8.8.8;1.1.1.1;
method=manual

[ipv6]
addr-gen-mode=default
method=auto

[proxy]

```
