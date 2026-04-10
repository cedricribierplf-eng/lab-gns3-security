# SW1 — Configuration

Switch d'accès de niveau 2 associé au service **Users** (VLAN 10).  
Connecté au Switch L3 CœurReseau via un lien trunk.

---

## Interfaces

| Interface | Mode | VLAN | Rôle |
|---|---|---|---|
| G0/0 | Trunk | 10 | Vers SWL3 CœurReseau |
| G0/1 | Access | 10 | Vers PC utilisateur 1 |
| G0/2 | Access | 10 | Vers PC utilisateur 2 |

---

## Configuration

```cisco
! Lien trunk vers CœurReseau
interface GigabitEthernet0/0
 switchport trunk encapsulation dot1q
 switchport mode trunk
 switchport trunk allowed vlan 10
 no shutdown
!
! Ports d'accès clients
interface GigabitEthernet0/1
 switchport mode access
 switchport access vlan 10
 no shutdown
!
interface GigabitEthernet0/2
 switchport mode access
 switchport access vlan 10
 no shutdown
```

---

## Réseau

| Hôte | IP | Rôle |
|---|---|---|
| PC utilisateur 1 | `192.168.10.1` | Poste client VLAN 10 |
| PC utilisateur 2 | `192.168.10.2` | Poste client VLAN 10 |
| Gateway | `192.168.10.254` | SVI Vlan10 — CœurReseau |

---

## Notes

- VLAN 10 uniquement autorisé sur ce switch
- Les postes reçoivent leur adresse IP via DHCP (Kea — `192.168.30.110`)
- Accès total : LAN, DMZ et WAN autorisés via les ACL du CœurReseau
