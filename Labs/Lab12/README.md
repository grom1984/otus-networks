# Лабораторная работа №12. iBGP.

### Задание:

1. Настроить iBGP в офисе Москва между маршрутизаторами R14 и R15;
2. Настроите iBGP в провайдере Триада;
3. Настроить офис Москва так, чтобы приоритетным провайдером стал Ламас;
4. В офисе С.-Петербург работает протокол iBGP. (Не использовать протокол OSPF);
5. Настроить офис С.-Петербург так, чтобы трафик до любого офиса распределялся по двум линкам одновременно;
6. Все сети в лабораторной работе должны иметь IP связность;
7. План работы и изменения зафиксированы в документации.

Конфигурационные файлы [здесь](config/)

### Решение:


### Топология


#### 1. Настроить iBGP в офисе Москва между маршрутизаторами R14 и R15

Создадим Loopback-адреса и анонсируем Loopback-интерфейсы через OSPF. Номер Loopback-интерфейса равен номеру AS.

Сетевые настройки для Loopback:\
ipv4: 100.0.__[номер_офиса]__.__[номер_роутера]__ \
ipv6: FC00::__{номер маршрутизатора}__\
ipv6 LL: FE80::__[номер_роутера]__

Например, для маршрутизатора R14\
ipv4: __100.0.1.14__\
ipv6: __FC00:14/128__\
ipv6 LL: __FE80::14__

Настроим на роутерах R14 и R15 параметры:
   - __next-hop-self__;
   - __update-source__; (Loopback)


<details>
 <summary>Настройки iBGP между R14-R15</summary>

``` bash
#################
# Настройки R14 #
#################

conf t

interface Loopback1001
 ip address 100.0.1.14 255.255.255.255
 ipv6 enable
 ipv6 address FC00::14/128
 ipv6 address FE80::14 link-local
 ipv6 ospf 1 area 0
 

router bgp 1001
 neighbor 100.0.1.15 remote-as 1001
 neighbor 100.0.1.15 update-source Loopback1001
 neighbor 100.0.1.15 next-hop-self
 neighbor FC00::15 remote-as 1001
 neighbor FC00::15 update-source Loopback1001
 neighbor FC00::15 next-hop-self

address-family ipv4
 neighbor 100.0.1.15 activate
 no neighbor FC00::15 activate
 
address-family ipv6
 no neighbor 100.0.1.15 activate
 neighbor FC00::15 activate

# Анонсируем loopback1001-интерфейс в OSPF и запретим его анонс через внешний интерфейс
# Политика по-умолчанию passive-interface default

router ospf 1
 network 100.0.1.14 0.0.0.0 area 0

#################
# Настройки R15 #
#################

conf t

interface Loopback1001
 ip address 100.0.1.15 255.255.255.255
 ipv6 enable
 ipv6 address FC00::15/128
 ipv6 address FE80::15 link-local
 ipv6 ospf 1 area 0

router bgp 1001
 neighbor 100.0.1.14 remote-as 1001
 neighbor 100.0.1.14 update-source Loopback1001
 neighbor 100.0.1.14 next-hop-self
 neighbor FC00::14 remote-as 1001
 neighbor FC00::14 update-source Loopback1001
 neighbor FC00::14 next-hop-self

address-family ipv4
 neighbor 100.0.1.14 activate
 no neighbor FC00::14 activate
 
address-family ipv6
 no neighbor 100.0.1.14 activate
 neighbor FC00::14 activate

# Анонсируем loopback1001-интерфейс в OSPF и запретим его анонс через внешний интерфейс
# Политика по-умолчанию passive-interface default

router ospf 1
 network 100.0.1.15 0.0.0.0 area 0
 
```
</details>

Проверка установки BGP-сессии между роутерами.

![iBGP_ipv4_R14-R15](iBGP_ipv4_R14-R15.png)

![iBGP_ipv6_R14-R15](iBGP_ipv6_R14-R15.png)


#### 2. Настроите iBGP в провайдере Триада




#### 4. В офисе С.-Петербург настроить iBGP. (Не использовать протокол OSPF)

Настроим маршрутизаторы R16, R17, R18, R32:
- Создадим loopback-интерфейсы для iBGP;
- Настроим EIGRP и анонсируем loopback-интерфейсы;
- Настроим iBGP;
- Анонсируем интерфейсы по iBGP.

