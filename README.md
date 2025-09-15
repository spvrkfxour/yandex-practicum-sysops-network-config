## Предусловие

<img width="612" height="362" alt="image" src="https://github.com/user-attachments/assets/998e1df6-b4e7-4c60-9fa8-801e2ec42189" />

В офисе работают две организации, обе арендуют часть офисного помещения. У первой на компьютерах прописаны адреса из сети 172.16.10.0/24: 172.16.10.10-12, а у второй — 172.16.20.0/24: 172.16.20.10-11. Менять адресацию на устройствах никто не хочет

Оборудование у организаций общее, предоставленное арендодателем. Задача системного администратора — сделать так, чтобы компьютеры не видели друг друга

Сисадмин начал настраивать сеть, но не успел довести работу до конца — его перебросили на другой объект

<img width="628" height="431" alt="image" src="https://github.com/user-attachments/assets/1e4c70c1-4bd4-473c-983f-2a337df40214" />

Все компьютеры подключены в один коммутатор. Какой компьютер в какой порт — неизвестно. Порт коммутатора GigabitEthernet0/1 подключён к порту роутера FastEthernet1\0

На роутере выполнены следующие команды:

```console
Router>enable
Router#
Router#configure tеrminаl
Router(config)#int FаstEthеrnet 1/0.100
Router(config-subif)#еncарsulatiоn dot1Q 100
Router(config-subif)#iр addrеss 172.16.10.1 255.255.255.0
Router(config-subif)#еx
Router(config)#int FаstEthеrnеt 1/0.200
Router(config-subif)#еncарsulation dot1Q 200
Router(config-subif)#iр аdd 172.16.20.1 255.255.255.0
Router(config-subif)#ex
Router(config)#  аccess-list 101 dеny ip 172.16.10.0 0.0.0.255 172.16.20.0 0.0.0.255
Router(config)#аccess-list 101 реrmit iр any any
Router(config)#  aссess-list 102 deny ip 172.16.20.0 0.0.0.255 172.16.10.0 0.0.0.255
Router(config)#aссess-list 102 permit ip any any
Router(config)#intеrface FаstEthеrnet 1/0.100
Router(config-subif)#  ip aссеss-group 101 in
Router(config-subif)#  ip aссеss-group 102 out
Router(config-subif)#ex
Router(config)#intеrface FаstEthernеt 1/0.200
Router(config-subif)#ip accеss-grоup 101 out
Router(config-subif)#ip access-grоup 102 in
Router(config-subif)#ex
Router(config)#ex
Router#wr
Building configuration...
[OK]
```

На интерфейсе маршрутизатора, подключённого к коммутатору, настроены два подинтерфейса для VLAN каждой подсети: VLAN 100 для 172.16.10.0/24, VLAN 200 для 172.16.20.0/24. Также на подинтерфейсы назначены адреса — 172.16.10.1 и 172.16.20.1, а ещё ваш предшественник настроил запрещающие правила между подсетями

## Задание №1

Донастройте коммутатор так, чтобы компьютеры организаций увидели шлюз своей подсети: 172.16.10.1 или 172.16.20.1

## Решение задания №1

```console
Switch>en
Switch#conf t
Switch(config)#hostname Switch0
Switch0(config)#line con 0
Switch0(config-line)#password 1234
Switch0(config-line)#login
Switch0(config-line)#exit
Switch0(config)#enable secret 1234
Switch0(config)#exit
Switch0#wr
Building configuration...
[OK]
```

Добавим интерфейсы в VLAN 100

```console
Switch0>en
Switch0#show vlan 
Switch0#conf t
Switch0(config)#interface gigabitEthernet 0/2
Switch0(config-if)#switchport mode access 
Switch0(config-if)#switchport access vlan 100
Switch0(config-if)#no shutdown
Switch0(config-if)#end
Switch0#wr
```

```console
Switch0>en
Switch0#show vlan 
Switch0#conf t
Switch0(config)#interface fastEthernet 0/6
Switch0(config-if)#switchport mode access 
Switch0(config-if)#switchport access vlan 100
Switch0(config-if)#no shutdown
Switch0(config-if)#end
Switch0#wr
```

```console
Switch0>en
Switch0#show vlan 
Switch0#conf t
Switch0(config)#interface fastEthernet 0/14
Switch0(config-if)#switchport mode access 
Switch0(config-if)#switchport access vlan 100
Switch0(config-if)#no shutdown
Switch0(config-if)#end
Switch0#wr
```

Добавим интерфейсы в VLAN 200

