University: [ITMO University](https://itmo.ru/ru/)

Faculty: [FICT](https://fict.itmo.ru)

Course: [Introduction in routing](https://github.com/itmo-ict-faculty/introduction-in-routing)

Year: 2024/2025

Group: K3321

Author: Sadovaya Anastasia Romanovna

Date of create: 25.09.2024

Date of finished: 16.10.2024

# Лабораторная работ №1 "Установка ContainerLab и развертывание тестовой сети связи"

## Цель:
Ознакомиться с инструментом ContainerLab и методами работы с ним, изучить работу VLAN, IP адресации и т.д.

## Ход работы:
1. После установки необходимого ПО(ContainerLab и образа MikroTik), был написан файл lab1.clab.yaml, описывающий все узлы и соединения между ними через mgmt сеть:

```
name: lab-1

mgmt:
  network: static
  ipv4-subnet: 192.168.100.0/24

topology:
  nodes:
    R01:
      kind: vr-mikrotik_ros
      image: vrnetlab/vr-routeros:6.47.9
      mgmt-ipv4: 192.168.100.60
    SW01:
      kind: vr-mikrotik_ros
      image: vrnetlab/vr-routeros:6.47.9
      mgmt-ipv4: 192.168.100.30
    SW02:
      kind: vr-mikrotik_ros
      image: vrnetlab/vr-routeros:6.47.9
      mgmt-ipv4: 192.168.100.40
    SW03:
      kind: vr-mikrotik_ros
      image: vrnetlab/vr-routeros:6.47.9
      mgmt-ipv4: 192.168.100.50
    PC1:
      kind: linux
      image: alpine:latest
      mgmt-ipv4: 192.168.100.10
    PC2:
      kind: linux
      image: alpine:latest
      mgmt-ipv4: 192.168.100.20

  links:
    - endpoints: ["R01:eth2", "SW01:eth2"]
    - endpoints: ["SW01:eth3", "SW02:eth2"]
    - endpoints: ["SW01:eth4", "SW03:eth2"]
    - endpoints: ["SW02:eth3", "PC1:eth2"]
    - endpoints: ["SW03:eth3", "PC2:eth2"]
```
2. Далее необходимо было подключиться к сетевым устройства через ssh(ssh admin@xx.xx.xx.xx) и прописать их конфигурацию:

   **Router(R01.TEST)**
```
/interface vlan
add interface=ether3 name=vlan10 vlan-id=10
add interface=ether3 name=vlan20 vlan-id=20
/interface wireless security-profiles
set [ find default=yes ] supplicant-identity=MikroTik
/ip pool
add name=pool10 ranges=10.10.10.10-10.10.10.254
add name=pool20 ranges=10.10.20.10-10.10.20.254
/ip dhcp-server
add address-pool=pool10 disabled=no interface=vlan10 name=dhcp10
add address-pool=pool20 disabled=no interface=vlan20 name=dhcp20
/ip address
add address=10.10.10.1/24 interface=vlan10 network=10.10.10.0
add address=10.10.20.1/24 interface=vlan20 network=10.10.20.0
/ip dhcp-client
add disabled=no interface=ether1
/ip dhcp-server network
add address=10.10.10.0/24 gateway=10.10.10.1
add address=10.10.20.0/24 gateway=10.10.20.1
/system identity
set name=R01.TEST
```

**Switch1(SW01.L3.01.TEST)**
```
/interface bridge
add name=bridge10
add name=bridge20
/interface vlan
add interface=ether3 name=vlan10 vlan-id=10
add interface=ether3 name=vlan20 vlan-id=20
add interface=ether4 name=vlan100 vlan-id=10
add interface=ether5 name=vlan200 vlan-id=20
/interface wireless security-profiles
set [ find default=yes ] supplicant-identity=MikroTik
/interface bridge port
add bridge=bridge10 interface=vlan10
add bridge=bridge20 interface=vlan20
add bridge=bridge10 interface=vlan100
add bridge=bridge20 interface=vlan200
/ip dhcp-client
add disabled=no interface=ether1
add disabled=no interface=bridge10
add disabled=no interface=bridge20
/system identity
set name=SW01.L3.01.TEST
```

**Switch2(SW02.L3.01.TEST)**
```
/interface bridge
add name=bridge10
/interface vlan
add interface=ether3 name=vlan10 vlan-id=10
/interface wireless security-profiles
set [ find default=yes ] supplicant-identity=MikroTik
/interface bridge port
add bridge=bridge10 interface=vlan10
add bridge=bridge10 interface=ether4
/ip dhcp-client
add disabled=no interface=ether1
add disabled=no interface=bridge10
/system identity
set name=SW02.L3.01.TEST
```

**Switch3(SW02.L3.02.TEST)**
```
/interface bridge
add name=bridge20
/interface vlan
add interface=ether3 name=vlan20 vlan-id=20
/interface wireless security-profiles
set [ find default=yes ] supplicant-identity=MikroTik
/interface bridge port
add bridge=bridge20 interface=vlan20
add bridge=bridge20 interface=ether4
/ip dhcp-client
add disabled=no interface=ether1
add disabled=no interface=bridge20
/system identity
set name=SW02.L3.02.TEST
```

3. И для проверки работы dhcp-сервера и локальной связности сети ниже представлены результаты команды ip neighbor print и пинги для различных устройств:

**R01**
![ip neighbor print](https://github.com/sadnastya/2023_2024-introduction_in_routing-k3321-sadovaya_a_r/blob/main/lab1/images/neighbors_router.png)
![ping](https://github.com/sadnastya/2023_2024-introduction_in_routing-k3321-sadovaya_a_r/blob/main/lab1/images/ping_from_router.png)

**SW01**
![neighbor print](https://github.com/sadnastya/2023_2024-introduction_in_routing-k3321-sadovaya_a_r/blob/main/lab1/images/neighbors_sw_01.png)

**SW02.02**
![all](https://github.com/sadnastya/2023_2024-introduction_in_routing-k3321-sadovaya_a_r/blob/main/lab1/images/ping_from_sw02_2.png)


## Вывод:
В результате выполнения лабораторной работы были освоены методы работы с ContainerLab, была изучены и практически настроены VLAN, DHCP-сервер и др.






