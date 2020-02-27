# VPN_OpenVpn   
### Задача    
1. Между двумя виртуалками поднять vpn в режимах:
- tun
- tap   
2. Поднять RAS на базе OpenVPN с клиентскими сертификатами,
подключиться с локальной машины на виртуалку.   

### Решение   
1. [tun\Vagrantfile](https://github.com/Hanafeevrus/VPN_OpenVpn/tree/master/tun) поднимает 2 машины с настроенным туннелем в режиме tun.   
[tap\Vagrantfile](https://github.com/Hanafeevrus/VPN_OpenVpn/tree/master/tap) поднимает 2 машины с настроенным туннелем в режиме tap.   
2. [RAS_OpenVpn\Vagrantfile](https://github.com/Hanafeevrus/VPN_OpenVpn/tree/master/RAS_OpenVpn) поднимает OpenVpn server.    
### 1. Описание TUN/TAP.
### TUN   
TUN (сетевой туннель) работает на сетевом уровне модели OSI, оперируя IP пакетами в следствии чего broadcast    
и multicast не работает. Проверить можно следующим образом.   
```
sudo arping -I tun1 10.0.0.13   
Interface "tun1" is not ARPable   
```
  где - tun1 интерфейс туннеля. Интерфейсу не присвоен MAC адрес.   
  ```
  tun1@eth1: <POINTOPOINT,NOARP,UP,LOWER_UP> mtu 1492 qdisc noqueue state UNKNOWN group default qlen 1000
    link/ipip 192.168.15.10 peer 192.168.15.11
    inet 10.0.0.1/24 scope global tun1
       valid_lft forever preferred_lft forever
 ```    
 
### TAP   
TAP эмулирует Ethernet устройство и работает на канальном уровне модели OSI, оперируя кадрами Ethernet.   
Работает broadcast, multicast и loopback. Интерфейсу присваивается MAC. Проверить можно следующим образом. При проверке ARP запроса   
должен отобразится в Unicast ответе MAC адрес.
```   
arping -I tap0 10.0.0.10
ARPING 10.0.0.10 from 10.0.0.11 tap0
Unicast reply from 10.0.0.10 [9A:79:33:2C:D8:11]  1.095ms   
```   
где - tap0 интерфейс туннеля. Интерфейсу присвоен MAC адрес. 
```   
tap0@NONE: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1462 qdisc pfifo_fast state UNKNOWN group default qlen 1000
    link/ether 82:f9:7b:7c:1c:f6 brd ff:ff:ff:ff:ff:ff
    inet 10.0.0.11/24 scope global tap0
       valid_lft forever preferred_lft forever
    inet6 fe80::80f9:7bff:fe7c:1cf6/64 scope link 
       valid_lft forever preferred_lft forever     
```    
### 2. RAS OpenVPN   
[OpenVPN](https://openvpn.net/about/) — свободная реализация технологии виртуальной частной сети (VPN) с открытым исходным кодом для создания зашифрованных каналов типа точка-точка или сервер-клиенты между компьютерами. Она позволяет устанавливать соединения между компьютерами, находящимися за NAT и сетевым экраном, без необходимости изменения их настроек. OpenVPN была создана Джеймсом Йонаном (James Yonan) и распространяется под лицензией GNU GPL.   
В директории `RAS_OpenVpn\server` сертификаты и ключи предварительно созданы и копируются на сервер при поднятии vagrant. Конфигурационный файл `server.conf`. Сервер настроен на работу в режиме `tun`.    
```   
###server.conf
port 1194                               
proto udp
dev tun
ca /etc/openvpn/ca.crt
cert /etc/openvpn/server.crt
key /etc/openvpn/server.key
dh /etc/openvpn/dh.pem
tls-auth /etc/openvpn/ta.key 0
server 10.200.254.0 255.255.255.0
route 10.200.254.0 255.255.255.0
#push "route 192.168.20.0 255.255.255.0"
ifconfig-pool-persist ipp.txt
client-to-client
#client-config-dir /etc/openvpn/client
keepalive 10 120
comp-lzo
persist-key
persist-tun
status /var/log/openvpn-status.log
log /var/log/openvpn.log
verb 3    
```   
После поднятия машины можно подключиться командой:    
`sudo openvnp --config path/client.conf`    
client.conf - конфигурационный файл клиента.    
```   
### client.conf
dev tun
proto udp
remote 192.168.20.10 1194
client
resolv-retry infinite
#ca ./ca.crt
#cert ./client.crt
#key ./client.key
persist-key
persist-tun
comp-lzo
verb 3
key-direction 1   
```
Проверка подлинности клиента происходит на основе сертификатов которые прописаны в конфигурационном файле клиента в формате:    
```    
...   
key-direction 1   
<ca>    
-----BEGIN CERTIFICATE-----   
...   
-----END CERTIFICATE-----   
</ca>   
<tls-auth>  
-----BEGIN OpenVPN Static key V1-----   
...   
-----END OpenVPN Static key V1-----   
</tls-auth>   
<cert>    
-----BEGIN CERTIFICATE-----   
...   
-----END CERTIFICATE-----   
</cert>   
<key>   
-----BEGIN PRIVATE KEY-----   
...   
-----END PRIVATE KEY-----   
</key>    
<dh>    
----BEGIN DH PARAMETERS-----    
...   
-----END DH PARAMETERS-----   
</dh>   
```   
* key-direction 1 — необходим для tls-auth, в противном случае, работать не будет; ca — ключ центра сертификации (ca.crt); tls-auth — ta.key; cert — открытый сертификат клиента (clients.crt); key — закрытый сертификат клиента (clients.key); dh — сертификат, созданный на базе протокола Диффи Хеллмана.
