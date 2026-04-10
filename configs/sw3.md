# SW3 — Configuration

Switch d'accès de niveau 2 associé au service **Serveur** (VLAN 30).  
Héberge le serveur DHCP (Kea) et le serveur WEB interne.  
Connecté au Switch L3 CœurReseau via un lien trunk.

---

## Interfaces

| Interface | Mode | VLAN | Rôle |
|---|---|---|---|
| G0/0 | Trunk | 30 | Vers SWL3 CœurReseau |
| G0/1 | Access | 30 | Vers serveur DHCP (Kea) |
| G0/2 | Access | 30 | Vers serveur WEB interne |
| G0/3 | Access | 30 | Vers serveur additionnel |

---

## Configuration

```cisco
! Lien trunk vers CœurReseau
interface GigabitEthernet0/0
 switchport trunk encapsulation dot1q
 switchport mode trunk
 switchport trunk allowed vlan 30
 no shutdown
!
! Port serveur DHCP
interface GigabitEthernet0/1
 switchport mode access
 switchport access vlan 30
 no shutdown
!
! Port serveur WEB interne
interface GigabitEthernet0/2
 switchport mode access
 switchport access vlan 30
 no shutdown
!
! Port serveur additionnel
interface GigabitEthernet0/3
 switchport mode access
 switchport access vlan 30
 no shutdown
```

---

## Réseau

| Hôte | IP | Rôle |
|---|---|---|
| Serveur DHCP (Kea) | `192.168.30.110` | Distribution d'adresses pour VLAN 10, 20, 30 |
| Serveur WEB interne | `192.168.30.150` | Serveur web accessible depuis LAN uniquement |
| Gateway | `192.168.30.254` | SVI Vlan30 — CœurReseau |

---

## Notes

- VLAN 30 uniquement autorisé sur ce switch
- Accès restreint : inter-LAN uniquement — pas d'accès DMZ ni WAN
- Le serveur Kea DHCP répond aux requêtes relayées par le `ip helper-address` configuré sur chaque SVI du CœurReseau
- Le serveur WEB interne (`192.168.30.150`) n'est pas exposé sur le WAN — distinct du WEB DMZ (`172.16.0.25`)
