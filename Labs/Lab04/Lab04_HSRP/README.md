# Исследование работы протоколов избыточности шлюза HSRP, GLBP.
# Лабораторная работа №4. Настройка HSRP.

### Топология
![network](network.png)

### Таблица адресации

<table>
  <tr>
    <th>Устройство</th>
    <th>Интерфейс</th>
    <th>IP-адрес</th>
    <th>Маска подсет</th>
    <th>Шлюз по умолчанию</th>
  </tr>
  <tr>
    <td rowspan="2">R1</td>
    <td>G0/1</td>
    <td>192.168.1.11</td>
    <td>255.255.255.0</td>
    <td> - </td>
  </tr>
  <tr>
    <td>S0/0/0 (DCE)</td>
    <td>10.1.1.1</td>
    <td>255.255.255.252</td>
    <td> - </td>
  </tr>
  <tr>
    <td rowspan="3">R2</td>
    <td>S0/0/0</td>
    <td>10.1.1.2</td>
    <td>255.255.255.252</td>
    <td> - </td>
  </tr>
  <tr>
    <td>S0/0/1 (DCE)</td>
    <td>10.2.2.2</td>
    <td>255.255.255.252</td>
    <td> - </td>
  </tr>
  <tr>
    <td>Lo1</td>
    <td>209.165.200.225</td>
    <td>255.255.255.224</td>
    <td> - </td>
  </tr>
  <tr>
    <td rowspan="2">R3</td>
    <td>G0/1</td>
    <td>192.168.1.3</td>
    <td>255.255.255.0</td>
    <td> - </td>
  </tr>
  <tr>
    <td>S0/0/1</td>
    <td>10.2.2.1</td>
    <td>255.255.255.252</td>
    <td> - </td>
  </tr>
  <tr>
    <td>S1</td>
    <td>VLAN 1</td>
    <td>192.168.1.11</td>
    <td>255.255.255.0</td>
    <td>192.168.1.1</td>
  </tr>
  <tr>
    <td>S2</td>
    <td>VLAN 1</td>
    <td>192.168.1.13</td>
    <td>255.255.255.0</td>
    <td>192.168.1.3</td>
  </tr>
  <tr>
    <td>PC-A</td>
    <td>NIC</td>
    <td>192.168.1.31</td>
    <td>255.255.255.0</td>
    <td>192.168.1.1</td>
  </tr>
  <tr>
    <td>PC-C</td>
    <td>NIC</td>
    <td>192.168.1.33</td>
    <td>255.255.255.0</td>
    <td>192.168.1.3</td>
  </tr>
</table>


### Задание:
#### [Часть 1. Построение сети и проверка соединения](README.md#)
#### [Часть 2. Настройка обеспечения избыточности на первом хопе с помощью HSRP](README.md#)
#### [Конфигурационные файлы ](README.md#конфигурационные-файлы-здесь)

### Решение:

*Шаг 1. Создать сеть согласно топологии.*

Подключить устройства, как показано в топологии, и подсоединить необходимые кабели.

![network_2_eve](network_2_eve.png)

### Таблица адресации

<table>
  <tr>
    <th>Устройство</th>
    <th>Интерфейс</th>
    <th>IP-адрес</th>
    <th>Маска подсет</th>
    <th>Шлюз по умолчанию</th>
  </tr>
  <tr>
    <td rowspan="2">R1</td>
    <td>e0/0</td>
    <td>192.168.1.11</td>
    <td>255.255.255.0</td>
    <td> - </td>
  </tr>
  <tr>
    <td>S1/0 (DCE)</td>
    <td>10.1.1.1</td>
    <td>255.255.255.252</td>
    <td> - </td>
  </tr>
  <tr>
    <td rowspan="3">R2</td>
    <td>S1/0</td>
    <td>10.1.1.2</td>
    <td>255.255.255.252</td>
    <td> - </td>
  </tr>
  <tr>
    <td>S1/1 (DCE)</td>
    <td>10.2.2.2</td>
    <td>255.255.255.252</td>
    <td> - </td>
  </tr>
  <tr>
    <td>Lo1</td>
    <td>209.165.200.225</td>
    <td>255.255.255.224</td>
    <td> - </td>
  </tr>
  <tr>
    <td rowspan="2">R3</td>
    <td>e0/0</td>
    <td>192.168.1.3</td>
    <td>255.255.255.0</td>
    <td> - </td>
  </tr>
  <tr>
    <td>S1/1</td>
    <td>10.2.2.1</td>
    <td>255.255.255.252</td>
    <td> - </td>
  </tr>
  <tr>
    <td>S1</td>
    <td>VLAN 1</td>
    <td>192.168.1.11</td>
    <td>255.255.255.0</td>
    <td>192.168.1.1</td>
  </tr>
  <tr>
    <td>S2</td>
    <td>VLAN 1</td>
    <td>192.168.1.13</td>
    <td>255.255.255.0</td>
    <td>192.168.1.3</td>
  </tr>
  <tr>
    <td>PC-A</td>
    <td>NIC</td>
    <td>192.168.1.31</td>
    <td>255.255.255.0</td>
    <td>192.168.1.1</td>
  </tr>
  <tr>
    <td>PC-C</td>
    <td>NIC</td>
    <td>192.168.1.33</td>
    <td>255.255.255.0</td>
    <td>192.168.1.3</td>
  </tr>
