# Switch L3 CœurReseau — Configuration

Switch de niveau 3 assurant le routage inter-VLAN et l'application des ACL entre les trois segments LAN.  
Point de convergence entre le Firewall et les switches d'accès SW1, SW2, SW3.

---

## Interfaces

| Interface | IP / VLAN | Rôle |
|---|---|---|
| G0/0 | `10.1.1.1 /30` | Vers Firewall |
| G0/1 | Trunk — VLAN 10 | Vers SW1 (Users) |
| G0/2 | Trunk — VLAN 20 | Vers SW2 (Info / SIEM) |
| G0/3 | Trunk — VLAN 30 | Vers SW3 (Serveur) |
| SVI Vlan10 | `192.168.10.254 /24` | Gateway VLAN 10 |
| SVI Vlan20 | `192.168.20.254 /24` | Gateway VLAN 20 |
| SVI Vlan30 | `192.168.30.254 /24` | Gateway VLAN 30 |

## Route par défaut

```cisco
ip route 0.0.0.0 0.0.0.0 10.1.1.2
```

---

## VLANs

```cisco
vlan 10
 name Users
!
vlan 20
 name Info
!
vlan 30
 name Serveur
```

---

## Configuration des interfaces

```cisco
! Interface vers Firewall
interface GigabitEthernet0/0
 ip address 10.1.1.1 255.255.255.252
 no shutdown
!
! Liens trunk vers switches d'accès
interface GigabitEthernet0/1
 switchport trunk encapsulation dot1q
 switchport mode trunk
 switchport trunk allowed vlan 10
 no shutdown
!
interface GigabitEthernet0/2
 switchport trunk encapsulation dot1q
 switchport mode trunk
 switchport trunk allowed vlan 20
 no shutdown
!
interface GigabitEthernet0/3
 switchport trunk encapsulation dot1q
 switchport mode trunk
 switchport trunk allowed vlan 30
 no shutdown
```

---

## SVI — Interfaces virtuelles (Gateway inter-VLAN)

```cisco
interface Vlan10
 ip address 192.168.10.254 255.255.255.0
 ip helper-address 192.168.30.110
 ip access-group ACL_LAN10_IN in
 no shutdown
!
interface Vlan20
 ip address 192.168.20.254 255.255.255.0
 ip helper-address 192.168.30.110
 ip access-group ACL_LAN20_IN in
 no shutdown
!
interface Vlan30
 ip address 192.168.30.254 255.255.255.0
 ip helper-address 192.168.30.110
 ip access-group ACL_LAN30_IN in
 no shutdown
```

---

## ACL_LAN10_IN — SVI Vlan10 (in)

Trafic provenant du réseau Users (`192.168.10.0/24`).

```cisco
ip access-list extended ACL_LAN10_IN
 ! LAN10 — accès total (LAN, DMZ, WAN)
 permit ip 192.168.10.0 0.0.0.255 any
 ! Tout le reste bloqué
 deny ip any any
```

---

## ACL_LAN20_IN — SVI Vlan20 (in)

Trafic provenant du réseau Info / SIEM (`192.168.20.0/24`).

```cisco
ip access-list extended ACL_LAN20_IN
 ! LAN20 → LAN10
 permit ip 192.168.20.0 0.0.0.255 192.168.10.0 0.0.0.255
 ! LAN20 → LAN30
 permit ip 192.168.20.0 0.0.0.255 192.168.30.0 0.0.0.255
 ! LAN20 → DMZ
 permit ip 192.168.20.0 0.0.0.255 172.16.0.0 0.0.255.255
 ! Logs → SIEM
 permit tcp 192.168.20.0 0.0.0.255 host 192.168.20.100 eq 5044
 ! LAN20 — pas d'accès WAN
 deny ip 192.168.20.0 0.0.0.255 any
```

---

## ACL_LAN30_IN — SVI Vlan30 (in)

Trafic provenant du réseau Serveur (`192.168.30.0/24`).

```cisco
ip access-list extended ACL_LAN30_IN
 ! LAN30 → LAN10
 permit ip 192.168.30.0 0.0.0.255 192.168.10.0 0.0.0.255
 ! LAN30 → LAN20
 permit ip 192.168.30.0 0.0.0.255 192.168.20.0 0.0.0.255
 ! Logs → SIEM
 permit tcp 192.168.30.0 0.0.0.255 host 192.168.20.100 eq 5044
 ! LAN30 — pas d'accès DMZ ni WAN
 deny ip any any
```

---

## Notes

- Le `ip helper-address 192.168.30.110` sur chaque SVI redirige les requêtes DHCP vers le serveur Kea DHCP (`192.168.30.110`)
- Chaque VLAN est strictement isolé — les flux inter-VLAN sont contrôlés par les ACL sur les SVI
- Le routage IP doit être activé sur le switch L3 : `ip routing`
