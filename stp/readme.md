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

### Часть 2. Определение корневого моста.
1. Шаг 1:	Отключите все порты на коммутаторах.
```
S1(config)#int range fa0/1-24
S1(config-if-range)#shutdown 
S1(config)#int range gigabitEthernet 0/1-2
S1(config-if-range)#shutdown
```
2.	Настройте подключенные порты в качестве транковых.
```
S1(config-if-range)#int rang fa 0/1-4
S1(config-if-range)#sw mode trunk 
``` 
3. Включите порты fa0/2 fa0/4 на всех коммутаторах.
```
S1(config)#int range fa0/2, fa0/4
S1(config-if-range)#no shut
S1(config-if-range)#no shutdown 
```
4. Отобразите данные протокола spanning-tree.
*S1
```
S1#sh spanning-tree 
VLAN0001
  Spanning tree enabled protocol ieee
  Root ID    Priority    32769
             Address     0005.5E7D.2E45
             Cost        19
             Port        2(FastEthernet0/2)
             Hello Time  2 sec  Max Age 20 sec  Forward Delay 15 sec

  Bridge ID  Priority    32769  (priority 32768 sys-id-ext 1)
             Address     0090.0CE4.AC88
             Hello Time  2 sec  Max Age 20 sec  Forward Delay 15 sec
             Aging Time  20

Interface        Role Sts Cost      Prio.Nbr Type
---------------- ---- --- --------- -------- --------------------------------
Fa0/2            Root FWD 19        128.2    P2p
Fa0/4            Desg FWD 19        128.4    P2p
```
*S2
```
S2#sh spanning-tree 
VLAN0001
  Spanning tree enabled protocol ieee
  Root ID    Priority    32769
             Address     0005.5E7D.2E45
             This bridge is the root
             Hello Time  2 sec  Max Age 20 sec  Forward Delay 15 sec

  Bridge ID  Priority    32769  (priority 32768 sys-id-ext 1)
             Address     0005.5E7D.2E45
             Hello Time  2 sec  Max Age 20 sec  Forward Delay 15 sec
             Aging Time  20

Interface        Role Sts Cost      Prio.Nbr Type
---------------- ---- --- --------- -------- --------------------------------
Fa0/4            Desg FWD 19        128.4    P2p
Fa0/2            Desg FWD 19        128.2    P2p
```
*S3
```
S3#sh spanning-tree 
VLAN0001
  Spanning tree enabled protocol ieee
  Root ID    Priority    32769
             Address     0005.5E7D.2E45
             Cost        19
             Port        2(FastEthernet0/2)
             Hello Time  2 sec  Max Age 20 sec  Forward Delay 15 sec

  Bridge ID  Priority    32769  (priority 32768 sys-id-ext 1)
             Address     00D0.BA18.407E
             Hello Time  2 sec  Max Age 20 sec  Forward Delay 15 sec
             Aging Time  20

Interface        Role Sts Cost      Prio.Nbr Type
---------------- ---- --- --------- -------- --------------------------------
Fa0/4            Altn BLK 19        128.4    P2p
Fa0/2            Root FWD 19        128.2    P2p
```
5. С учетом выходных данных, поступающих с коммутаторов, ответьте на следующие вопросы.

Какой коммутатор является корневым мостом?                                             | S2
---------------------------------------------------------------------------------------|------------------------
Почему этот коммутатор был выбран протоколом spanning-tree в качестве корневого моста? | В нашем случае одинаковый Bridge Priority (одинаковый vlan) ,поэтому выбирается тот, у                                                                                          | кого меньше mac-адреса.
Какие порты на коммутаторе являются корневыми портами?                                 | S1 : Fa0/2 ; S3 : Fa0/2
Какие порты на коммутаторе являются назначенными портами?                              | S1 : Fa0/4
                                                                                       | S2 : Fa0/2 и Fa0/4
Какой порт отображается в качестве альтернативного и в настоящее время заблокирован?   | S3 : Fa0/4
Почему протокол spanning-tree выбрал этот порт в качестве невыделенного порта?         | т.к. BID на S2 меньше, чем на S3

### Часть 3. Наблюдение за процессом выбора протоколом STP порта, исходя из стоимости портов
1. Шаг 1:	Определите коммутатор с заблокированным портом.
S3
*S3
```
S3#sh spanning-tree 
VLAN0001
  Spanning tree enabled protocol ieee
  Root ID    Priority    32769
             Address     0005.5E7D.2E45
             Cost        19
             Port        2(FastEthernet0/2)
             Hello Time  2 sec  Max Age 20 sec  Forward Delay 15 sec

  Bridge ID  Priority    32769  (priority 32768 sys-id-ext 1)
             Address     00D0.BA18.407E
             Hello Time  2 sec  Max Age 20 sec  Forward Delay 15 sec
             Aging Time  20

Interface        Role Sts Cost      Prio.Nbr Type
---------------- ---- --- --------- -------- --------------------------------
Fa0/4            Altn BLK 19        128.4    P2p
Fa0/2            Root FWD 19        128.2    P2p
```

