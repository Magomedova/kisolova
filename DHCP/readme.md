# Lab - Configure Router-on-a-Stick Inter-VLAN Routing

## Задание

1. Организовать сеть и провести базовую настройку устройств
2. Создать ВЛАНы и назначить на порту коммутаторов
3. Настроить 802.1Q транк между коммутаторами
4. Настроить маршрутизацию между ВЛАН на маршрутизаторе
5. Убедиться что все работает

### Топология



### Таблица адресации
| Device | Interface      | IP address   | Subnet Mask     | Default Gateway |
| ------ | -------------- | ------------ | -------------   | --------------- |
| R1     | G0/0/0         | 10.0.0.1     | 255.255.255.252 | N/A             |
|        | G0/0/1         | N/A          | N/A             | N/A             |
|        | G0/0/1.100     |              |                 | N/A             |
|        | G0/0/1.200     |              |                 | N/A             |
|        | G0/0/1.1000    | N/A          | N/A             | N/A             | 
| R2     | G0/0/0         | 10.0.0.2     | 255.255.2552520 | N/A             |
|        | G0/0/1         |              |                 |                 |
| S1     | Vlan 200       |              |                 |                 |
| S2     | Vlan 1         |              |                 |                 |
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