Номер Loopback-интерфейсов равен номеру AS.
Для удобства настройки iBGP применим __peer-group__

<details>
 <summary>Настройка EIGRP и анонсирование Lo-интерфейсов</summary>

``` bash
#############
# EIGRP R17 #
#############

conf t
ipv6 unicast-routing
ipv6 router eigrp 1
 eigrp router-id 0.0.2.17
 passive-interface default
 no passive-interface e0/1
 no shutdown
end

conf t
interface Loopback2042
 ip address 100.0.2.17 255.255.255.255
 ipv6 enable
 ipv6 address FC00::17/128
 ipv6 address FE80::17 link-local
 ipv6 eigrp 1
end
 
conf t
ipv6 unicast-routing
ipv6 router eigrp 1
 eigrp router-id 0.0.2.17
 passive-interface default
 no passive-interface e0/1
 no shutdown
end
 
conf t
router eigrp 1
 eigrp router-id 0.0.2.17
 network 100.0.2.17 0.0.0.0
   
 passive-interface default
 no passive-interface e0/1

end

conf t
 int range e0/0-2
 ipv6 eigrp 1
end

#############
# EIGRP R18 #
#############

conf t
ipv6 unicast-routing
ipv6 router eigrp 1
 eigrp router-id 0.0.2.18
 passive-interface default
 no passive-interface e0/1
 no passive-interface e0/0
 no shutdown
end

conf t
interface Loopback2042
 ip address 100.0.2.18 255.255.255.255
 ipv6 enable
 ipv6 address FC00::18/128
 ipv6 address FE80::18 link-local
 ipv6 eigrp 1
end

conf t
ipv6 unicast-routing
ipv6 router eigrp 1
 eigrp router-id 0.0.2.18
 passive-interface default
 no passive-interface e0/1
 no passive-interface e0/0
 no shutdown
end
conf t
router eigrp 1
 eigrp router-id 0.0.2.18
 network 100.0.2.18 0.0.0.0
  
 passive-interface default
 no passive-interface e0/1
 no passive-interface e0/0
end

conf t
 int range e0/0-3
 ipv6 eigrp 1
end
 
#############
# EIGRP R16 #
#############

conf t
ipv6 unicast-routing
ipv6 router eigrp 1
 eigrp router-id 0.0.2.16
 passive-interface default
 no passive-interface e0/1
 no shutdown
end

conf t
interface Loopback2042
 ip address 100.0.2.16 255.255.255.255
 ipv6 enable
 ipv6 address FC00::16/128
 ipv6 address FE80::16 link-local
 ipv6 eigrp 1
end
 
conf t
ipv6 unicast-routing
ipv6 router eigrp 1
 eigrp router-id 0.0.2.16
 passive-interface default
 no passive-interface e0/1
 no passive-interface e0/3
 no shutdown

conf t
router eigrp 1
 eigrp router-id 0.0.2.16
 network 100.0.2.16 0.0.0.0
  
 passive-interface default
 no passive-interface e0/1
 no passive-interface e0/3
end

conf t
 int range e0/0-3
 ipv6 eigrp 1
end
 
#############
# EIGRP R32 #
#############

conf t
ipv6 unicast-routing
ipv6 router eigrp 1
 eigrp router-id 0.0.2.32
 passive-interface default
 no passive-interface e0/0
 no shutdown

conf t
interface Loopback2042
 ip address 100.0.2.32 255.255.255.255
 ipv6 enable
 ipv6 address FC00::32/128
 ipv6 address FE80::32 link-local
 ipv6 eigrp 1
end

conf t
ipv6 unicast-routing
ipv6 router eigrp 1
 eigrp router-id 0.0.2.32
 passive-interface default
 no passive-interface e0/0
 no shutdown

conf t
router eigrp 1
 eigrp router-id 0.0.2.32
 network 100.0.2.32 0.0.0.0
  
 passive-interface default
 no passive-interface e0/0
 end
 
 conf t
 int range e0/0
 ipv6 eigrp 1
end

```
</details>

<details>
 <summary>Проверка работоспособности EIGRP ipv4/ipv6 в офисе СПб</summary>

__Доступность Lo-интерфейсов между собой__


![ping_Lo_Spb](ping_Lo_Spb.png)

