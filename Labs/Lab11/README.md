# Лабораторная работа №11. Основы BGP.

### Задание:
1. eBGP между офисом Москва и двумя провайдерами - Киторн и Ламас
2. Настроить eBGP между провайдерами Киторн и Ламас
3. Настроить eBGP между Ламас и Триада
4. eBGP между офисом С.-Петербург и провайдером Триада
5. Организовать IP доступность между офисами Москва и С.-Петербург
6. Настроить отслеживание линка через технологию IP SLA

Конфигурационные файлы [здесь](config/)

### Решение:


### Топология

![network](network.png)

##### Таблица соединений с соседями по BGP 


| ASN    | Routers  | Nbr ASN | Neighbor IP          | 
|:-------|:----|:--------|:---------------------|
| 1001  | R14->R22  | 101   | 7.7.7.22          |
| 1001  | R14->R22  | 101   | 2001:FFCC:1000:1422::22          | 
| 1001  | R15->R21  | 301   | 2.2.2.21 |
| 1001  | R15->R21  | 301   | 2001:FFCC:1000:1521::21 |
| 101  | R22->R14  | 1001   | 7.7.7.14 |
| 101  | R22->R14  | 1001   | 2001:FFCC:1000:1422::14 |
| 301  | R21->R15  | 1001  | 2.2.2.15  |
| 301  | R21->R15  | 1001  | 2001:FFCC:1000:1521::15  |
| 301  | R21->R22  | 101  |  212.188.8.50 |
| 301 | R21->R22  | 101 | 2001:FFCC:5000:2122::22  |
| 101  | R22->R21  | 301 | 212.188.8.49  |
| 101 | R22->R21  | 301  | 2001:FFCC:5000:2122::21  |
| 301 | R21->R24  | 520  | 178.248.237.50  |
| 301 | R21->R24  | 520  | 2001:FFCC:7000:2124::24  |
| 502 | R24->R21  | 301  | 178.248.237.49  |
| 502  | R24->R21  | 301  | 2001:FFCC:7000:2124::21  |
| 2042  | R18->R24  | 502 | 87.250.250.24  |
| 2042  | R18->R24  | 502 | 2001:FFCC:2000:1824::24  |
| 2042 | R18->R26  | 502  | 82.208.114.26  |
| 2042  | R18->R26  | 502   | 2001:FFCC:2000:1826::26  |
| 520  | R24->R18  | 2042  | 87.250.250.18  |
| 520  | R24->R18  | 2042  | 2001:FFCC:2000:1824::18  |
| 520  | R26->R18  | 2042  | 82.208.114.18  |
| 520  | R26->R18  | 2042  | 2001:FFCC:2000:1826::18  |






### 1. Настроить eBGP между офисом Москва и провайдерами Киторн и Ламас

Настроим eBGP между маршрутизаторами R14-R22 и R15-R21.

<details>
 <summary>Пример настройки eBGP между R14-R22</summary>

``` bash
#################
# Настройки R14 #
#################

conf t
router bgp 1001
neighbor 7.7.7.22 remote-as 101
neighbor 2001:FFCC:1000:1422::22 remote-as 101

#################
# Настройки R22 #
#################

conf t
router bgp 101
neighbor 7.7.7.14 remote-as 1001
neighbor 2001:FFCC:1000:1422::14 remote-as 1001

```
</details>


### 2. Настроить eBGP между провайдерами Киторн и Ламас

Настроим eBGP между маршрутизаторами R21-R22.

<details>
 <summary>Настройки R21 и R22</summary>

``` bash
#################
# Настройки R21 #
#################

conf t
router bgp 301
neighbor 212.188.8.50 remote-as 101
neighbor 2001:FFCC:5000:2122::22 remote-as 101

#################
# Настройки R22 #
#################

conf t
router bgp 101
neighbor 212.188.8.49 remote-as 301
neighbor 2001:FFCC:5000:2122::21 remote-as 301

```
</details>

### 3. Настроить eBGP между Ламас и Триада

Настроим eBGP между маршрутизаторами R21-R24.

<details>
 <summary>Настройки R21 и R24</summary>

