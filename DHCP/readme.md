# Lab - Configure Router-on-a-Stick Inter-VLAN Routing

## Задание

1. Организовать сеть и провести базовую настройку устройств
2. Создать ВЛАНы и назначить на порту коммутаторов
3. Настроить 802.1Q транк между коммутаторами
4. Настроить маршрутизацию между ВЛАН на маршрутизаторе
5. Убедиться что все работает

### Топология

![топология](https://user-images.githubusercontent.com/5254857/125946372-37d750d5-8f33-4f92-bab1-b5caea38c13a.png)

### Таблица адресации
| Device | Interface      | IP address   | Subnet Mask     | Default Gateway |
| ------ | -------------- | ------------ | -------------   | --------------- |
| R1     | G0/0/0         | 10.0.0.1     | 255.255.255.252 | N/A             |
|        | G0/0/1         | N/A          | N/A             | N/A             |
|        | G0/0/1.100     | 192.168.1.1  | 255.255.255.192 | N/A             |
|        | G0/0/1.200     | 192.168.1.65 | 255.255.255.224 | N/A             |
|        | G0/0/1.1000    | N/A          | N/A             | N/A             | 
| R2     | G0/0/0         | 10.0.0.2     | 255.255.255.252 | N/A             |
|        | G0/0/1         | 192.168.1.97 | 255.255.255.240 |                 |
| S1     | Vlan 200       | 192.168.1.66 | 255.255.255.224 |                 |
| S2     | Vlan 1         | 192.168.1.98 | 255.255.255.240 |                 |
| PC-A   | NIC            | DHCP         | DHCP            | DHCP            |
| PC-B   | NIC            | DHCP         | DHCP            | DHCP            |

### Таблица Vlan
| Vlan   | Name        | Interface Assigned            |
| ------ | ----------  | ----------------------------- |
| 1      | N/A         | S2: fa0/18                    |
| 100    | Clients     | S1: fa0/6                     |
| 200    | Management  | S1: vlan 200                  |
| 999    | Parking Lot | S1: f0/1-4, f0/7-24, g0/1-2   |
| 1000   | Native      | N/A                           | 





##  Решение:

### В сети 192.168.1.0/24 на 3 подсети:

1.	Подсеть A, поддерживающая 58 хостов (клиентский VLAN на R1): 192.168.1.0/26
2.	Подсеть B, поддерживающая 28 хостов (VLAN управления на R1): 192.168.1.64/27
3.	Подсеть С, поддерживающая 12 хостов (клиентская подсеть на R2): 192.168.1.96/28

### Подключение устройств согласно топологии и базовая настройка.

R1:

```
Router>en
Router#conf t
Router(config)#hostname R1
R1(config)#no ip domain-lookup 
R1(config)#enable secret class
R1(config)#line console 0
R1(config-line)#password cisco
R1(config-line)#login
R1(config-line)#exit
R1(config)#line vty 0 4
R1(config-line)#password cisco
R1(config-line)#login
R1(config-line)#exit
R1(config)#service password-encryption 
R1(config)#banner motd 'Authorized Accessonly'
R1(config)#end
R1#clock set 11:10:00 7 March 2021
R1#copy running-config startup-config 
Destination filename [startup-config]? 
Building configuration...
[OK]

```


S1:
```

Switch#enable
Switch#configure terminal 
Switch(config)#hostname S1
S1(config)#no ip domain-lookup 
S1(config)#enable secret class
S1(config)#line console 0
S1(config-line)#password cisco
S1(config-line)#login
S1(config-line)#exit
S1(config)#line vty 0 15
S1(config-line)#password cisco
S1(config-line)#login
S1(config-line)#exit
S1(config)#service password-encryption 
S1(config)#banner motd 'Authorized Accessonly'
S1#clock set 11:00:00 7 March 2021
S1#copy running-config startup-config 

```


### Настройка межвлановой маршрутизации на R1.

R1:

```
R1(config)#int GigabitEthernet0/0/1.100
R1(config-subif)#des client
R1(config-subif)#ip address 192.168.1.1 255.255.255.192
R1(config-subif)#encapsulation dot1Q 100
R1(config-subif)#ex
R1(config)#int gig 0/0/1.200
R1(config-subif)#des managment
R1(config-subif)#ip add 192.168.1.65 255.255.255.224
R1(config-subif)#encapsulation dot1Q 100
R1(config-subif)#ex
R1(config)#int gig  0/0/1.1000
R1(config-subif)#des native
R1(config-subif)#encapsulation dot1Q 1000 native 
R1(config-subif)#ex
```
### Конфигурация  G0/0/1 на R2 и G0/0/0 и настройка статической маршрутизации.


R1:
```
R1(config)#int gig0/0/0
R1(config-if)#ip address 10.0.0.1 255.255.255.252
R1(config-if)#no shut

R1(config)#ip route 0.0.0.0 0.0.0.0 10.0.0.2
```
R2:
```
R2(config)#int gig0/0/1
R2(config-if)#ip ad
R2(config-if)#ip address 192.168.1.97 255.255.255.240
R2(config-if)#no shut
R2(config-if)#ex
R2(config)#interface gig0/0/0
R2(config-if)#ip address 10.0.0.2 255.255.255.252
R2(config-if)#no shut
R2(config-if)#ex

R2(config)#ip route 0.0.0.0 0.0.0.0 10.0.0.1
```
Проверка доступности интерфейса gig0/0/1 на R2 c R1:
```
R1#ping 192.168.1.97

Type escape sequence to abort.
Sending 5, 100-byte ICMP Echos to 192.168.1.97, timeout is 2 seconds:
!!!!!
Success rate is 100 percent (5/5), round-trip min/avg/max = 0/0/0 ms
```
### Настройка базовой конфигурации на S1 и S2. 

S1 (S2 аналогично):
```
Switch(config)#hostname S1
S1(config)#no ip domain lookup
S1(config)#enable secret class
S1(config)#line console 0
S1(config-line)#password cisco
S1(config-line)#login
S1(config-line)#line vty 0 4
S1(config-line)#password cisco
S1(config-line)#login
S1(config-line)#exit
S1(config)#service password-encryption
S1(config)#banner motd $ only for autorized users $
S1(config)#exit
S1#copy running-config startup-config
S1#clock set  18:19:00 04 April 2021
```
### Создание и активация вланов  на S1 и S2.
S1:
```
S1(config)#int vlan 200
S1(config-if)#ip add
S1(config-if)#ip address 192.168.1.66 255.255.255.224
S1(config-if)#no shut
S1(config-if)#ex
```
S2:
```
S2(config)#int vlan 1
S2(config-if)#ip add
S2(config-if)#ip address 192.168.1.98 255.255.255.240
S2(config-if)#no shut
S2(config-if)#ex
```
