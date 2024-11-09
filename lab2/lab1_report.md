
University: [ITMO University](https://itmo.ru/ru/)'

Faculty: [FICT](https://fict.itmo.ru)

Course: [Introduction in routing](https://github.com/itmo-ict-faculty/introduction-in-routing)

Year: 2024/2025

Group: K3321

Author: Sadovaya Anastasia Romanovna

Lab: Lab1

Date of create: 25.10.2024

Date of finished: 09.11.2024

# Лабораторная работ №2 "Эмуляция распределенной корпоративной сети связи, настройка статической маршрутизации между филиалами"


## Цель:

Ознакомиться с принципами планирования IP адресов, настройке статической маршрутизации и сетевыми функциями устройств.

## Ход работы:

1. Был написан файл lab2.clab.yaml, описывающий все узлы и соединения между ними через mgmt сеть:

```
name: lab-2

mgmt:
  network: static
  ipv4-subnet: 192.168.100.0/24

topology:
  nodes:
    R01.MSK:
      kind: vr-mikrotik_ros
      image: vrnetlab/vr-routeros:6.47.9
      mgmt-ipv4: 192.168.100.10
    R02.BRL:
      kind: vr-mikrotik_ros
      image: vrnetlab/vr-routeros:6.47.9
      mgmt-ipv4: 192.168.100.20
    R03.FRT:
      kind: vr-mikrotik_ros
      image: vrnetlab/vr-routeros:6.47.9
      mgmt-ipv4: 192.168.100.30
    PC1:
      kind: linux
      image: alpine:latest
      mgmt-ipv4: 192.168.100.40
    PC2:
      kind: linux
      image: alpine:latest
      mgmt-ipv4: 192.168.100.50
    PC3:
      kind: linux
      image: alpine:latest
      mgmt-ipv4: 192.168.100.60

  links:
      - endpoints: ["R01.MSK:eth2", "R02.BRL:eth2"]
      - endpoints: ["R01.MSK:eth3", "R03.FRT:eth3"]
      - endpoints: ["R01.MSK:eth4", "PC1:eth2"]
      - endpoints: ["R02.BRL:eth3", "R03.FRT:eth2"]
      - endpoints: ["R02.BRL:eth4", "PC2:eth4"]
      - endpoints: ["R03.FRT:eth4", "PC3:eth2"]
```
2. Далее необходимо было подключиться к сетевым устройства через ssh(ssh admin@xx.xx.xx.xx) и прописать их конфигурацию:

**R01.MSK**

```
/interface ethernet
set [ find default-name=ether1 ] disable-running-check=no
set [ find default-name=ether2 ] disable-running-check=no
set [ find default-name=ether3 ] disable-running-check=no
set [ find default-name=ether4 ] disable-running-check=no
set [ find default-name=ether5 ] disable-running-check=no
/interface wireless security-profiles
set [ find default=yes ] supplicant-identity=MikroTik
/ip pool
add name=pool_msk ranges=10.10.10.10-10.10.10.253
/ip dhcp-server
add address-pool=pool_msk disabled=no interface=ether5 name=dhcp_server_msk
/ip address
add address=172.31.255.30/30 interface=ether1 network=172.31.255.28
add address=20.10.0.2/24 interface=ether3 network=20.10.0.0
add address=20.30.0.2/24 interface=ether4 network=20.30.0.0
add address=10.10.10.2/24 interface=ether5 network=10.10.10.0
/ip dhcp-client
add disabled=no interface=ether1
/ip dhcp-server network
add address=10.10.10.0/24 dns-server=8.8.8.8 gateway=10.10.10.2
/ip route
add distance=1 dst-address=10.10.20.0/24 gateway=20.10.0.3
add distance=1 dst-address=10.10.30.0/24 gateway=20.30.0.3
/system identity
set name=R01.MSK
```

**R02.BRL**

```
/interface ethernet
set [ find default-name=ether1 ] disable-running-check=no
set [ find default-name=ether2 ] disable-running-check=no
set [ find default-name=ether3 ] disable-running-check=no
set [ find default-name=ether4 ] disable-running-check=no
set [ find default-name=ether5 ] disable-running-check=no
/interface wireless security-profiles
set [ find default=yes ] supplicant-identity=MikroTik
/ip pool
add name=pool_brl ranges=10.10.20.50-10.10.20.253
/ip dhcp-server
add address-pool=pool_brl disabled=no interface=ether5 name=dhcp_server_brl
/ip address
add address=172.31.255.30/30 interface=ether1 network=172.31.255.28
add address=20.10.0.3/24 interface=ether3 network=20.10.0.0
add address=20.20.0.2/24 interface=ether4 network=20.20.0.0
add address=10.10.20.2/24 interface=ether5 network=10.10.20.0
/ip dhcp-client
add disabled=no interface=ether1
/ip dhcp-server network
add address=10.10.20.0/24 dns-server=8.8.8.8 gateway=10.10.20.2
/ip route
add distance=1 dst-address=10.10.10.0/24 gateway=20.10.0.2
add distance=1 dst-address=10.10.30.0/24 gateway=20.20.0.3
/system identity
set name=R02.BRL
```

**R03.FRT**

```
/interface ethernet
set [ find default-name=ether1 ] disable-running-check=no
set [ find default-name=ether2 ] disable-running-check=no
set [ find default-name=ether3 ] disable-running-check=no
set [ find default-name=ether4 ] disable-running-check=no
set [ find default-name=ether5 ] disable-running-check=no
/interface wireless security-profiles
set [ find default=yes ] supplicant-identity=MikroTik
/ip pool
add name=pool_frt ranges=10.10.30.50-10.10.30.253
/ip dhcp-server
add address-pool=pool_frt disabled=no interface=ether5 name=dhcp_server_frt
/ip address
add address=172.31.255.30/30 interface=ether1 network=172.31.255.28
add address=20.20.0.3/24 interface=ether3 network=20.20.0.0
add address=20.30.0.3/24 interface=ether4 network=20.30.0.0
add address=10.10.30.2/24 interface=ether5 network=10.10.30.0
/ip dhcp-client
add disabled=no interface=ether1
/ip dhcp-server network
add address=10.10.30.0/24 dns-server=8.8.8.8 gateway=10.10.30.2
/ip route
add distance=1 dst-address=10.10.10.0/24 gateway=20.30.0.2
add distance=1 dst-address=10.10.20.0/24 gateway=20.20.0.2
/system identity
set name=R03.FRT
```

3. Схема настроенной сети:
   
   ![схема_сети](https://github.com/sadnastya/2023_2024-introduction_in_routing-k3321-sadovaya_a_r/blob/main/lab2/images/schema_lab2.png)
   
4. И для проверки локальной связности сети ниже представлены результаты команды ip route print и пинги для различных устройств:

**R01.MSK**

![ip route print msk](https://github.com/sadnastya/2023_2024-introduction_in_routing-k3321-sadovaya_a_r/blob/main/lab2/images/msk_ping.png)

![ping msk](https://github.com/sadnastya/2023_2024-introduction_in_routing-k3321-sadovaya_a_r/blob/main/lab2/images/msk_route.png)

**R02.BRL**

![ip route print brl](https://github.com/sadnastya/2023_2024-introduction_in_routing-k3321-sadovaya_a_r/blob/main/lab2/images/brl_ping.png)

![ping brl](https://github.com/sadnastya/2023_2024-introduction_in_routing-k3321-sadovaya_a_r/blob/main/lab2/images/brl_route.png)

**R03.FRT**

![ip route print frt](https://github.com/sadnastya/2023_2024-introduction_in_routing-k3321-sadovaya_a_r/blob/main/lab2/images/frt_ping.png)

![ping frt](https://github.com/sadnastya/2023_2024-introduction_in_routing-k3321-sadovaya_a_r/blob/main/lab2/images/frt_route.png)


## Вывод:

В результате выполнения лабораторной работы были освоены методы работы с ContainerLab, была изучены и практически настроены статическая маршрутизация меджу роутерами, DHCP-сервер и др.