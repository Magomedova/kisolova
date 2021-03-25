# Lab - Configure Router-on-a-Stick Inter-VLAN Routing

## Задание

1. Организовать сеть и провести базовую настройку устройств
2. Создать ВЛАНы и назначить на порту коммутаторов
3. Настроить 802.1Q транк между коммутаторами
4. Настроить маршрутизацию между ВЛАН на маршрутизаторе
5. Убедиться что все работает

### Топология

![топология](https://user-images.githubusercontent.com/5254857/112503036-a73a5f80-8d9b-11eb-8462-6a0ab1ba5ec5.png)




### Таблица адресации
| Device | Interface | IP address   | Subnet Mask   | Default Gateway |
| ------ | --------- | ------------ | ------------- | --------------- |
| R1     | G0/0/1.3  | 192.168.3.1  | 255.255.255.0 | N/A             |
|        | G0/0/1.4  | 192.168.4.1  | 255.255.255.0 | N/A             |
|        | G0/0/1.8  |    N/A       | N/A           | N/A             |
| S1     | VLAN 3    | 192.168.3.11 | 255.255.255.0 | 192.168.3.1     |
| S2     | VLAN 3    | 192.168.3.12 | 255.255.255.0 | 192.168.3.1     | 
| PC0    | NIC       | 192.168.3.3  | 255.255.255.0 | 192.168.3.1     |
| PC1    | NIC       | 192.168.4.3  | 255.255.255.0 | 192.168.4.1     |

### Таблица Vlan
| Vlan   | Name       | Interface Assigned            |
| ------ | ---------- | ----------------------------- |
| 3      | Management | S1: VLAN 3                    |
|        |            | S2: VLAN 3                    |
|        |            | S1: F0/6                      |
| 4      | Operations | S2: F0/18                     |
| 7      | ParkingLot | S1: F0/2-4, F0/7-24, G0/1-2   | 
|        |            | S2: F0/2-17, F0/19-24, G0/1-2 |
| 8      | Native     | N/A                           |


##  Решение:
#### Организовать сеть и провести базовую настройку устройств

* Подключение сети согласно топологии.

![схема](https://user-images.githubusercontent.com/5254857/112504594-fe8cff80-8d9c-11eb-9c66-de41979bb0c7.png)

* Настройка основных параметров маршрутизатора.

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
R1#clock set 19:14:00 5 March 2021
R1#copy running-config startup-config 
Destination filename [startup-config]? 
Building configuration...
[OK]

```

* Настройка основных параметров для каждого коммутатора.

1. S1
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
S1#clock set 19:14:00 5 March 2021
S1#copy running-config startup-config 

```

1. S2
```

Switch#enable
Switch#configure terminal 
Switch(config)#hostname S2
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
S1#clock set 19:14:00 5 March 2021
S1#copy running-config startup-config 

```

