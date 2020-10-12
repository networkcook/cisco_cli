
# Cisco IPSec VPN 
## GRE Over IPSec

> IKE1, IKE2, 初始化和密钥阶段配置

```
hostname xx123xx
!
ip domain name hktmpls.com
! 
crypto keyring ce-keyring  
  pre-shared-key address 0.0.0.0 0.0.0.0 key hkt-ipsec-vpn
!

crypto isakmp policy 1
 encr aes
 authentication pre-share
 group 2
crypto isakmp identity hostname
crypto isakmp keepalive 60 10 periodic
crypto isakmp profile ce-isakmp-profile
   keyring ce-keyring
   match identity address 0.0.0.0 
   initiate mode aggressive
!
crypto ipsec transform-set default-transform-set esp-aes esp-sha-hmac 
 mode tunnel
!
crypto ipsec profile ce-ipsec-profile
 set transform-set default-transform-set 
 set isakmp-profile ce-isakmp-profile
!
```

> 配置WAN端口的IP地址, 以及去往IPSecGW的路由.

```
interface GigabitEthernet0/0/0
 ip address 10.178.254.131 255.255.255.0
 load-interval 30
 negotiation auto
!
ip route 14.23.151.100 255.255.255.255 10.178.254.254
!
```

> 配置 loopback端口以及 tunnel 端口

```
interface Loopback0
 ip address 10.240.21.218 255.255.255.255
!
interface Tunnel0
 description XP798813
 ip unnumbered Loopback0
 ip mtu 1400
 ip tcp adjust-mss 1360
 load-interval 30
 tunnel source GigabitEthernet0/0/0
 tunnel mode ipsec ipv4
 tunnel destination 14.23.151.100
 tunnel protection ipsec profile ce-ipsec-profile
!
```

> 配置LAN-IP-Address,

```
interface GigabitEthernet0/0/1
 ip address 192.168.10.1 255.255.255.0
 negotiation auto
!
```

> 配置动态路由协议, EIGRE:

宣告 WAN, LAN 的地址网段

```
router eigrp 90
 redistribute static
 redistribute connected
 passive-interface default
 no passive-interface Tunnel0

 network 10.240.21.218 0.0.0.0
 network 192.168.10.1 0.0.0.25

 no auto-summary
!
```

> 某些设备或IOS版本无法输入纯数字的hostname, 可以使用如下方式:

```
no ip domain name hktmpls.com
hostname **1234567.hktmpls.com**
```
