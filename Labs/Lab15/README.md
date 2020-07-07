# Лабораторная работа №15. VPN. GRE. DmVPN

### Задание:

1. Настроить GRE между офисами Москва и С.-Петербург
2. Настроить DMVPN между Москва и Чокурдах, Лабытнанги

Конфигурационные файлы [здесь](config/)

#### 1. Настроить GRE между офисами Москва и С.-Петербург

###### Топология GRE туннелей

![GRE_Tun](GRE_Tun.png)

Создадим виртуальные интерфейсы __tun__ на роутерах R14, R15, R18. Для резервирования сделаем два туннеля R14-R18 (__tun14__) и R15-R18(__tun15__).


| Equip | Port | IP ver.| IP/Mask | tun_src | tun_dst |
|:--- |:--- |:--- |:--- |---:|:--- |
| R14 | tun1418 | ipv4 | 192.168.0.1/30 | 77.77.77.114 | 178.178.178.118 |
| R14 | tun14186 | ipv6 | FD00:FFCC:1418::0/127 | 2001:FFCC:1001:77::114 | 2001:FFCC:2042:178::118 |
| R14 | tun14186 | ipv6 LL | FE80::14 |  |  |
| R18 | tun1418 | ipv4 | 192.168.0.2/30 | 178.178.178.118 | 77.77.77.114 |
| R18 | tun14186 | ipv6 | FD00:FFCC:1418::1/127 | 2001:FFCC:2042:178::118  | 2001:FFCC:1001:77::114 |
| R18 | tun14186 | ipv6 LL | FE80:18  |  |  |
| R15 | tun1518 | ipv4 | 192.168.0.5/30 | 77.77.77.115 | 178.178.178.115 |
| R15 | tun15186 | ipv6 | FD00:FFCC:1518::0/127 | 2001:FFCC:1001:77:115 | 2001:FFCC:2042:178::119 |
| R15 | tun15186 | ipv6 LL | FE80:15 |  |  |
| R18 | tun1518 | ipv4 | 192.168.0.6/30 | 178.178.178.115 | 77.77.77.115 |
| R18 | tun15186 | ipv6 | FD00:FFCC:1518::1/127 | 2001:FFCC:2042:178::119 | 2001:FFCC:1001:77:115 |
| R18 | tun15186 | ipv6 LL |  |  |  |

<details>
 <summary>Настройки NAT на R14-R15</summary>

``` bash
###################
# Настройка R14   #
###################

conf t
int tun1418
  description "GREv4 Tunnel MSK-SPb to R18"
  ip addr 192.168.0.1 255.255.255.252
  ip mtu 1400
  ip tcp adjust-mss 1360
  tunnel source 77.77.77.114
  tunnel destination 178.178.178.118
  tunnel mode gre ipv4
  no shutdown

int tun14186
  description "GREv6 Tunnel MSK-SPb to R18"
  ipv6 enable
  ipv6 address FE80::14 link-local
  ipv6 address FD00:FFCC:1418::0/127
  tunnel source 2001:FFCC:1001:77::114
  tunnel destination 2001:FFCC:2042:178::118
  tunnel mode gre ipv6
  no shutdown

###################
# Настройка R18   #
###################

conf t
int tun1418
  description "GREv4 Tunnel SPb-MSK to R14"
  ip addr 192.168.0.2 255.255.255.252
  ip mtu 1400
  ip tcp adjust-mss 1360
  tunnel source 178.178.178.118
  tunnel destination 77.77.77.114
  tunnel mode gre ipv4
  no shutdown

int tun14186  
  description "GREv6 Tunnel MSK-SPb to R14"
  ipv6 enable
  ipv6 address FE80::18 link-local
  ipv6 address FD00:FFCC:1418::1/127
  tunnel source 2001:FFCC:2042:178::118
  tunnel destination 2001:FFCC:1001:77::114
  tunnel mode gre ipv6
  no shutdown
  
int tun1518
  description "GREv4 Tunnel SPb-MSK to R15"
  ip addr 192.168.0.6 255.255.255.252
  ip mtu 1400
  ip tcp adjust-mss 1360
  tunnel source 178.178.178.119
  tunnel destination 77.77.77.115
  tunnel mode gre ipv4
  no shutdown

int tun15186  
  description "GREv6 Tunnel SPb-MSK to R15"
  ipv6 enable
  ipv6 address FE80::18 link-local
  ipv6 address FD00:FFCC:1518::1/127
  tunnel source 2001:FFCC:2042:178::119
  tunnel destination 2001:FFCC:1001:77::115
  tunnel mode gre ipv6
  no shutdown 
  
###################
# Настройка R15   #
###################

conf t
int tun1518
  description "GREv4 Tunnel SPb-MSK to R18"
  ip addr 192.168.0.5 255.255.255.252
  ip mtu 1400
  ip tcp adjust-mss 1360
  tunnel source 77.77.77.115
  tunnel destination 178.178.178.119
  tunnel mode gre ipv4
  no shutdown

int Tun15186
  description "GREv6 Tunnel SPb-MSK to R18"
  ipv6 enable
  ipv6 address FE80::15 link-local
  ipv6 address FD00:FFCC:1518::0/127
  tunnel source 2001:FFCC:1001:77::115
  tunnel destination 2001:FFCC:2042:178::119
  tunnel mode gre ipv6
  no shutdown


```
</details>

#### 2. Настроить DMVPN между Москва и Чокурдах, Лабытнанги

###### Топология DMVPN туннелей

