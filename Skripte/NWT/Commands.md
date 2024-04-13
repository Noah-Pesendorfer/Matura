# 1. Grundkonfiguration eines Switches oder Routers
## 1.1. Configuration Mode: Um Router zu konfigurieren
```
Router> enable
Router# configure terminal
```
## 1.2. Hostname: Name des Routers
```
hostname r2
```

## 1.3. Passwort aktivieren
```
Router(config)# enable password [password]
```

## 1.4. Konsolenpasswort vergeben
```
Router(config)# lin con 0
Router(config-line)# password [password]
Router(config-line)# login
Router(config-line)# exit
```

## 1.5. Telnet-Passwort vergeben
```
Router(config)# lin vty 0 15
Router(config-line)# password [password]
Router(config-line)# login
Router(config-line)# exit
```

## 1.6. Passwort Verschlüsselung
```
Router(config)# service password-encryption
```

## 1.7. Deaktivieren von automatischer DNS-Auflösung für ungültige Befehlseingaben
```
no ip domain-lookup
```

## 1.8. Konfiguration im Arbeitsspeicher --> Konfigurationsdatei im Flash-Speicher
```
copy running-config startup-config
```
<hr>

# 2. Router Interface Konfiguration
## 2.1. IPv4 Adresse + Subnetzmaske
```
Router> conf t
Router(config)# int g0/0
Router(config)# description Netzwerk links
Router(config)# ip address 192.168.10.254 255.255.255.0
Router(config)# no shutdown
```

- `conf t`: Wechselt in den **Konfigurationsmodus**.
- `int g0/0`: **Wechselt zur Konfiguration** der GigabitEthernet0/0-Schnittstelle.
- `description Netzwerk links`: Fügt eine **Beschreibung** zur Schnittstelle hinzu, die besagt, dass es sich um das Netzwerk links handelt.
- `ip address 192.168.10.254 255.255.255.0`: **Weist** der Schnittstelle die **IP-Adresse** 192.168.10.254 mit einem **Subnetzmaskenwert** von 255.255.255.0 **zu**.
- `no shutdown`: **Aktiviert** die **Schnittstelle**, sodass sie Datenverkehr weiterleiten kann.

## 2.2. Clock rate

```
conf t
int s0/1/0
description Router-Verbindung
ip address 172.17.25.10 255.255.255.252
clock rate 128000 (nur eine Seite...)
no shutdown
end
```


- `conf t`: **Konfigurationsmodus**
- `int s0/1/0`: Konfiguration **Serial0/1/0-Schnittstelle**.
- `description Router-Verbindung`: **Beschreibung**
- `ip address 172.17.25.10 255.255.255.252`: **IP-Adresse** 172.17.25.10, **Subnetzmaske** 255.255.255.252.
- `clock rate 128000`: **Clock-Rate** der seriellen Schnittstelle auf 128000 **Bits pro Sekunde**. **Nur auf einer Seite der Verbindung erforderlich**, normalerweise auf der **DTE (Data Terminal Equipment)-Seite**.
- `no shutdown`: **Aktiviert die Schnittstelle**
- `end`: **Verlässt den Konfigurationsmodus**
<hr>

# 3. Switch VLAN Konfigurieren:
## 3.1. Konfigurieren von VLAN (ID: 10) auf einem Switch
```
s1(config)# vlan 10
s1(config-vlan)# name [name]
s1(config-vlan)# exit
```
## 3.2. Konfigurieren eines Ports als Trunk-Port (Tagged Port)
```
Switch(config)# int g0/1
Switch(config-if)# switchport mode trunk
```

## 3.3. Konfigurieren eines Ports als Access-Port (Untagged Port) für ein bestimmes VLAN
```
s1(config)# int range fa0/13-24
s1(config-if-range)# switchport mode access
s1(config-if-range)# switchport access vlan 10
s1(config-if-range)# exit
```

## 3.4. Konfigurieren eines Sub-Interface für das VLAN 10
```
Router(config)# int g0/0.10
Router(config-subif)# encapsulation dot1Q 10
Router(config-subif)# ip address 192.168.10.1 255.255.255.0
Router(config-subif)# exit
```

- `int g0/0.20`: Dieser Befehl erstellt ein Subinterface für das VLAN 10 auf der GigabitEthernet-Schnittstelle 0/0 des Routers.
- `encapsulation dot1Q 10`: Legt die Encapsulation-Methode für das Sub-Interface für das VLAN 10 fest. Diese Encapsulation-Methode wird verwendet, um **Datenverkehr** für **verschiedene VLANs** über **dasselbe physische Interface** zu transportieren.
- `ip address 192.168.10.1 255.255.255.0`: Zuweisung IP-Adresse 192.168.10.1, Subnetzmaske von 255.255.255.0.
<hr>