``` bash
#################
# Настройки R21 #
#################

conf t
router bgp 301
neighbor 178.248.237.50 remote-as 520
neighbor 2001:FFCC:7000:2124::24 remote-as 520

#################
# Настройки R24 #
#################

conf t
router bgp 520
neighbor 178.248.237.49 remote-as 301
neighbor 2001:FFCC:7000:2124::21 remote-as 301

```
</details>

### 4. eBGP между офисом С.-Петербург и провайдером Триада

Настроим eBGP между R18-R24 и R18-R26

<details>
 <summary>Настройки R18, R24, R26</summary>

``` bash
#################
# Настройки R18 #
#################

conf t
router bgp 2042
neighbor 87.250.250.24 remote-as 520
neighbor 2001:FFCC:2000:1824::24 remote-as 520
neighbor 82.208.114.26 remote-as 520
neighbor 2001:FFCC:2000:1826::26 remote-as 520

#################
# Настройки R24 #
#################

conf t
router bgp 520
neighbor 87.250.250.18 remote-as 2042
neighbor 2001:FFCC:2000:1824::18 remote-as 2042

#################
# Настройки R26 #
#################

conf t
router bgp 520
neighbor 82.208.114.18 remote-as 2042
neighbor 2001:FFCC:2000:1826::18 remote-as 2042


```
</details>

### 6. Настроить отслеживание линка через технологию IP SLA

Настроим IP SLA для R18.
На первом этапе пропишем маршрут по-умолчанию для двух линков (к R24 и R26). Чтобы работала автоматическая балансировка трафика укажем одинаковые метрики для обоих маршрутов. Затем распредлим трафик от между чётными/нечётными подсетями. Для "чётных" подсетей маршрут по-умолчанию будет через R26, для нечётных - через R24.Затем, настроим PBR и отслеживание линков через IP SLA.

P.S. Для IPv6 нет параметра _verify-availability_ поэтому невозможно настроить IP SLA на эмуляторе eve-ng.

<details>
 <summary>Настройка отслеживания линков на R18</summary>

``` bash

#######
# ACL #
#######

ip access-list standard ACL_PBR_TO_R24
  permit 10.10.1.0 0.0.254.255
  deny any
  exit
ipv6 access-list ACL_PBR_TO_R24-v6
  permit 2001:FFCC:2000:3::/64 any
  deny any any
  exit
ip access-list standard ACL_PBR_TO_R26
  permit 10.10.0.0 0.0.254.255
  deny any
  exit
ipv6 access-list ACL_PBR_TO_R26-v6
  permit 2001:FFCC:3000:2::/64 any
  deny any any
  exit

##############
# Route-map  #
##############

route-map PBR_TO_R24 permit 10
  match ip address ACL_PBR_TO_R24
  set ip next-hop verify-availability 87.250.250.24 1 track 24
  exit

route-map PBR_TO_R24-v6 permit 10
  match ipv6 address ACL_PBR_TO_R24-v6
  set ipv6 next-hop 2001:FFCC:2000:1824::24
  exit

route-map PBR_TO_R26 permit 10
  match ip address ACL_PBR_TO_R26
  set ip next-hop verify-availability 82.208.114.26 1 track 26
  exit

route-map PBR_TO_R26-v6 permit 10
  match ipv6 address ACL_PBR_TO_R26-v6
  set ipv6 next-hop 2001:FFCC:2000:1826::26
  exit

###########
## IP SLA #
###########

ip sla 24
  icmp-echo 87.250.250.24 source-interface e0/2
  frequency 15
ip sla schedule 24 life forever start-time now
track 24 ip sla 24 reachability

ip sla 26
  icmp-echo 82.208.114.26 source-interface e0/3
  frequency 15
ip sla schedule 26 life forever start-time now
track 26 ip sla 26 reachability

#############
# Interface #
#############

int e0/2
  ip policy route-map PBR_TO_R24
  ipv6 policy route-map PBR_TO_R24-v6
exit

int e0/3
  ip policy route-map PBR_TO_R26
  ipv6 policy route-map PBR_TO_R26-v6
exit


```
</details>


### Конфигурационные файлы [здесь](config/)