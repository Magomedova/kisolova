# Lab - Развертывание коммутируемой сети с резервными каналами

## Топология

![топология](https://user-images.githubusercontent.com/5254857/115736717-2bc8cf80-a394-11eb-9ab7-95bb6703e269.PNG)


## Таблица адресации

| Device | Interface | IP address   | Subnet Mask   |
| ------ | --------- | ------------ | ------------- |
| S1     | VLAN 1    | 192.168.1.1  | 255.255.255.0 |
| S2     | VLAN 1    | 192.168.1.2  | 255.255.255.0 |
| S3     | VLAN 1    | 192.168.1.3  | 255.255.255.0 | 


## Выполнение лабороторной работы
### Часть 1: 	Создание сети и настройка основных параметров устройства
1. Создайте сеть согласно топологии.

![сеть](https://user-images.githubusercontent.com/5254857/115738232-8a427d80-a395-11eb-85bf-9898294142db.png)

2. Настройте базовые параметры каждого коммутатора.
S1:

```
Switch(config)#hostname S1
S1(config)#no ip domain-lookup
S1(config)#enable secret class
S1(config)#line console 0
S1(config-line)#logging synchronous 
S1(config-line)#password cisco
S1(config-line)#login
S1(config-line)#exit
S1(config)#line vty 0 15
S1(config-line)#password cisco
S1(config-line)#login
S1(config-line)#exit
S1(config)#interface vlan 1
S1(config-if)#ip address 192.168.1.1 255.255.255.0
S1(config)#banner motd 'Authorized Accessonly'
S1(config)#end
S1#clock set 19:14:00 25 March 2021
S1#copy running-config startup-config 
```
2. Настройка S2 и S3 с сетевыми настройками, согласно таблице адресации.
3. Проверка связности:
* эхо-запрос от коммутатора S1 на коммутатор S2:
![S1-S2](https://user-images.githubusercontent.com/5254857/115741885-d2af6a80-a398-11eb-8b12-3100e4bbdd74.png)

* эхо-запрос от коммутатора S1 на коммутатор S3:
![S1-S3](https://user-images.githubusercontent.com/5254857/115741938-dcd16900-a398-11eb-8a9a-8cd1c8ab2f9e.png)

* эхо-запрос от коммутатора S2 на коммутатор S3:
![S2-S3](https://user-images.githubusercontent.com/5254857/115741958-e22eb380-a398-11eb-9ac9-f2a8eb534065.png)