__Таблица роутинга ipv4 на R17__ 
```bash

R17#sh ip route eigrp

Gateway of last resort is not set

      10.0.0.0/8 is variably subnetted, 8 subnets, 3 masks
D        10.10.12.0/24 [90/307200] via 10.10.10.18, 01:07:57, Ethernet0/1
D        10.10.13.0/24 [90/332800] via 10.10.10.18, 01:05:15, Ethernet0/1
      82.0.0.0/27 is subnetted, 1 subnets
D        82.208.114.0 [90/307200] via 10.10.10.18, 01:07:51, Ethernet0/1
      87.0.0.0/27 is subnetted, 1 subnets
D        87.250.250.0 [90/307200] via 10.10.10.18, 01:07:51, Ethernet0/1
      100.0.0.0/32 is subnetted, 4 subnets
D        100.0.2.16 [90/435200] via 10.10.10.18, 01:05:15, Ethernet0/1
D        100.0.2.18 [90/409600] via 10.10.10.18, 01:13:14, Ethernet0/1
D        100.0.2.32 [90/460800] via 10.10.10.18, 00:41:04, Ethernet0/1
```

__Таблица роутинга ipv6 на R17__ 
```bash
R17#sh ipv6 route eigrp

D   2001:FFCC:2000:1618::/64 [90/307200]
     via FE80::18, Ethernet0/1
D   2001:FFCC:2000:1632::/64 [90/332800]
     via FE80::18, Ethernet0/1
D   2001:FFCC:2000:1824::/64 [90/307200]
     via FE80::18, Ethernet0/1
D   2001:FFCC:2000:1826::/64 [90/307200]
     via FE80::18, Ethernet0/1
D   FC00::18/128 [90/409600]
     via FE80::18, Ethernet0/1
D   FC00::32/128 [90/460800]
     via FE80::18, Ethernet0/1

```

 </details>

