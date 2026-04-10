# SW2 — Configuration

Switch d'accès de niveau 2 associé au service **Info / SIEM** (VLAN 20).  
Héberge le SIEM Wazuh et le serveur SSH/OpenVPN.  
Connecté au Switch L3 CœurReseau via un lien trunk.

---

## Interfaces

| Interface | Mode | VLAN | Rôle |
|---|---|---|---|
| G0/0 | Trunk | 20 | Vers SWL3 CœurReseau |
| G0/1 | Access | 20 | Vers SIEM Wazuh |
| G0/2 | Access | 20 | Vers serveur SSH / OpenVPN |
| G0/3 | Access | 20 | Vers PC Info |

---

## Configuration

```cisco
! Lien trunk vers CœurReseau
interface GigabitEthernet0/0
 switchport trunk encapsulation dot1q
 switchport mode trunk
 switchport trunk allowed vlan 20
 no shutdown
!
! Port SIEM Wazuh
interface GigabitEthernet0/1
 switchport mode access
 switchport access vlan 20
 no shutdown
!
! Port serveur SSH / OpenVPN
interface GigabitEthernet0/2
 switchport mode access
 switchport access vlan 20
 no shutdown
!
! Port PC Info
interface GigabitEthernet0/3
 switchport mode access
 switchport access vlan 20
 no shutdown
```

---

## Réseau

| Hôte | IP | Rôle |
|---|---|---|
| SIEM Wazuh | `192.168.20.100` | Collecte logs (TCP/5044) — Dashboard (TCP/5601) |
| Serveur SSH / OpenVPN | `192.168.20.10` | Accès distant sécurisé depuis WAN |
| PC Info | `192.168.20.X` | Poste administration |
| Gateway | `192.168.20.254` | SVI Vlan20 — CœurReseau |

---

## Notes

- VLAN 20 uniquement autorisé sur ce switch
- Le réseau `192.168.20.0/24` est le seul autorisé à administrer le Firewall (HTTPS/SSH)
- Pas d'accès WAN depuis ce VLAN — uniquement LAN et DMZ
- Le SIEM reçoit les logs de tous les équipements LAN et DMZ sur TCP/5044