</table>

*Шаг 2. Настроить узлы ПК.*

Настройки PC-A
``` bash
VPCS> ip 192.168.1.31/24 192.168.1.1
Checking for duplicate address...
PC1 : 192.168.1.31 255.255.255.0 gateway 192.168.1.1
```

Настройки PC-C
``` bash
VPCS> ip 192.168.1.33/24 192.168.1.3
Checking for duplicate address...
PC1 : 192.168.1.33 255.255.255.0 gateway 192.168.1.3
```
*Шаг 3. Выполните инициализацию и перезагрузку маршрутизатора и коммутаторов.*

*Шаг 4. Произведите базовую настройку маршрутизаторов.*

- Отключите поиск DNS.
- Присвойте имена устройствам в соответствии с топологией.
- Настройте IP-адреса для маршрутизаторов, указанных в таблице адресации.
- Установите тактовую частоту на 128000 для всех последовательных интерфейсов маршрутизатора DCE.
- Назначьте class в качестве зашифрованного пароля доступа к привилегированному режиму.
- Назначьте cisco в качестве пароля консоли и VTY и включите запрос пароля при подключении.
- Настройте logging synchronous, чтобы сообщения от консоли не могли прерывать ввод команд.
- Скопируйте текущую конфигурацию в файл загрузочной конфигурации.

Пример для S1.
``` bash
Для S1

Switch>ena
Switch#conf t
Switch(config)#hostname S1
S1(config)#no ip domain-lookup
S1(config)#service password-encryption
S1(config)#enable secret class
S1(config)#ip default-gateway 192.168.1.1
S1(config)#line console 0
S1(config-line)#password cisco
S1(config-line)#exec-timeout 5 0
S1(config-line)#logging synchronous
S1(config-line)#login
S1(config-line)#exit
S1(config)#line vty 0 4
S1(config-line)#password cisco
S1(config-line)#exec-timeout 5 0
S1(config-line)#logging synchronous
S1(config-line)#login
S1(config-line)#exit
S1(config)#exit
S1#wr
```
Пример для R1

``` bash
R3#conf t
R3(config)#hostname R3
R3(config)#no ip domain-lookup
R3(config)#int s1/1
R3(config-if)#ip address 10.2.2.1 255.255.255.252
R3(config-if)#no shutdown
R3(config-if)#exit
R3(config)#int e0/0
R3(config-if)#ip address 192.168.1.3 255.255.255.0
R3(config-if)#no shutdown
R3(config-if)#service password-encryption 
R3(config)#enable secret class
R3(config)#line console 0
R3(config-line)#password cisco
R3(config-line)#exec-timeout 5 0
R3(config-line)#logging synchronous
R3(config-line)#login
R3(config-line)#exit
R3(config)#line vty 0 4
R3(config-line)#password cisco
R3(config-line)#exec-timeout 5 0
R3(config-line)#logging synchronous
R3(config-line)#login
R3(config-line)#exit
R3(config)#exit
R3#wr
```
*Шаг 6. Проверьте подключение между PC-A и PC-C.*
Отправьте ping-запрос с компьютера PC-A на компьютер PC-C. Удалось ли получить ответ?
> Команда пинг с PC-A до PC-C прошла успешно.

Если команды ping завершились неудачно и связь установить не удалось, исправьте ошибки в основных настройках устройства.

*Шаг 7. Настройте маршрутизацию.*
