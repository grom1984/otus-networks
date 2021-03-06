# Настройка базового протокола OSPFv3 для одной области
# Лабораторная работа №4. OSPF v3

### Задание:
#### Часть 1. [Создание сети и настройка основных параметров устройства](README.md#часть-1-создание-сети-инастройка-основных-параметров-устройства-1)

#### Часть 2. [Настройка и проверка маршрутизации OSPFv3](README.md#часть-2-настройка-ипроверка-маршрутизации-ospfv3-1)

#### Часть 3. [Настройка пассивных интерфейсов OSPFv3](README.md#часть-3-настройка-пассивных-интерфейсов-ospfv3-1)


### Решение:

### Часть 1. Создание сети и настройка основных параметров устройства

Создали стенд в eve-ng согласно топологии.
### Топология
![network](network.png)

### Таблица адресации

<table>
  <tr>
    <th>Устройство</th>
    <th>Интерфейс</th>
    <th colspan="2">IP-адрес IPv6</th>
    <th>Шлюз по умолчанию</th>
  </tr>
  <tr>
    <td rowspan="3">R1</td>
    <td>E0/0</td>
    <td>2001:DB8:ACAD:A::1/64</td>
    <td>FE80::1</td>
    <td rowspan="9"></td>
  </tr>
  <tr>
    <td>S1/0 (DCE)</td>
    <td>2001:DB8:ACAD:12::1/64</td>
    <td>FE80::1</td>
  </tr>
  <tr>
    <td>S1/1</td>
    <td>2001:DB8:ACAD:13::1/64</td>
    <td>FE80::1</td>
  </tr>
  <tr>
    <td rowspan="3">R2</td>
    <td>E0/0</td>
    <td>2001:DB8:ACAD:B::2/64</td>
    <td>FE80::2</td>
  </tr>
  <tr>
    <td>S1/0</td>
    <td>2001:DB8:ACAD:12::2/64</td>
    <td>FE80::2</td>
  </tr>
  <tr>
    <td>S1/1 (DCE)</td>
    <td>2001:DB8:ACAD:23::2/64</td>
    <td>FE80::2</td>
  </tr>
  <tr>
    <td rowspan="3">R3</td>
    <td>E0/0</td>
    <td>2001:DB8:ACAD:C::3/64</td>
    <td>FE80::3</td>
  </tr>
  <tr>
    <td>S1/0 (DCE)</td>
    <td>2001:DB8:ACAD:13::3/64</td>
    <td>FE80::3</td>
  </tr>
  <tr>
    <td>S1/1</td>
    <td>2001:DB8:ACAD:23::3/64</td>
    <td>FE80::3</td>
  </tr>
  <tr>
    <td>PC-A</td>
    <td>NIC</td>
    <td>2001:DB8:ACAD:A::A/64</td>
    <td></td>
    <td>FE80::1</td>
  </tr>
  <tr>
    <td>PC-B</td>
    <td>NIC</td>
    <td>2001:DB8:ACAD:B::B/64</td>
    <td></td>
    <td>FE80::2</td>
  </tr>
  <tr>
    <td>PC-C</td>
    <td>NIC</td>
    <td>2001:DB8:ACAD:C::C/64</td>
    <td></td>
    <td>FE80::3</td>
  </tr>
</table>

<details>
 <summary>Произвели базовую настройку маршрутизаторов</summary>

``` bash
```
Пример настройки S1.
``` bash
Router#conf t
Enter configuration commands, one per line.  End with CNTL/Z.
Router(config)#hostname R1
R1(config)#no logging console
R1(config)#no ip domain-lookup
R1(config)#
R1(config)#int e0/0
R1(config-if)#ipv6 address 2001:DB8:ACAD:A::1/64
R1(config-if)#ipv6 address FE80::1 link-local
R1(config-if)#ipv6 enable
R1(config-if)#no shutdown
R1(config-if)#exit
R1(config)#
R1(config)#int s1/0
R1(config-if)#ipv6 address 2001:DB8:ACAD:12::1/64
R1(config-if)#ipv6 address FE80::1 link-local
R1(config-if)#ipv6 enable
R1(config-if)#clock rate 128000
R1(config-if)#no shutdown
R1(config-if)#exit
R1(config)#
R1(config)#int s1/1
R1(config-if)#ipv6 address 2001:DB8:ACAD:13::1/64
R1(config-if)#ipv6 address FE80::1 link-local
R1(config-if)#ipv6 enable
R1(config-if)#no shutdown
R1(config-if)#exit
R1(config)#
R1(config)#service password-encryption
R1(config)#enable secret class
R1(config)#line console 0
R1(config-line)#password cisco
R1(config-line)#logging synchronous
R1(config-line)#login
R1(config-line)#exit
R1(config)#line vty 0 4
R1(config-line)#password cisco
R1(config-line)#logging synchronous
R1(config-line)#login
R1(config-line)#exit
R1(config)#exit
R1#wr
```
</details>
<details>
 <summary>Произвели настройку ПК</summary>

``` bash
set pcname PC-B
ip 2001:DB8:ACAD:A::A/64 FE80::1
```
``` bash
set pcname PC-B
ip 2001:DB8:ACAD:B::B/64 FE80::2
```
``` bash
set pcname PC-C
ip 2001:DB8:ACAD:C::C/64 FE80::3
```
</details>

