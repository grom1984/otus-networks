# Настройка базового протокола OSPFv2 для нескольких областей
# Лабораторная работа №5. OSPF v2 Multiarea

### Задание:
#### Часть 1. [Создание сети и настройка основных параметров устройства](README.md#создание-сети-и-настройка-основных-параметров-устройства-1)



### Решение:

### Часть 1. Создание сети и настройка основных параметров устройства

### Топология
Создали стенд в eve-ng согласно топологии.
![network.png](network.png)

### Таблица адресации

<table>
  <tr>
    <th>Устройство</th>
    <th>Интерфейс</th>
    <th>IP-адрес</th>
    <th>Маска подсети</th>
  </tr>
  <tr>
    <td rowspan="4">R1</td>
    <td>Lo/0</td>
    <td>209.165.200.225</td>
    <td>255.255.255.252</td>
  </tr>
  <tr>
    <td>Lo1</td>
    <td>192.168.1.1</td>
    <td>255.255.255.0</td>
  </tr>
  <tr>
    <td>Lo2</td>
    <td>192.168.2.1</td>
    <td>255.255.255.0</td>
  </tr>
  <tr>
    <td>S1/0 (DCE)</td>
    <td>192.168.12.1</td>
    <td>255.255.255.252</td>
  </tr>
  <tr>
    <td rowspan="3">R2</td>
    <td>Lo6</td>
    <td>192.168.6.1</td>
    <td>255.255.255.0</td>
  </tr>
  <tr>
    <td>S1/0</td>
    <td>192.168.12.2</td>
    <td>255.255.255.252</td>
  </tr>
  <tr>
    <td>S1/1 (DCE)</td>
    <td>192.168.23.1</td>
    <td>255.255.255.252</td>
  </tr>
  <tr>
    <td rowspan="3">R3</td>
    <td>Lo4</td>
    <td>192.168.4.1</td>
    <td>255.255.255.0</td>
  </tr>
  <tr>
    <td>Lo5</td>
    <td>192.168.5.1</td>
    <td>255.255.255.0</td>
  </tr>
  <tr>
    <td>S1/1</td>
    <td>192.168.23.2</td>
    <td>255.255.255.252</td>
  </tr>
</table>

Произвели базовую настройку маршрутизаторов.

<details>
 <summary>Базовые настройки R1</summary>

``` bash
Router>
Router>ena
Router#conf t
Router(config)#hostname R1
R1(config)#no logging console
R1(config)#no ip domain-lookup
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
Building configuration...
[OK]
R1#
R1#conf t
Enter configuration commands, one per line.  End with CNTL/Z.
R1(config)#int Lo0
R1(config-if)#ip address 209.165.200.225 255.255.255.252
R1(config-if)#no shutdown
R1(config-if)#end
R1#
R1#conf t
Enter configuration commands, one per line.  End with CNTL/Z.
R1(config)#int Lo1
R1(config-if)#ip address 192.168.1.1 255.255.255.0
R1(config-if)#no shutdown
R1(config-if)#end
R1#
R1#conf t
Enter configuration commands, one per line.  End with CNTL/Z.
R1(config)#int Lo2
R1(config-if)#ip address 192.168.2.1 255.255.255.0
R1(config-if)#no shutdown
R1(config-if)#end
R1#
R1#conf t
Enter configuration commands, one per line.  End with CNTL/Z.
R1(config)#int s1/0
R1(config-if)#ip address 192.168.12.1 255.255.255.252
R1(config-if)#clock rate 128000
R1(config-if)#no shutdown
R1(config-if)#end
R1#wr
Building configuration...
[OK]
R1#

```
</details>

<details>
 <summary>Базовые настройки R2</summary>

``` bash
Router>
Router>ena
Router#conf t
Router(config)#hostname R2
R2(config)#no logging console
R2(config)#no ip domain-lookup
R2(config)#service password-encryption 
R2(config)#enable secret class
R2(config)#line console 0
R2(config-line)#password cisco
R2(config-line)#logging synchronous
R2(config-line)#login
R2(config-line)#exit
R2(config)#line vty 0 4
R2(config-line)#password cisco
R2(config-line)#logging synchronous
R2(config-line)#login
R2(config-line)#exit
R2(config)#exit
R2#wr
Building configuration...
[OK]
R2#
R2#conf t
Enter configuration commands, one per line.  End with CNTL/Z.
R2(config)#int Lo6
R2(config-if)#ip address 192.168.6.1 255.255.255.0
R2(config-if)#no shutdown
R2(config-if)#end
R2#conf t
Enter configuration commands, one per line.  End with CNTL/Z.
R2(config)#int S1/0
R2(config-if)#ip address 192.168.12.2 255.255.255.252
R2(config-if)#no shutdown
R2(config-if)#end
R2#conf t
Enter configuration commands, one per line.  End with CNTL/Z.
R2(config)#int s1/1
R2(config-if)#ip address 192.168.23.1 255.255.255.252
R2(config-if)#clock rate 128000
R2(config-if)#no shutdown
R2(config-if)#end
R2#wr
Building configuration...
[OK]
R2#

```
</details>

<details>
 <summary>Базовые настройки R3</summary>

``` bash
Router>
Router>ena
Router#conf t
Enter configuration commands, one per line.  End with CNTL/Z.
Router(config)#hostname R3
R3(config)#no logging console
R3(config)#no ip domain-lookup
R3(config)#service password-encryption 
R3(config)#enable secret class
R3(config)#line console 0
R3(config-line)#password cisco
R3(config-line)#logging synchronous
R3(config-line)#login
R3(config-line)#exit
R3(config)#line vty 0 4
R3(config-line)#password cisco
R3(config-line)#logging synchronous
R3(config-line)#login
R3(config-line)#exit
R3(config)#exit
R3#wr
Building configuration...
[OK]
R3#
R3#conf t
Enter configuration commands, one per line.  End with CNTL/Z.
R3(config)#int Lo4
R3(config-if)#ip address 192.168.4.1 255.255.255.0
R3(config-if)#no shutdown
R3(config-if)#end
R3#
R3#conf t
Enter configuration commands, one per line.  End with CNTL/Z.
R3(config)#int Lo5
R3(config-if)#ip address 192.168.5.1 255.255.255.0
R3(config-if)#no shutdown
R3(config-if)#end
R3#
R3#conf t
Enter configuration commands, one per line.  End with CNTL/Z.
R3(config)#int s1/1
R3(config-if)#ip address 192.168.23.2 255.255.255.252
R3(config-if)#no shutdown
R3(config-if)#end
R3#wr
Building configuration...
[OK]
R3#

```
</details>

Проверка наличия подключения на уровне 3.

<details>
 <summary>R2</summary>

``` bash

R2>ping 192.168.12.1
Success rate is 100 percent (5/5), round-trip min/avg/max = 5/8/9 ms

R2>ping 192.168.23.2
Success rate is 100 percent (5/5), round-trip min/avg/max = 8/8/9 ms
R2>

```
</details>

### Часть 2. Настройка сети OSPFv2 для нескольких областей