```console
Switch0>en
Switch0#show vlan 
Switch0#conf t
Switch0(config)#interface fastEthernet 0/9
Switch0(config-if)#switchport mode access 
Switch0(config-if)#switchport access vlan 200
Switch0(config-if)#no shutdown
Switch0(config-if)#end
Switch0#wr
```

```console
Switch0>en
Switch0#show vlan 
Switch0#conf t
Switch0(config)#interface fastEthernet 0/12
Switch0(config-if)#switchport mode access 
Switch0(config-if)#switchport access vlan 200
Switch0(config-if)#no shutdown
Switch0(config-if)#end
Switch0#wr
```

Настраиваем trunk порт на маршрутизаторе

```console
Switch0>en
Switch0#conf t
Switch0(config)#interface gigabitEthernet 0/1
Switch0(config-if)#switchport mode trunk 
Switch0(config-if)#switchport trunk allowed vlan 100,200
Switch0(config-if)#no shutdown
Switch0(config-if)#description to-router
Switch0(config-if)#end
Switch0#wr
```

Проверяем пинг с VLAN 100 и VLAN 200 до шлюзов

org1pc1 (VLAN 100)

<img width="455" height="216" alt="image" src="https://github.com/user-attachments/assets/aea2b12f-2a04-4ba4-a370-0ddf3e44bf90" />
<br>
<img width="465" height="185" alt="image" src="https://github.com/user-attachments/assets/02f41a25-1494-4350-9795-0b46ec57db8d" />
<br><br>

org2pc1 (VLAN 200)

<img width="469" height="182" alt="image" src="https://github.com/user-attachments/assets/1ed3f88b-12cc-44c0-ad2c-733a468fad8f" />
<br>
<img width="449" height="213" alt="image" src="https://github.com/user-attachments/assets/e5d57e50-fa4a-40a5-b0a4-0688e7fa158b" />
<br><br>
<img width="566" height="584" alt="image" src="https://github.com/user-attachments/assets/295c0199-212b-4521-8a77-7e9befe045b5" />

## Задание №2

Наши компании решили разместить у того же арендодателя свои серверы. Инфраструктура состоит из маршрутизатора, коммутатора и двух серверов

На коммутаторе настроены два VLAN — 10 (для первой организации) и 20 (для второй), есть trunk-порт в маршрутизатор

На маршрутизаторе настроены подинтерфейсы на каждый VLAN:
* VLAN 10 для первой организации с подсетью 192.168.10.0/24, адресом на маршрутизаторе 192.168.10.1 и адресом сервера 192.168.10.10
* VLAN 20 для второй с подсетью 192.168.20.0/24, адресом на маршрутизаторе 192.168.20.1 и адресом сервера 192.168.20.10

Маршрутизаторы R0 и R1 соединены через ненастроенные интерфейсы FastEthernet0/0 на двух маршрутизаторах

VLAN компаний видят друг друга. Ваша задача — ограничить доступ между сетями 192.168.10.0/24 и 192.168.20.0/24, а также настроить связь между двумя площадками. Каждая организация должна видеть только свой сервер

## Решение задания №2

Настроим статическую маршрутизацию на роутерах R0 и R1

```console
Router>en
Router#conf t
Router(config)#hostname Router0
Router0(config)#line con 0
Router0(config-line)#password 1234
Router0(config-line)#login
Router0(config-line)#exit
Router0(config)#enable secret 1234
Router0(config)#exit
Router0#wr
Building configuration...
[OK]
```

```console
Router0>en
Router0#conf t
Router0(config)#interface fastEthernet 0/0
Router0(config-if)#ip address 10.0.0.1 255.255.255.252
Router0(config-if)#no shutdown
Router0(config-if)#exit
Router0(config)#ip route 192.168.10.0 255.255.255.0 10.0.0.2
Router0(config)#ip route 192.168.20.0 255.255.255.0 10.0.0.2
Router0(config)#exit
Router0#wr
```

```console
Router>en
Router#conf t
Router(config)#hostname Router1
Router1(config)#line con 0
Router1(config-line)#password 1234
Router1(config-line)#login
Router1(config-line)#exit
Router1(config)#enable secret 1234
Router1(config)#exit
Router1#wr
Building configuration...
[OK]
```

```console
Router1>en
Router1#conf t
Enter configuration commands, one per line.  End with CNTL/Z.
Router1(config)#interface fastEthernet 0/0
Router1(config-if)#ip address 10.0.0.2 255.255.255.252
Router1(config-if)#no shutdown
Router1(config-if)#exit
Router1(config)#ip route 172.16.10.0 255.255.255.0 10.0.0.1
Router1(config)#ip route 172.16.20.0 255.255.255.0 10.0.0.1
Router1(config)#exit
Router1#wr
```

Проверим маршрутизацию пинганув с узла (172.16.10.10) из первого задания в роутер и любой узел во втором задании

