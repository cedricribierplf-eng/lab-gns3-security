# Plan d'adressage IP

Plan d'adressage complet du lab — toutes zones confondues.

---

## WAN — Simulation Internet

### Liens point-à-point

| Réseau | Équipement A | IP A | Équipement B | IP B |
|---|---|---|---|---|
| `200.0.0.0/30` | Routeur Internet | `200.0.0.1` | Routeur NAT | `200.0.0.2` |
| `10.1.2.0/30` | Routeur NAT | `10.1.2.2` | Firewall | `10.1.2.1` |
| `10.1.1.0/30` | Firewall | `10.1.1.2` | SWL3 CœurReseau | `10.1.1.1` |

### Clients WAN simulés

| Réseau | Routeur Internet | Client / Serveur | IP Client | Rôle |
|---|---|---|---|---|
| `10.0.1.0/30` | `10.0.1.1` | PC Accès SSH | `10.0.1.2` | Test SSH vers `192.168.20.10` |
| `10.0.2.0/30` | `10.0.2.1` | PC Accès OpenVPN | `10.0.2.2` | Test VPN vers `192.168.20.10` |
| `10.0.3.0/30` | `10.0.3.1` | PC WEB & Conférence | `10.0.3.2` | Test HTTPS/8080/10000 vers DMZ |
| `10.0.4.0/30` | `10.0.4.1` | Serveur NTP | `10.0.4.1` | Synchronisation horaire globale |

---

## DMZ — `172.16.0.0/24`

| Hôte | IP | OS | Rôle | Ports |
|---|---|---|---|---|
| Gateway (Firewall) | `172.16.0.254` | Cisco IOS | Passerelle DMZ | — |
| Serveur DNS | `172.16.0.53` | Debian Trixie 13.6 | PowerDNS — résolution interne | UDP/53 — TCP/53 |
| Serveur WEB | `172.16.0.25` | Debian Trixie 13.6 | HTTP/HTTPS exposé WAN | TCP/80 — TCP/443 |
| Serveur Conférence | `172.16.0.63` | Debian Trixie 13.6 | HTTPS + flux multimédia | TCP/8080 — UDP/10000 |

---

## LAN — Réseaux internes

### VLAN 10 — Users (`192.168.10.0/24`)

| Hôte | IP | Rôle |
|---|---|---|
| Gateway (SVI CœurReseau) | `192.168.10.254` | Passerelle VLAN 10 |
| PC utilisateur 1 | `192.168.10.1` | Poste client |
| PC utilisateur 2 | `192.168.10.2` | Poste client |
| Plage DHCP | `192.168.10.10 – 192.168.10.200` | Attribution dynamique |

### VLAN 20 — Info / SIEM (`192.168.20.0/24`)

| Hôte | IP | OS | Rôle | Ports |
|---|---|---|---|---|
| Gateway (SVI CœurReseau) | `192.168.20.254` | Cisco IOS | Passerelle VLAN 20 | — |
| SIEM Wazuh | `192.168.20.100` | Debian Trixie 13.6 | Collecte logs + dashboard | TCP/5044 — TCP/5601 — HTTPS/443 |
| Serveur SSH / OpenVPN | `192.168.20.10` | Debian Trixie 13.6 | Accès distant sécurisé | TCP/22 — UDP/1194 |
| PC Info | `192.168.20.X` | — | Poste administration | — |
| Plage DHCP | `192.168.20.10 – 192.168.20.200` | — | Attribution dynamique | — |

### VLAN 30 — Serveur (`192.168.30.0/24`)

| Hôte | IP | OS | Rôle | Ports |
|---|---|---|---|---|
| Gateway (SVI CœurReseau) | `192.168.30.254` | Cisco IOS | Passerelle VLAN 30 | — |
| Serveur DHCP (Kea) | `192.168.30.110` | Debian Trixie 13.6 | Distribution DHCP VLAN 10/20/30 | UDP/67 — UDP/68 |
| Serveur WEB interne | `192.168.30.150` | Debian Trixie 13.6 | WEB accessible LAN uniquement | TCP/80 — TCP/443 |

---

## Récapitulatif des zones

| Zone | Réseau | Gateway | Accès autorisé |
|---|---|---|---|
| WAN | `200.0.0.0/30` | — | Internet |
| DMZ | `172.16.0.0/24` | `172.16.0.254` | WAN (ports spécifiques) — SIEM |
| VLAN 10 Users | `192.168.10.0/24` | `192.168.10.254` | LAN + DMZ + WAN |
| VLAN 20 Info | `192.168.20.0/24` | `192.168.20.254` | LAN + DMZ — WAN bloqué |
| VLAN 30 Serveur | `192.168.30.0/24` | `192.168.30.254` | LAN uniquement |

---

> Dernière mise à jour : avril 2026