<details>
 <summary>Настройки iBGP между R16, R17, R18, R32</summary>

 ``` bash
#################
# Настройки R18 #
#################

conf t

router bgp 2042
 neighbor AS2042 peer-group
 neighbor AS2042 remote-as 2042
 neighbor AS2042 update-source Loopback2042
 neighbor AS2042 next-hop-self
 
 neighbor AS2042-6 peer-group
 neighbor AS2042-6 remote-as 2042
 neighbor AS2042-6 update-source Loopback2042
 neighbor AS2042-6 next-hop-self
 
 neighbor 100.0.2.17 peer-group AS2042 
 neighbor 100.0.2.16 peer-group AS2042
 neighbor 100.0.2.32 peer-group AS2042
 neighbor FC00::17 peer-group AS2042-6 
 neighbor FC00::16 peer-group AS2042-6
 neighbor FC00::32 peer-group AS2042-6
 
address-family ipv4
 neighbor 100.0.2.17 activate 
 neighbor 100.0.2.16 activate
 neighbor 100.0.2.32 activate
 no neighbor FC00::17 activate 
 no neighbor FC00::16 activate
 no neighbor FC00::32 activate
 
address-family ipv6
 no neighbor 100.0.2.17 activate 
 no neighbor 100.0.2.16 activate
 no neighbor 100.0.2.32 activate
 neighbor FC00::17 activate 
 neighbor FC00::16 activate
 neighbor FC00::32 activate


#################
# Настройки R17 #
#################

conf t


router bgp 2042
 neighbor AS2042 peer-group
 neighbor AS2042 remote-as 2042
 neighbor AS2042 update-source Loopback2042
 neighbor AS2042 next-hop-self
 
 neighbor AS2042-6 peer-group
 neighbor AS2042-6 remote-as 2042
 neighbor AS2042-6 update-source Loopback2042
 neighbor AS2042-6 next-hop-self
 
 
 neighbor 100.0.2.18 peer-group AS2042 
 neighbor 100.0.2.16 peer-group AS2042
 neighbor 100.0.2.32 peer-group AS2042
 neighbor FC00::18 peer-group AS2042-6 
 neighbor FC00::16 peer-group AS2042-6
 neighbor FC00::32 peer-group AS2042-6
 
address-family ipv4
 neighbor 100.0.2.18 activate 
 neighbor 100.0.2.16 activate
 neighbor 100.0.2.32 activate
 no neighbor FC00::18 activate 
 no neighbor FC00::16 activate
 no neighbor FC00::32 activate
 
address-family ipv6
 no neighbor 100.0.2.18 activate 
 no neighbor 100.0.2.16 activate
 no neighbor 100.0.2.32 activate
 neighbor FC00::18 activate 
 neighbor FC00::16 activate
 neighbor FC00::32 activate
 
#################
# Настройки R16 #
#################

conf t

router bgp 2042
 neighbor AS2042 peer-group
 neighbor AS2042 remote-as 2042
 neighbor AS2042 update-source Loopback2042
 neighbor AS2042 next-hop-self
 
 neighbor AS2042-6 peer-group
 neighbor AS2042-6 remote-as 2042
 neighbor AS2042-6 update-source Loopback2042
 neighbor AS2042-6 next-hop-self
 
 neighbor 100.0.2.18 peer-group AS2042 
 neighbor 100.0.2.17 peer-group AS2042
 neighbor 100.0.2.32 peer-group AS2042
 neighbor FC00::18 peer-group AS2042-6
 neighbor FC00::17 peer-group AS2042-6
 neighbor FC00::32 peer-group AS2042-6
 
address-family ipv4
 neighbor 100.0.2.18 activate 
 neighbor 100.0.2.17 activate
 neighbor 100.0.2.32 activate
 no neighbor FC00::18 activate 
 no neighbor FC00::17 activate
 no neighbor FC00::32 activate
 
address-family ipv6
 no neighbor 100.0.2.18 activate 
 no neighbor 100.0.2.17 activate
 no neighbor 100.0.2.32 activate
 neighbor FC00::18 activate 
 neighbor FC00::17 activate
 neighbor FC00::32 activate

 
#################
# Настройки R32 #
#################

conf t

router bgp 2042
 neighbor AS2042 peer-group
 neighbor AS2042 remote-as 2042
 neighbor AS2042 update-source Loopback2042
 neighbor AS2042 next-hop-self
 
 neighbor AS2042-6 peer-group
 neighbor AS2042-6 remote-as 2042
 neighbor AS2042-6 update-source Loopback2042
 neighbor AS2042-6 next-hop-self
 
 neighbor 100.0.2.18 peer-group AS2042 
 neighbor 100.0.2.17 peer-group AS2042
 neighbor 100.0.2.16 peer-group AS2042
 neighbor FC00::18 peer-group AS2042-6 
 neighbor FC00::17 peer-group AS2042-6
 neighbor FC00::16 peer-group AS2042-6
 
address-family ipv4
 neighbor 100.0.2.18 activate 
 neighbor 100.0.2.17 activate
 neighbor 100.0.2.16 activate
 no neighbor FC00::18 activate 
 no neighbor FC00::17 activate
 no neighbor FC00::16 activate
 
address-family ipv6
 no neighbor 100.0.2.18 activate 
 no neighbor 100.0.2.17 activate
 no neighbor 100.0.2.16 activate
 neighbor FC00::18 activate 
 neighbor FC00::17 activate
 neighbor FC00::16 activate


 ```
</details>


<details>
 <summary>Проверка работоспособности iBGP ipv4/ipv6 в офисе СПб</summary>

Таблица маршрутов iBGP ipv4 [R17]

``` bash
R17#sh ip route bgp

Gateway of last resort is not set

      83.0.0.0/28 is subnetted, 3 subnets
B        83.239.45.16 [200/0] via 100.0.2.18, 00:04:46
B        83.239.45.32 [200/0] via 100.0.2.18, 00:04:46
B        83.239.45.48 [200/0] via 100.0.2.18, 00:04:40
      178.248.0.0/29 is subnetted, 1 subnets
B        178.248.237.48 [200/0] via 100.0.2.18, 00:04:40

```

Таблица маршрутов iBGP ipv6 [R17]

``` bash
R17#sh ipv6 route bgp

B   2001:FFCC:3000:2628::/64 [200/0]
     via 2001:FFCC:2000:1826::26
B   2001:FFCC:7000:2124::/64 [200/0]
     via 2001:FFCC:2000:1824::24
B   2001:FFCC:8000:2324::/64 [200/0]
     via 2001:FFCC:2000:1824::24
B   2001:FFCC:8000:2426::/64 [200/0]
     via 2001:FFCC:2000:1826::26
B   2001:FFCC:8000:2526::/64 [200/0]
     via 2001:FFCC:2000:1826::26

```

</details>