<img width="454" height="213" alt="image" src="https://github.com/user-attachments/assets/e3cf1154-f2c1-4666-8cdf-d7e15fde0957" />
<br>
<img width="453" height="214" alt="image" src="https://github.com/user-attachments/assets/7e780dc6-36f2-42b7-b605-f8e52ac23e8e" />
<br><br>

И также с узла из второго задания (192.168.20.10) в роутер и любой узел из первого задания

<img width="450" height="212" alt="image" src="https://github.com/user-attachments/assets/0427d892-fddd-4ed8-96a5-78e2fdccdf7a" />
<br>
<img width="450" height="213" alt="image" src="https://github.com/user-attachments/assets/965c4b27-c4c9-47c6-b15a-daf3d42e62d2" />
<br><br>

Ограничим доступ между сетями 192.168.10.0/24 (VLAN 10) и 192.168.20.0/24 (VLAN 20)

Используем одно правило для обоих сабинтерфейсов, так как оно выполняет один функционал, хотя для части трафика оно никогда не срабатывает

```console
Router1>en
Router1#conf t
Router1(config)#access-list 110 deny ip 192.168.10.0 0.0.0.255 192.168.20.0 0.0.0.255
Router1(config)#access-list 110 deny ip 192.168.20.0 0.0.0.255 192.168.10.0 0.0.0.255
Router1(config)#access-list 110 permit ip any any
Router1(config)#exit
Router1#wr
Router1#conf t
Router1(config)#interface fastEthernet 1/0.10
Router1(config-subif)#ip access-group 110 in
Router1(config-subif)#exit
Router1(config)#interface fastEthernet 1/0.20
Router1(config-subif)#ip access-group 110 in
Router1(config-subif)#exit
Router1(config)#exit
Router1#wr
```

Проверим правила

<img width="412" height="159" alt="image" src="https://github.com/user-attachments/assets/470d561b-60a4-40c8-8f64-77348d7aa562" />
<br>
<img width="410" height="155" alt="image" src="https://github.com/user-attachments/assets/609c060b-8886-4c26-ab14-a5457765d0de" />
<br><br>

Ограничим доступ между сетями разных организаций (VLAN 100 – VLAN 10, VLAN 200 – VLAN 20)

```console
Router0>en
Router0#conf t
Router0(config)#access-list 120 deny ip 172.16.10.0 0.0.0.255 192.168.20.0 0.0.0.255
Router0(config)#access-list 120 deny ip 172.16.20.0 0.0.0.255 192.168.10.0 0.0.0.255
Router0(config)#access-list 120 permit ip any any
Router0(config)#exit
Router0#wr
Router0#conf t
Router0(config)#interface fastEthernet 1/0.100
Router0(config-if)#ip access-group 120 in
Router0(config-if)#exit
Router0(config)#interface fastEthernet 1/0.200
Router0(config-subif)#ip access-group 120 in
Router0(config-subif)#exit
Router0(config)#exit
Router0#wr
```

```console
Router1>en
Router1#conf t
Router1(config)#access-list 120 deny ip 192.168.10.0 0.0.0.255 172.16.20.0 0.0.0.255
Router1(config)#access-list 120 deny ip 192.168.20.0 0.0.0.255 172.16.10.0 0.0.0.255
Router1(config)#access-list 120 permit ip any any
Router1(config)#exit
Router1#wr
Router1#conf t
Router1(config)#interface fastEthernet 1/0.10
Router1(config-subif)#ip access-group 120 in
Router1(config-subif)#exit
Router1(config)#interface fastEthernet 1/0.20
Router1(config-subif)#ip access-group 120 in
Router1(config-subif)#exit
Router1(config)#exit
Router1#wr
```

Проверим правила c VLAN 100 и VLAN 10

org1pc2 (VLAN 100)

<img width="397" height="188" alt="image" src="https://github.com/user-attachments/assets/af9ad5e2-a61a-4114-b18e-820d17cec137" />
<br>
<img width="409" height="159" alt="image" src="https://github.com/user-attachments/assets/f31fd875-4629-4921-a8b5-367920a4400e" />
<br><br>

Server1 (VLAN 10)

<img width="407" height="159" alt="image" src="https://github.com/user-attachments/assets/30cd0d68-76f0-4cbb-9abf-5c7eab3847cb" />
<br>
<img width="391" height="183" alt="image" src="https://github.com/user-attachments/assets/58c8ee26-9127-4271-a838-b005e739e591" />
<br><br>

Проверим правила c VLAN 200 и VLAN 20

org2pc2 (VLAN 200)

