# Firewall — Configuration

Routeur Cisco faisant office de pare-feu entre le LAN, la DMZ et le WAN.  
Applique les ACL de filtrage sur les interfaces LAN et DMZ.

---

## Interfaces

| Interface | IP | Masque | Rôle |
|---|---|---|---|
| G1/0 | `10.1.1.2` | `/30` | Vers SWL3 CœurReseau (LAN) |
| G2/0 | `172.16.0.254` | `/24` | Vers DMZ |
| G3/0 | `10.1.2.1` | `/30` | Vers Routeur NAT (WAN) |

## Route par défaut

```cisco
ip route 0.0.0.0 0.0.0.0 10.1.2.2
```

---

## Configuration des interfaces

```cisco
interface GigabitEthernet1/0
 ip address 10.1.1.2 255.255.255.252
 ip access-group ACL-FROM-LAN in
 no shutdown
!
interface GigabitEthernet2/0
 ip address 172.16.0.254 255.255.255.0
 ip access-group ACL-FROM-DMZ in
 no shutdown
!
interface GigabitEthernet3/0
 ip address 10.1.2.1 255.255.255.252
 no shutdown
```

---

## ACL-FROM-LAN — G1/0 (in)

Filtre le trafic provenant du réseau LAN (via SWL3 CœurReseau).

```cisco
ip access-list extended ACL-FROM-LAN
 ! Logs → SIEM (tous les équipements LAN et DMZ)
 permit tcp any host 192.168.20.100 eq 5044
 ! Dashboard SIEM
 permit tcp any host 192.168.20.100 eq 5601
 ! Administration du firewall — LAN20 uniquement
 permit tcp 192.168.20.0 0.0.0.255 host 10.1.2.1 eq 443
 permit tcp 192.168.20.0 0.0.0.255 host 10.1.2.1 eq 22
 ! LAN10 → DMZ
 permit ip 192.168.10.0 0.0.0.255 172.16.0.0 0.0.255.255
 ! LAN20 → DMZ
 permit ip 192.168.20.0 0.0.0.255 172.16.0.0 0.0.255.255
 ! LAN10 → WAN
 permit ip 192.168.10.0 0.0.0.255 any
 ! LAN20 — pas d'accès WAN
 deny ip 192.168.20.0 0.0.0.255 any
 ! LAN30 — bloqué hors inter-LAN
 deny ip 192.168.30.0 0.0.0.255 any
 ! Tout le reste bloqué
 deny ip any any
```

---

## ACL-FROM-DMZ — G2/0 (in)

Filtre le trafic provenant de la DMZ vers le LAN ou le WAN.

```cisco
ip access-list extended ACL-FROM-DMZ
 ! Logs DMZ → SIEM
 permit tcp 172.16.0.0 0.0.255.255 host 192.168.20.100 eq 5044
 ! NTP — synchronisation horaire
 permit udp 172.16.0.0 0.0.255.255 any eq 123
 ! DMZ ne peut pas accéder aux LANs
 deny ip 172.16.0.0 0.0.255.255 192.168.0.0 0.0.255.255
 ! Tout le reste bloqué
 deny ip any any
```

---

## Notes

- Le LAN20 (`192.168.20.0/24`) est le seul réseau autorisé à administrer le pare-feu (HTTPS + SSH)
- La DMZ peut uniquement envoyer ses logs au SIEM et synchroniser l'heure via NTP
- Aucun flux initié depuis la DMZ vers le LAN n'est autorisé
- Le trafic retour (established) est géré par le routeur NAT en amont
