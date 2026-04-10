# Documentation

Index de la documentation technique du lab. Ce dossier centralise le plan d'adressage, la politique de sécurité et les schémas de topologie.

---

## Contenu

| Fichier | Description |
|---|---|
| [`plan-adressage.md`](./plan-adressage.md) | Plan d'adressage IP complet — toutes zones |
| [`politique-acl.md`](./politique-acl.md) | Politique de sécurité — règles et justifications |
| [`topologie.png`](./topologie.png) | Schéma de topologie exporté depuis GNS3 |

---

## Organisation du lab

Le lab est structuré en 4 zones réseau distinctes :

| Zone | Réseau | Rôle |
|---|---|---|
| **WAN** | `200.0.0.0/30` — `10.0.X.X/30` | Simulation Internet et clients externes |
| **DMZ** | `172.16.0.0/24` | Services exposés — DNS, WEB, Conférence |
| **LAN** | `192.168.X.0/24` | Réseaux internes segmentés par VLAN |
| **Transit** | `10.1.X.0/30` | Liens point-à-point entre équipements |

---

## Phases du projet

| Phase | Description | Statut |
|---|---|---|
| 1 | Conception topologie — Packet Tracer | ✅ Terminé |
| 2 | Implémentation réseau GNS3 — routage, VLANs, trunks, SVI | ✅ Terminé |
| 3 | Rédaction et application des ACL | ✅ Terminé |
| 4 | Déploiement services Linux sur VM Debian Trixie | 🔄 En cours |
| 5 | Intégration Wazuh + collecte de logs | ⏳ À venir |
| 6 | Tests ACL, simulation d'attaques, analyse alertes | ⏳ À venir |
| 7 | Documentation finale et conclusions | ⏳ À venir |

---

## Services prévus

| Service | Technologie | IP | Zone |
|---|---|---|---|
| SIEM | Wazuh | `192.168.20.100` | LAN 20 |
| SSH / OpenVPN | Debian Trixie | `192.168.20.10` | LAN 20 |
| DNS | PowerDNS | `172.16.0.53` | DMZ |
| WEB DMZ | Debian Trixie | `172.16.0.25` | DMZ |
| Conférence | Debian Trixie | `172.16.0.63` | DMZ |
| DHCP | Kea | `192.168.30.110` | LAN 30 |
| WEB interne | Debian Trixie | `192.168.30.150` | LAN 30 |
| NTP | Simulé WAN | `10.0.4.1` | WAN |

---

> Dernière mise à jour : avril 2026
