# Services

Index des services Linux déployés sur VM Debian Trixie 13.6 et intégrés dans GNS3.

---

## Contenu

| Dossier | Service | Technologie | IP | Statut |
|---|---|---|---|---|
| [`wazuh/`](./wazuh/) | SIEM | Wazuh | `192.168.20.100` | ⏳ Phase 5 |
| [`openvpn/`](./openvpn/) | VPN + SSH | OpenVPN / OpenSSH | `192.168.20.10` | ⏳ Phase 4 |
| [`dns-powerdns/`](./dns-powerdns/) | DNS | PowerDNS | `172.16.0.53` | ⏳ Phase 4 |
| [`web/`](./web/) | Serveurs WEB | Apache / Nginx | `172.16.0.25` — `192.168.30.150` | ⏳ Phase 4 |
| [`dhcp-kea/`](./dhcp-kea/) | DHCP | Kea DHCP | `192.168.30.110` | ⏳ Phase 4 |

---

## Infrastructure commune

Tous les serveurs tournent sur **Debian Trixie 13.6** et partagent une configuration de base commune :

- Mise à jour système — `apt update && apt upgrade`
- Désactivation des services inutiles
- Configuration SSH — port 22, authentification par clé
- Agent Wazuh installé sur chaque VM pour envoi des logs au SIEM (`192.168.20.100:5044`)
- Synchronisation NTP via `10.0.4.1`

---

## Ordre de déploiement recommandé

```
1. DNS (PowerDNS)       — requis par les autres services pour la résolution de noms
2. DHCP (Kea)           — distribution d'adresses aux postes clients
3. WEB                  — serveurs HTTP/HTTPS DMZ et LAN
4. SSH / OpenVPN        — accès distant sécurisé
5. Wazuh SIEM           — collecte des logs de tous les services précédents
```

---

## Structure type de chaque dossier service

```
services/<service>/
├── README.md          ← installation, configuration, intégration GNS3
├── config/            ← fichiers de configuration du service
└── screenshots/       ← captures de validation
```

---

> Les dossiers seront complétés au fur et à mesure du déploiement — Phase 4 et 5.  
> Dernière mise à jour : avril 2026
