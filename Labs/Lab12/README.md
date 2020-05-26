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

Настроим на роутерах R14 и R15 параметры:
- __router-id__ для BGP;
- __next-hop-self__;
- __update-source__;
- __static route__ до сети соседнего ISP;

Создадим Loopback-адреса для анонсов информации iBGP \
ipv4: __100.0.[номер_офиса].[номер_роутера]__ \
ipv6: __{префикс IPv6 для сайта}::{номер маршрутизатора}__\
ipv6 LL: __FE80::[номер_роутера]__

Добавим _ipv6 link-local_ адреса для интерфейсов _Loopback999_ на всех роутерах.
__IPv6-адрес__ будет вычисляться исходя из формулы FE80::__[номер_роутера]__.

<details>
 <summary>Настройки iBGP между R14-R15</summary>

``` bash
#################
# Настройки R14 #
#################

conf t

int Lo999
 ipv6 address FE80::14 link-local

router bgp 1001
 bgp router-id 0.0.1.14
 neighbor 10.1.12.15 remote-as 1001
 neighbor 10.1.12.15 update-source Lo 999
 neighbor 10.1.12.15 next-hop-self
 neighbor 2001:FFCC:1000:1415::15 remote-as 1001
 neighbor 2001:FFCC:1000:1415::15 update-source LL
 neighbor 2001:FFCC:1000:1415::15 next-hop-self

address-family ipv4
 neighbor 10.1.12.15 activate
 neighbor 10.1.12.15 update-source Lo 999 activate
 neighbor 10.1.12.15 next-hop-self activate
 no neighbor 2001:FFCC:1000:1415::15 remote-as 1001 activate
 no neighbor 2001:FFCC:1000:1415::15 update-source LL activate
 no neighbor 2001:FFCC:1000:1415::15 next-hop-self activate

address-family ipv6
 neighbor 2001:FFCC:1000:1415::15 remote-as 1001 activate
 neighbor 2001:FFCC:1000:1415::15 update-source LL activate
 neighbor 2001:FFCC:1000:1415::15 next-hop-self activate

ip route 2.2.2.0 255.255.255.224 10.1.12.15 

#################
# Настройки R15 #
#################

conf t
int Lo999
 ipv6 address FE80::15 link-local

router bgp 1001
 bgp router-id 0.0.1.15
 neighbor 10.1.12.14 remote-as 1001
 neighbor 10.1.12.14 update-source Lo 999
 neighbor 10.1.12.14 next-hop-self
 neighbor 2001:FFCC:1000:1415::14 remote-as 1001
 neighbor 2001:FFCC:1000:1415::14 update-source LL
 neighbor 2001:FFCC:1000:1415::14 next-hop-self

address-family ipv4
 neighbor 10.1.12.14 remote-as 1001 activate
 neighbor 10.1.12.14 update-source Lo 999 activate
 neighbor 10.1.12.14 next-hop-self activate
 no neighbor 2001:FFCC:1000:1415::14 remote-as 1001 activate
 no neighbor 2001:FFCC:1000:1415::14 update-source LL activate
 no neighbor 2001:FFCC:1000:1415::14 next-hop-self activate

address-family ipv6
 neighbor 2001:FFCC:1000:1415::14 remote-as 1001 activate
 neighbor 2001:FFCC:1000:1415::14 update-source LL activate
 neighbor 2001:FFCC:1000:1415::14 next-hop-self activate

ip route 7.7.7.0 255.255.255.224 10.1.12.14

```
</details>

### Балансировка

Балансировка исходящего трафика, для большинства провайдеров, не столь критична, поэтому темой статьи является управление именно входящим трафиком. Нам нужно разложить наш трафик по двум каналам: