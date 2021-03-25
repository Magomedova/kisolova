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
S1#clock set 19:14:00 24 March 2021
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
S1#clock set 19:14:00 24 March 2021
S1#copy running-config startup-config 

```

* Настройка хостов ПК.

1. ПК-А

![ПК_А](https://user-images.githubusercontent.com/5254857/112508228-6e50b980-8da0-11eb-8be1-50b1dbece29c.png)

2. ПК-В

![ПК_В](https://user-images.githubusercontent.com/5254857/112508282-76105e00-8da0-11eb-880e-7b4db0656740.png)

#### Создание сетей VLAN и назначение портов коммутатора

* Создание Vlan
```
S1(config)#vlan 3
S1(config-vlan)#name Management
S1(config-vlan)#vlan 4
S1(config-vlan)#name Operations
S1(config-vlan)#vlan 7
S1(config-vlan)#name ParkingLot
S1(config-vlan)#vlan 8
S1(config-vlan)#name Native
```
* Назначение VLAN интерфейсам коммутатора.
```
S2(config)#interface fa0/18
S2(config-if)#switchport mode access
S2(config-if)#switchport access vlan 4
```
* Назначение vlan ParkingLot  на все не используемые порты, с последующим отключением
```
S1(config)#interface range fa0/2 - 4 , fa0/7 - 24, gi0/1 - 2
S1(config-if-range)#switchport mode access 
S1(config-if-range)#switchport access vlan 7
S1(config-if-range)#shutdown
```
* Обозначение интерфейса управления с маской
```
S1(config)#interface vlan 3
S1(config-if)#ip address 192.168.3.11 255.255.255.0
S1(config-if)#no shutdown 
```
* Вывод команды show vlan brief:

S1>sh vlan
```
VLAN Name                             Status    Ports
---- -------------------------------- --------- -------------------------------
1    default                          active    
3    Management                       active    Fa0/6
4    Operation                        active    
7    ParkingLot                       active    Fa0/2, Fa0/3, Fa0/4, Fa0/7
                                                Fa0/8, Fa0/9, Fa0/10, Fa0/11
                                                Fa0/12, Fa0/13, Fa0/14, Fa0/15
                                                Fa0/16, Fa0/17, Fa0/18, Fa0/19
                                                Fa0/20, Fa0/21, Fa0/22, Fa0/23
                                                Fa0/24, Gig0/1, Gig0/2
```
####  Настройка магистрали 802.1Q на коммутаторах
* Настройка транковых портов
```
S1(config)#interface fa0/1
S1(config-if)#switchport mode trunk
```
* Установка нативного влана и разрешение прохождения vlan 3,4,8 
```
S1(config-if)#switchport trunk native vlan 8
S1(config-if)#switchport trunk allowed vlan 3,4,8
```
* Вывод команды show interfaces trunk
```
Port        Mode         Encapsulation  Status        Native vlan
Fa0/1       on           802.1q         trunking      8
Fa0/5       on           802.1q         trunking      8

Port        Vlans allowed on trunk
Fa0/1       3-4,8
Fa0/5       3-4,8

Port        Vlans allowed and active in management domain
Fa0/1       3,4
Fa0/5       3,4

Port        Vlans in spanning tree forwarding state and not pruned
Fa0/1       3,4
Fa0/5       3,4
```

####  Настройка маршрутизатора
* Настройка магистрального GigabitEthernet 0/0/1 и сабынтерфейсов 
```
R1(config)#interface gigabitEthernet 0/0/1
R1(config-if)#no shut
R1(config)#interface gi0/0/1.3
R1(config-subif)#description Management
R1(config-subif)#encapsulation dot1Q 3
R1(config-subif)#ip address 192.168.3.1 255.255.255.0
R1(config-subif)#interface gi0/0/1.4
R1(config-subif)#description Operations
R1(config-subif)#encapsulation dot1Q 4
R1(config-subif)#ip address 192.168.4.1 255.255.255.0
R1(config-subif)#interface gi0/0/1.8
R1(config-subif)#description Native
R1(config-subif)#encapsulation dot1Q 8 Native
```
* Вывод команды show ip interface brief:
```
Interface              IP-Address      OK? Method Status                Protocol 
GigabitEthernet0/0/0   unassigned      YES unset  administratively down down 
GigabitEthernet0/0/1   unassigned      YES unset  up                    up 
GigabitEthernet0/0/1.3 192.168.3.1     YES manual up                    up 
GigabitEthernet0/0/1.4 192.168.4.1     YES manual up                    up 
GigabitEthernet0/0/1.8 unassigned      YES unset  up                    up 
Vlan1                  unassigned      YES unset  administratively down down
```

### Проверка
* Эхо-запрос с компьютера PC-A на его шлюз по умолчанию.

![pingatogate](https://user-images.githubusercontent.com/5254857/112514353-46645480-8da6-11eb-80cc-807180136b7f.png)

* Пинг с ПК-А на ПК-Б

![A-B](https://user-images.githubusercontent.com/5254857/112515709-91329c00-8da7-11eb-8988-5f583d406706.png)

* Пинг с ПК-А на S2

![A-S2](https://user-images.githubusercontent.com/5254857/112516240-1fa71d80-8da8-11eb-9a9d-d35c90e68a0e.png)

* tracert ПК-B

![tracert](https://user-images.githubusercontent.com/5254857/112516552-73196b80-8da8-11eb-88ba-5a8b6a0f660e.png)

