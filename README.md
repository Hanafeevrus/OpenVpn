# VPN_OpenVpn   
[tun\Vagrantfile](https://github.com/Hanafeevrus/VPN_OpenVpn/tree/master/tun) поднимает 2 машины с настроенным туннелем в режиме tun.   
[tap\Vagrantfile](https://github.com/Hanafeevrus/VPN_OpenVpn/tree/master/tap) поднимает 2 машины с настроенным туннелем в режиме tap.   

##### TUN   
TUN (сетевой туннель) работает на сетевом уровне модели OSI, оперируя IP пакетами в следствии чего broadcast    
и multicast не работает. Проверить можно следующим образом.   
```
sudo arping -I tun1 10.0.0.13   
Interface "tun1" is not ARPable   
```
  где - tun1 интерфейс туннеля. Интерфейсу не присвоем MAC адрес.   
  ```
  tun1@eth1: <POINTOPOINT,NOARP,UP,LOWER_UP> mtu 1492 qdisc noqueue state UNKNOWN group default qlen 1000
    link/ipip 192.168.15.10 peer 192.168.15.11
    inet 10.0.0.1/24 scope global tun1
       valid_lft forever preferred_lft forever
 ```    
 
##### TAP   
TAP эмулирует Ethernet устройство и работает на канальном уровне модели OSI, оперируя кадрами Ethernet.   
Работает broadcast, multicast и loopback. Интерфейсу присваивается MAC. Проверить можно следующим образом. При проверке ARP запроса   
должен отобразится в Unicast ответе MAC адрес.
```   
arping -I tap0 10.0.0.10
ARPING 10.0.0.10 from 10.0.0.11 tap0
Unicast reply from 10.0.0.10 [9A:79:33:2C:D8:11]  1.095ms   
```   
```   
tap0@NONE: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1462 qdisc pfifo_fast state UNKNOWN group default qlen 1000
    link/ether 82:f9:7b:7c:1c:f6 brd ff:ff:ff:ff:ff:ff
    inet 10.0.0.11/24 scope global tap0
       valid_lft forever preferred_lft forever
    inet6 fe80::80f9:7bff:fe7c:1cf6/64 scope link 
       valid_lft forever preferred_lft forever     
       ```
