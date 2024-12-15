
University: [ITMO University](https://itmo.ru/ru/)'

Faculty: [FICT](https://fict.itmo.ru)

Course: [Introduction in routing](https://github.com/itmo-ict-faculty/introduction-in-routing)

Year: 2024/2025

Group: K3321

Author: Sadovaya Anastasia Romanovna

Lab: Lab3

Date of create: 11.12.2024

Date of finished: 15.12.2024

# Лабораторная работ №3 "Эмуляция распределенной корпоративной сети связи, настройка OSPF и MPLS, организация первого EoMPLS"


## Цель:

Изучить протоколы OSPF и MPLS, механизмы организации EoMPLS.

## Ход работы:

1. Был написан файл lab3.clab.yaml, описывающий все узлы и соединения между ними через mgmt сеть. Наша сеть состоит из 6 роутеров Mikrotik и 2 linux-устройств (симулируют ПК клиента и сервер):

```
name: lab-3

mgmt:
  network: static
  ipv4-subnet: 192.168.100.0/24

topology:
  nodes:
    R01.SPB:
      kind: vr-mikrotik_ros
      image: vrnetlab/vr-routeros:6.47.9
      mgmt-ipv4: 192.168.100.10
    R01.MSK:
      kind: vr-mikrotik_ros
      image: vrnetlab/vr-routeros:6.47.9
      mgmt-ipv4: 192.168.100.20
    R01.HKI:
      kind: vr-mikrotik_ros
      image: vrnetlab/vr-routeros:6.47.9
      mgmt-ipv4: 192.168.100.30
    R01.LBN:
      kind: vr-mikrotik_ros
      image: vrnetlab/vr-routeros:6.47.9
      mgmt-ipv4: 192.168.100.40
    R01.LND:
      kind: vr-mikrotik_ros
      image: vrnetlab/vr-routeros:6.47.9
      mgmt-ipv4: 192.168.100.50
    R01.NY:
      kind: vr-mikrotik_ros
      image: vrnetlab/vr-routeros:6.47.9
      mgmt-ipv4: 192.168.100.60
    
    PC1:
      kind: linux
      image: alpine:latest
      mgmt-ipv4: 192.168.100.15
    SGI-PRISM:
      kind: linux
      image: alpine:latest
      mgmt-ipv4: 192.168.100.65

  links:
    - endpoints: ["R01.NY:eth2", "R01.LBN:eth2"]
    - endpoints: ["R01.NY:eth3", "R01.LND:eth2"]
    - endpoints: ["R01.NY:eth4", "SGI-PRISM:eth"]
    - endpoints: ["R01.LND:eth3", "R01.HKI:eth2"]
    - endpoints: ["R01.HKI:eth3", "R01.LBN:eth3"]
    - endpoints: ["R01.HKI:eth4", "R01.SPB:eth2"]
    - endpoints: ["R01.SPB:eth3", "R01.MSK:eth3"]
    - endpoints: ["R01.MSK:eth2", "R01.LBN:eth4"]
    - endpoints: ["R01.SPB:eth4", "PC1:eth"]
```
2. Далее необходимо было подключиться к сетевым устройства через ssh(ssh admin@xx.xx.xx.xx) и прописать их конфигурацию в соответсвии с заданием. Принципиально настройка роутеров разделилась на:

* PE-роутеры (Provider-edge) — пограничные MPLS маршрутизаторы (SPB и NY):

**R01.SPB**

```
/interface bridge
add name=loopbackbridge
add name=vpn
/interface ethernet
set [ find default-name=ether1 ] disable-running-check=no
set [ find default-name=ether2 ] disable-running-check=no
set [ find default-name=ether3 ] disable-running-check=no
set [ find default-name=ether4 ] disable-running-check=no
set [ find default-name=ether5 ] disable-running-check=no
/interface vpls
add disabled=no l2mtu=1500 mac-address=02:BD:27:52:DB:8E name=EoMPLS remote-peer=10.10.10.7 vpls-id=100:100
/interface wireless security-profiles
set [ find default=yes ] supplicant-identity=MikroTik
/routing ospf instance
set [ find default=yes ] router-id=10.10.10.2
/interface bridge port
add bridge=vpn interface=ether5
add bridge=vpn interface=EoMPLS
/ip address
add address=172.31.255.30/30 interface=ether1 network=172.31.255.28
add address=10.10.10.2 interface=loopbackbridge network=10.10.10.2
add address=100.100.106.102/30 interface=ether3 network=100.100.106.100
add address=100.100.107.101/30 interface=ether4 network=100.100.107.100
/ip dhcp-client
add disabled=no interface=ether1
/mpls ldp
set enabled=yes lsr-id=10.10.10.2 transport-address=10.10.10.2
/mpls ldp interface
add interface=ether3
add interface=ether4
add interface=ether5
/routing ospf network
add area=backbone network=10.10.10.2/32
add area=backbone network=100.100.106.100/30
add area=backbone network=100.100.107.100/30
add area=backbone network=100.100.108.100/30
/system identity
set name=R01.SPB
```

**R01.NY**

```
/interface bridge
add name=loopbackbridge
add name=vpn
/interface ethernet
set [ find default-name=ether1 ] disable-running-check=no
set [ find default-name=ether2 ] disable-running-check=no
set [ find default-name=ether3 ] disable-running-check=no
set [ find default-name=ether4 ] disable-running-check=no
set [ find default-name=ether5 ] disable-running-check=no
/interface vpls
add disabled=no l2mtu=1500 mac-address=02:BD:27:52:DB:8E name=EoMPLS remote-peer=10.10.10.2 vpls-id=100:100
/interface wireless security-profiles
set [ find default=yes ] supplicant-identity=MikroTik
/routing ospf instance
set [ find default=yes ] router-id=10.10.10.7
/interface bridge port
add bridge=vpn interface=ether5
add bridge=vpn interface=EoMPLS
/ip address
add address=172.31.255.30/30 interface=ether1 network=172.31.255.28
add address=10.10.10.7 interface=loopbackbridge network=10.10.10.7
add address=100.100.102.101/30 interface=ether4 network=100.100.102.100
add address=100.100.101.101/30 interface=ether3 network=100.100.101.100
/ip dhcp-client
add disabled=no interface=ether1
/mpls ldp
set enabled=yes lsr-id=10.10.10.7 transport-address=10.10.10.7
/mpls ldp interface
add interface=ether3
add interface=ether4
add interface=ether5
/routing ospf network
add area=backbone network=10.10.10.7/32
add area=backbone network=100.100.101.100/30
add area=backbone network=100.100.102.100/30
/system identity
set name=R01.NY
```

* P-роутеры (provider router) — MPLS маршрутизаторы, которые пропускают MPLS-трафик через себя(MSK, HKI, LBN, LND)

**R01.MSK**

```
/interface bridge
add name=loopbackbridge
/interface ethernet
set [ find default-name=ether1 ] disable-running-check=no
set [ find default-name=ether2 ] disable-running-check=no
set [ find default-name=ether3 ] disable-running-check=no
set [ find default-name=ether4 ] disable-running-check=no
/interface wireless security-profiles
set [ find default=yes ] supplicant-identity=MikroTik
/routing ospf instance
set [ find default=yes ] router-id=10.10.10.3
/ip address
add address=172.31.255.30/30 interface=ether1 network=172.31.255.28
add address=10.10.10.3 interface=loopbackbridge network=10.10.10.3
add address=100.100.107.102/30 interface=ether4 network=100.100.107.100
add address=100.100.105.102/30 interface=ether3 network=100.100.105.100
/ip dhcp-client
add disabled=no interface=ether1
/mpls ldp
set enabled=yes lsr-id=10.10.10.3 transport-address=10.10.10.3
/mpls ldp interface
add interface=ether3
add interface=ether4
/routing ospf network
add area=backbone network=10.10.10.3/32
add area=backbone network=100.100.107.100/30
add area=backbone network=100.100.105.100/30
/system identity
set name=R01.MSK
```

**R01.HKI**

```
/interface bridge
add name=loopbackbridge
/interface ethernet
set [ find default-name=ether1 ] disable-running-check=no
set [ find default-name=ether2 ] disable-running-check=no
set [ find default-name=ether3 ] disable-running-check=no
set [ find default-name=ether4 ] disable-running-check=no
set [ find default-name=ether5 ] disable-running-check=no
/interface wireless security-profiles
set [ find default=yes ] supplicant-identity=MikroTik
/routing ospf instance
set [ find default=yes ] router-id=10.10.10.5
/ip address
add address=172.31.255.30/30 interface=ether1 network=172.31.255.28
add address=10.10.10.5 interface=loopbackbridge network=10.10.10.5
add address=100.100.103.102/30 interface=ether3 network=100.100.103.100
add address=100.100.104.101/30 interface=ether4 network=100.100.104.100
add address=100.100.106.101/30 interface=ether5 network=100.100.106.100
/ip dhcp-client
add disabled=no interface=ether1
/mpls ldp
set enabled=yes lsr-id=10.10.10.5 transport-address=10.10.10.5
/mpls ldp interface
add interface=ether3
add interface=ether4
add interface=ether5
/routing ospf network
add area=backbone network=10.10.10.5/32
add area=backbone network=100.100.103.100/30
add area=backbone network=100.100.106.100/30
add area=backbone network=100.100.104.100/30
/system identity
set name=R01.HKI
```

**R01.LND**

```
/interface bridge
add name=loopbackbridge
/interface ethernet
set [ find default-name=ether1 ] disable-running-check=no
set [ find default-name=ether2 ] disable-running-check=no
set [ find default-name=ether3 ] disable-running-check=no
set [ find default-name=ether4 ] disable-running-check=no
/interface wireless security-profiles
set [ find default=yes ] supplicant-identity=MikroTik
/routing ospf instance
set [ find default=yes ] router-id=10.10.10.6
/ip address
add address=172.31.255.30/30 interface=ether1 network=172.31.255.28
add address=10.10.10.6 interface=loopbackbridge network=10.10.10.6
add address=100.100.102.102/30 interface=ether3 network=100.100.102.100
add address=100.100.103.101/30 interface=ether4 network=100.100.103.100
/ip dhcp-client
add disabled=no interface=ether1
/mpls ldp
set enabled=yes lsr-id=10.10.10.6 transport-address=10.10.10.6
/mpls ldp interface
add interface=ether3
add interface=ether4
/routing ospf network
add area=backbone network=10.10.10.6/32
add area=backbone network=100.100.102.100/30
add area=backbone network=100.100.103.100/30
/system identity
set name=R01.LND
```

**R01.LBN**

```
/interface bridge
add name=loopbackbridge
/interface ethernet
set [ find default-name=ether1 ] disable-running-check=no
set [ find default-name=ether2 ] disable-running-check=no
set [ find default-name=ether3 ] disable-running-check=no
set [ find default-name=ether4 ] disable-running-check=no
set [ find default-name=ether5 ] disable-running-check=no
/interface wireless security-profiles
set [ find default=yes ] supplicant-identity=MikroTik
/routing ospf instance
set [ find default=yes ] router-id=10.10.10.4
/ip address
add address=172.31.255.30/30 interface=ether1 network=172.31.255.28
add address=10.10.10.4 interface=loopbackbridge network=10.10.10.4
add address=100.100.101.102/30 interface=ether3 network=100.100.101.100
add address=100.100.104.102/30 interface=ether4 network=100.100.104.100
add address=100.100.105.101/30 interface=ether5 network=100.100.105.100
/ip dhcp-client
add disabled=no interface=ether1
/mpls ldp
set enabled=yes lsr-id=10.10.10.4 transport-address=10.10.10.4
/mpls ldp interface
add interface=ether3
add interface=ether4
add interface=ether5
/routing ospf network
add area=backbone network=10.10.10.4/32
add area=backbone network=100.100.105.100/30
add area=backbone network=100.100.101.100/30
add area=backbone network=100.100.104.100/30
/system identity
set name=R01.LBN
```

3. Получились следующие маршруты на каждом роутере:

**R01.SPB**

![route_spb](https://github.com/sadnastya/2023_2024-introduction_in_routing-k3321-sadovaya_a_r/blob/main/lab3/images/route_spb.png)

**R01.NY**

![route_ny](https://github.com/sadnastya/2023_2024-introduction_in_routing-k3321-sadovaya_a_r/blob/main/lab3/images/route_ny.png)

**R01.MSK**

![route_msk](https://github.com/sadnastya/2023_2024-introduction_in_routing-k3321-sadovaya_a_r/blob/main/lab3/images/route_msk.png)

**R01.LBN**

![route_lbn](https://github.com/sadnastya/2023_2024-introduction_in_routing-k3321-sadovaya_a_r/blob/main/lab3/images/route_lbn.png)

**R01.LND**

![route_lnd](https://github.com/sadnastya/2023_2024-introduction_in_routing-k3321-sadovaya_a_r/blob/main/lab3/images/route_lnd.png)

**R01.HKI**

![route_hki](https://github.com/sadnastya/2023_2024-introduction_in_routing-k3321-sadovaya_a_r/blob/main/lab3/images/route_hki.png)

4. Также необходимо было повесить на интерфейсы клиентских машин ip-адреса из одной сети:

![настройка_пк](https://github.com/sadnastya/2023_2024-introduction_in_routing-k3321-sadovaya_a_r/blob/main/lab3/images/add_ip_pc.png)

5. После всех необходимых настроек получили следующую схему сети:
   
![схема_сети](https://github.com/sadnastya/2023_2024-introduction_in_routing-k3321-sadovaya_a_r/blob/main/lab3/images/schema_lab3.png)
   
6. Для проверки работоспособности сети пропингуем с PC1 наш сервер SGI-PRISM и в обратную сторону:

![ping1](https://github.com/sadnastya/2023_2024-introduction_in_routing-k3321-sadovaya_a_r/blob/main/lab3/images/ping1.png)

![ping2](https://github.com/sadnastya/2023_2024-introduction_in_routing-k3321-sadovaya_a_r/blob/main/lab3/images/ping2.png)

7. Посмотрим трассировку с R01.SPB в R01.NY через различные пути:

![tracerote](https://github.com/sadnastya/2023_2024-introduction_in_routing-k3321-sadovaya_a_r/blob/main/lab3/images/traceroute.png)

## Вывод:

В результате выполнения лабораторной работы были освоены методы работы с протоколами OSPF и MPLS, изучен и настроен механизм организации EoMPLS.