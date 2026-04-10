# Routeur Internet — Configuration

Simule le routeur d'un FAI et le réseau WAN dans GNS3.  
Héberge les clients externes (SSH, OpenVPN, WEB/Conférence) et le serveur NTP.  
Applique le premier niveau de filtrage ACL avant le routeur NAT.

---

## Interfaces

| Interface | IP | Masque | Rôle |
|---|---|---|---|
| G1/0 | `200.0.0.1` | `/30` | Vers Routeur NAT (WAN) |
| Fa0/0 | `10.0.1.1` | `/30` | Vers client SSH |
| Fa0/1 | `10.0.2.1` | `/30` | Vers client OpenVPN |
| Fa0/2 | `10.0.3.1` | `/30` | Vers clients WEB & Conférence |
| Fa0/3 | `10.0.4.1` | `/30` | Vers serveur NTP |

## Route par défaut

```cisco
ip route 0.0.0.0 0.0.0.0 200.0.0.2
```

---

## Configuration des interfaces

```cisco
interface GigabitEthernet1/0
 ip address 200.0.0.1 255.255.255.252
 ip access-group ACL_WAN_IN in
 no shutdown
!
interface FastEthernet0/0
 ip address 10.0.1.1 255.255.255.252
 no shutdown
!
interface FastEthernet0/1
 ip address 10.0.2.1 255.255.255.252
 no shutdown
!
interface FastEthernet0/2
 ip address 10.0.3.1 255.255.255.252
 no shutdown
!
interface FastEthernet0/3
 ip address 10.0.4.1 255.255.255.252
 no shutdown
```

---

## ACL_WAN_IN — G1/0 (in)

Premier filtre appliqué sur le trafic entrant depuis le réseau interne vers Internet.

```cisco
ip access-list extended ACL_WAN_IN
 ! SSH — client dédié uniquement
 permit tcp 10.0.1.0 0.0.0.3 host 200.0.0.2 eq 22
 ! OpenVPN — client dédié uniquement
 permit udp 10.0.2.0 0.0.0.3 host 200.0.0.2 eq 1194
 ! WEB DMZ — HTTPS
 permit tcp 10.0.3.0 0.0.0.3 host 200.0.0.2 eq 443
 ! Conférence — HTTPS
 permit tcp 10.0.3.0 0.0.0.3 host 200.0.0.2 eq 8080
 ! Conférence — flux multimédia UDP
 permit udp 10.0.3.0 0.0.0.3 host 200.0.0.2 eq 10000
 ! NTP — diffusion vers tous
 permit udp host 10.0.4.1 any eq 123
 ! Tout le reste bloqué
 deny ip any any
```

---

## Clients WAN simulés

| Hôte | IP | Rôle |
|---|---|---|
| PC-PT Accès SSH | `10.0.1.2` | Test connexion SSH vers `192.168.20.10` |
| PC-PT Accès OpenVPN | `10.0.2.2` | Test tunnel OpenVPN vers `192.168.20.10` |
| PC-PT Utilisateur WEB & Conférence | `10.0.3.2` | Test accès HTTPS/8080/10000 vers DMZ |
| Serveur NTP | `10.0.4.1` | Synchronisation horaire de tous les équipements |

---

## Notes

- Ce routeur est uniquement une simulation — il n'existe pas dans un environnement réel
- Chaque client WAN dispose d'un sous-réseau `/30` dédié pour isoler les flux par service
- L'ACL `ACL_WAN_IN` bloque tout trafic non explicitement autorisé depuis le WAN vers le NAT
- Le serveur NTP `10.0.4.1` est accessible depuis l'ensemble du réseau interne via UDP/123