# 4. Network Address Translation (NAT)
## 4.1. Statisches NAT konfigurieren
```
ISP(config)# int g0/1
ISP(config-if)# ip nat out
ISP(config-if)# exit
```

- `int g0/1`: **Konfigurationsmodus** - GigabitEthernet-Schnittstelle 0/1.
- `ip nat out`: Konfiguriert die **Schnittstelle als Ausgangsschnittstelle für NAT**. Router übersetzt IP-Adressen der von der Schnittstelle ausgehenden Quellpakete.

```
ISP(config)# int g0/0
ISP(config-if)# ip nat inside
ISP(config-if)# exit
```

- `int g0/0`: **Konfigurationsmodus** - GigabitEthernet-Schnittstelle 0/0.
- `ip nat inside`: Konfiguriert die **Schnittstelle als Eingangsschnittstelle für NAT**. Router übersetzt IP-Adressen der von der Schnittstelle empfangenen Zielpakete.

## 4.2. Übersetzung statisches NAT
```
ISP(config)# ip nat inside source static 192.168.20.10 100.100.100.100
```

- `ip nat inside source static 192.168.20.10 100.100.100.100`: **Statische Übersetzung für interne IP-Adresse** (192.168.20.10) **zu einer öffentlichen IP-Adresse** (100.100.100.100). 


- Wird verwendet, um **spezifische Server** im internen Netzwerk für **externe Benutzer** erreichbar zu machen, indem ihre interne IP-Adresse durch eine öffentliche IP-Adresse ersetzt wird.

## 4.3. NAT-Übersetzungen anzeigen lassen
```
ISP#show ip nat tr
Pro  Inside global     Inside local       Outside local      Outside global
---  100.100.100.100   192.168.20.10      ---                ---
```

## NAT-Overload mit PAT
```
ISP(config)#access-list 10 permit 192.168.10.0 255.255.255.0 
ISP(config)#ip nat inside source list 10 int g0/1 overload
```

- `access-list 10 permit 192.168.10.0 255.255.255.0`: Erstellt **Zugriffsliste** mit der Nummer 10, die den **IP-Verkehr vom Subnetz** 192.168.10.0/24 **erlaubt**.

- `ip nat inside source list 10 interface g0/1 overload`: Legt NAT-Konfiguration fest, um **alle IP-Adressen** aus der **Zugriffsliste 10** auf der **internen Schnittstelle** g0/1 zu **übersetzen** und dabei die **Überladungsmethode** zu verwenden. 

Die Überladung ermöglicht die Übersetzung mehrerer privater IP-Adressen in eine einzelne öffentliche IP-Adresse unter Verwendung verschiedener Ports.
<hr>

# Routing Protokolle einrichten
## Statisch IPv4
```
ip route 192.168.20.0 255.255.255.0 172.17.25.10
```
```
ip route 192.168.10.0 255.255.255.0 172.17.25.9
```
## Dynamisch mit RIPv2
```
r1(config)#router rip
r1(config-router)#version 2
r1(config-router)#network 192.168.10.0
r1(config-router)#network 172.17.25.8
r1(config-router)#exit
```

- `router rip`: **Konfigurationsmodus** - RIP-Routing-Protokoll.
- `version 2`: Legt die Version des RIP-Protokolls auf Version 2 fest.
- `network 192.168.10.0`: **Fügt** das **Netzwerk** 192.168.10.0 zum RIP-Routing-Prozess **hinzu**. Router **empfängt RIP-Pakete** auf dieser Schnittstelle und **sendet RIP-Updates** für dieses Netzwerk.

### Routing Tabelle anzeigen
```
r1#show ip route
Codes: L - local, C - connected, S - static, R - RIP, M - mobile, B - BGP
       D - EIGRP, EX - EIGRP external, O - OSPF, IA - OSPF inter area
       N1 - OSPF NSSA external type 1, N2 - OSPF NSSA external type 2
       E1 - OSPF external type 1, E2 - OSPF external type 2, E - EGP
       i - IS-IS, L1 - IS-IS level-1, L2 - IS-IS level-2, ia - IS-IS inter area
       * - candidate default, U - per-user static route, o - ODR
       P - periodic downloaded static route

Gateway of last resort is not set

     172.17.0.0/16 is variably subnetted, 2 subnets, 2 masks
C       172.17.25.8/30 is directly connected, Serial0/1/0
L       172.17.25.9/32 is directly connected, Serial0/1/0
     192.168.10.0/24 is variably subnetted, 2 subnets, 2 masks
C       192.168.10.0/24 is directly connected, GigabitEthernet0/0
L       192.168.10.254/32 is directly connected, GigabitEthernet0/0
R    192.168.20.0/24 [120/1] via 172.17.25.10, 00:00:17, Serial0/1/0
```

