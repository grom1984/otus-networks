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

Создадим Loopback-адреса для анонсов информации iBGP \
Номер Loopback-интерфейса равен номеру AS\
Сетевые настройки для Loopback
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
 network 10.0.1.14 0.0.0.0 area 0

#################
# Настройки R15 #
#################

conf t

interface Loopback1001
 ip address 100.0.1.15 255.255.255.255
 ipv6 enable
 ipv6 address FC00::15/128
 ipv6 address FE80::15 link-local

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
 network 10.0.1.15 0.0.0.0 area 0
 
```
</details>

#### 4. В офисе С.-Петербург настроить iBGP. (Не использовать протокол OSPF)

Настроим маршрутизаторы R16, R17, R18, R32:
- Создадим loopback-интерфейсы для iBGP;
- Настроим iBGP;
- Анонсируем интерфейсы по iBGP.

Номер Loopback-интерфейсов равен номеру AS.
Для удобства настройки iBGP применим __peer-group__

<details>
 <summary>Настройки iBGP между R14-R15</summary>

 ``` bash
#################
# Настройки R18 #
#################

conf t

interface Loopback2042
 ip address 100.0.2.18 255.255.255.255
 ipv6 enable
 ipv6 address FC00::18/128
 ipv6 address FE80::18 link-local

router bgp 2042
 neighbor AS2042 peer-group
 neighbor AS2042 remote-as 2042
 neighbor AS2042 update-source Loopback2042
 
 neighbor 100.0.2.17 peer-group AS2042 
 neighbor 100.0.2.16 peer-group AS2042
 neighbor 100.0.2.32 peer-group AS2042
 neighbor FC00::17 peer-group AS2042 
 neighbor FC00::16 peer-group AS2042
 neighbor FC00::32 peer-group AS2042
 
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

interface Loopback2042
 ip address 100.0.2.17 255.255.255.255
 ipv6 enable
 ipv6 address FC00::17/128
 ipv6 address FE80::17 link-local

router bgp 2042
 neighbor AS2042 peer-group
 neighbor AS2042 remote-as 2042
 neighbor AS2042 update-source Loopback2042
 
 neighbor 100.0.2.18 peer-group AS2042 
 neighbor 100.0.2.16 peer-group AS2042
 neighbor 100.0.2.32 peer-group AS2042
 neighbor FC00::18 peer-group AS2042 
 neighbor FC00::16 peer-group AS2042
 neighbor FC00::32 peer-group AS2042
 
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

interface Loopback2042
 ip address 100.0.2.16 255.255.255.255
 ipv6 enable
 ipv6 address FC00::16/128
 ipv6 address FE80::16 link-local

router bgp 2042
 neighbor AS2042 peer-group
 neighbor AS2042 remote-as 2042
 neighbor AS2042 update-source Loopback2042
 
 neighbor 100.0.2.18 peer-group AS2042 
 neighbor 100.0.2.17 peer-group AS2042
 neighbor 100.0.2.32 peer-group AS2042
 neighbor FC00::18 peer-group AS2042 
 neighbor FC00::17 peer-group AS2042
 neighbor FC00::32 peer-group AS2042
 
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

interface Loopback2042
 ip address 100.0.2.32 255.255.255.255
 ipv6 enable
 ipv6 address FC00::32/128
 ipv6 address FE80::32 link-local

router bgp 2042
 neighbor AS2042 peer-group
 neighbor AS2042 remote-as 2042
 neighbor AS2042 update-source Loopback2042
 
 neighbor 100.0.2.18 peer-group AS2042 
 neighbor 100.0.2.17 peer-group AS2042
 neighbor 100.0.2.16 peer-group AS2042
 neighbor FC00::18 peer-group AS2042 
 neighbor FC00::17 peer-group AS2042
 neighbor FC00::16 peer-group AS2042
 
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