<img width="406" height="158" alt="image" src="https://github.com/user-attachments/assets/c851d23f-27f1-4afb-9311-ad4d95f11156" />
<br>
<img width="398" height="160" alt="image" src="https://github.com/user-attachments/assets/9d352a2d-a28e-4525-bc41-dec5e7320ba9" />
<br><br>

Server2 (VLAN 20)

<img width="395" height="153" alt="image" src="https://github.com/user-attachments/assets/b406d141-d737-4811-b1e8-e78c9e919a31" />
<br>
<img width="412" height="161" alt="image" src="https://github.com/user-attachments/assets/d8664ee1-b272-43ff-bc95-6a07e8443e15" />
<br><br>

<img width="518" height="338" alt="image" src="https://github.com/user-attachments/assets/fb69a502-dad3-4cf5-99e7-0ceea6b2e112" />

## Задание №3

Во внешней провайдерской сети есть три сервера с адресами 210.0.11.50-52 . Из неё — 210.0.11.0/24 — арендодателю выдано два адреса: 210.0.11.10 и 210.0.11.20. Адрес 210.0.11.10 настроен на интерфейсе FastEthernet0/1 маршрутизатора R1

Настройте с помощью NAT доступ к этим серверам с обеих площадок

## Решение задания №3

Создадим ACL который будет привязан к каждой организации. ACL Будет определять какой трафик будет попадать под NAT

```console
Router1>en
Router1#conf t
Router1(config)#ip access-list standard ORG-1
Router1(config-std-nacl)#permit 172.16.10.0 0.0.0.255
Router1(config-std-nacl)#permit 192.168.10.0 0.0.0.255
Router1(config-std-nacl)#exit
Router1(config)#ip access-list standard ORG-2
Router1(config-std-nacl)#permit 172.16.20.0 0.0.0.255
Router1(config-std-nacl)#permit 192.168.20.0 0.0.0.255
Router1(config-std-nacl)#exit
Router1(config)#exit
Router1#wr
```
Создадим пул адресов которые может использовать NAT

```console
Router1>en
Router1#conf t
Router1(config)#ip nat pool POOL-ORG-1 210.0.11.10 210.0.11.10 netmask 255.255.255.0
Router1(config)#ip nat pool POOL-ORG-2 210.0.11.20 210.0.11.20 netmask 255.255.255.0
Router1(config)#exit
Router1#wr
```

Применяем overload NAT к каждому хосту указанному в ACL ORG-1 и ORG-2

```console
Router1>en
Router1#conf t
Router1(config)#ip nat inside source list ORG-1 pool POOL-ORG-1 overload
Router1(config)#ip nat inside source list ORG-2 pool POOL-ORG-2 overload
Router1(config)#exit
Router1#wr
```

И указываем какие из интерфейсов внешние и внутренние для NAT

```console
Router1>en
Router1#conf t
Router1(config)#interface fastEthernet 0/0
Router1(config-if)#ip nat inside
Router1(config-if)#exit
Router1(config)#interface fastEthernet 1/0.10
Router1(config-if)#ip nat inside
Router1(config-if)#exit
Router1(config)#interface fastEthernet 1/0.20
Router1(config-if)#ip nat inside
Router1(config-if)#exit
Router1(config)#interface fastEthernet 0/1
Router1(config-if)#ip nat outside
Router1(config-if)#exit
Router1(config)#exit
Router1#wr
```

Добавляем статический маршрут для R0

```console
Router0>en
Router0#conf t
Router0(config)#ip route 210.0.11.0 255.255.255.0 10.0.0.2
Router0(config)#exit
Router0#wr
```

Проверяем пинг до серверов за NAT с разных хостов

org1pc3 (VLAN 100)

<img width="359" height="118" alt="image" src="https://github.com/user-attachments/assets/71995a9e-1e60-41fe-9865-25e2f667e144" />
<br><br>

org2pc2 (VLAN 200)

<img width="350" height="115" alt="image" src="https://github.com/user-attachments/assets/8c81ecd4-5e26-4580-8a03-ba99a6516658" />
<br><br>

server1 (VLAN 10)

<img width="346" height="117" alt="image" src="https://github.com/user-attachments/assets/f27b7cdd-a660-49cb-a749-da3026de3020" />
<br><br>

server2 (VLAN 20)

<img width="350" height="115" alt="image" src="https://github.com/user-attachments/assets/521a0f32-5bc2-4ae6-a906-f09f6058287e" />
<br><br>

<img width="343" height="331" alt="image" src="https://github.com/user-attachments/assets/8d7b0aa6-f7d0-400c-87ab-c327c1bb0044" />
<br><br>

pkt файл: https://disk.yandex.ru/d/qm_p-pLSnInWpw