## Default Route
```
ip route 0.0.0.0 0.0.0.0 172.17.10.254
```

- Alle Pakete mit unbekannten Zielnetzwerk (0.0.0.0/0) an 172.17.10.254 weiterleiten
### Routing Tabelle anzeigen
```
r2#show ip route
Codes: L - local, C - connected, S - static, R - RIP, M - mobile, B - BGP
       D - EIGRP, EX - EIGRP external, O - OSPF, IA - OSPF inter area
       N1 - OSPF NSSA external type 1, N2 - OSPF NSSA external type 2
       E1 - OSPF external type 1, E2 - OSPF external type 2, E - EGP
       i - IS-IS, L1 - IS-IS level-1, L2 - IS-IS level-2, ia - IS-IS inter area
       * - candidate default, U - per-user static route, o - ODR
       P - periodic downloaded static route

Gateway of last resort is 172.17.10.254 to network 0.0.0.0

     172.17.0.0/16 is variably subnetted, 4 subnets, 3 masks
C       172.17.10.0/24 is directly connected, GigabitEthernet0/1
L       172.17.10.1/32 is directly connected, GigabitEthernet0/1
C       172.17.25.8/30 is directly connected, Serial0/1/0
L       172.17.25.10/32 is directly connected, Serial0/1/0
R    192.168.10.0/24 [120/1] via 172.17.25.9, 00:00:16, Serial0/1/0
     192.168.20.0/24 is variably subnetted, 2 subnets, 2 masks
C       192.168.20.0/24 is directly connected, GigabitEthernet0/0
L       192.168.20.254/32 is directly connected, GigabitEthernet0/0
S*   0.0.0.0/0 [1/0] via 172.17.10.254
```


```
r2(config)#router rip
r2(config-router)#default-information originate
```

-Dieser Befehl veranlasst den Router, eine **Standardroute** (0.0.0.0/0) **in den RIP-Aktualisierungen** **zu senden**. Dadurch können RIP-Router im Netzwerk wissen, dass dieser Router eine Route zum Internet oder zu einem anderen externen Netzwerk hat.

### Routing Tabelle anzeigen

```
r1#show ip route
Codes: L - local, C - connected, S - static, R - RIP, M - mobile, B - BGP
       D - EIGRP, EX - EIGRP external, O - OSPF, IA - OSPF inter area
       N1 - OSPF NSSA external type 1, N2 - OSPF NSSA external type 2
       E1 - OSPF external type 1, E2 - OSPF external type 2, E - EGP
       i - IS-IS, L1 - IS-IS level-1, L2 - IS-IS level-2, ia - IS-IS inter area
       * - candidate default, U - per-user static route, o - ODR
       P - periodic downloaded static route

Gateway of last resort is 172.17.25.10 to network 0.0.0.0

     172.17.0.0/16 is variably subnetted, 3 subnets, 3 masks
R       172.17.10.0/24 [120/1] via 172.17.25.10, 00:00:09, Serial0/1/0
C       172.17.25.8/30 is directly connected, Serial0/1/0
L       172.17.25.9/32 is directly connected, Serial0/1/0
     192.168.10.0/24 is variably subnetted, 2 subnets, 2 masks
C       192.168.10.0/24 is directly connected, GigabitEthernet0/0
L       192.168.10.254/32 is directly connected, GigabitEthernet0/0
R    192.168.20.0/24 [120/1] via 172.17.25.10, 00:00:09, Serial0/1/0
R*   0.0.0.0/0 [120/1] via 172.17.25.10, 00:00:09, Serial0/1/0
```


<hr>

# Access Control Listen konfigurieren
```
r2(config)#ip access-list standard 20
r2(config-std-nacl)#deny 192.168.10.0 0.0.0.255
r2(config-std-nacl)#permit any
r2(config-std-nacl)#exit
```

- `ip access-list standard 20`: Erstellt **Standard-Zugriffsliste** mit der Nummer 20.

- `deny 192.168.10.0 0.0.0.255`: **Verweigert** allen **Datenverkehr**, der von der IP-Adresse 192.168.10.0 stammt. "0.0.0.255" = Subnetz 192.168.10.0 **gesamt verweigert**.

- `permit any`: **Erlaubt** allen anderen **Datenverkehr**, der **nicht durch die vorherige Verweigerungsregel abgefangen** wurde. "Any" = **Alle Ip-Adressen**

## Access List anzeigen
```
r2#show ac
Standard IP access list 20
    10 deny 192.168.10.0 0.0.0.255
    20 permit any
```