# Politique de sécurité — ACL

Document de référence décrivant les règles de filtrage appliquées dans le lab, leur localisation et leur justification.

---

## Principes appliqués

- **Moindre privilège** — chaque zone n'accède qu'aux ressources strictement nécessaires
- **Deny all implicite** — tout trafic non explicitement autorisé est bloqué
- **Défense en profondeur** — les ACL sont appliquées sur plusieurs équipements (routeur, firewall, switch L3)
- **Séparation des flux** — les logs, l'administration et le trafic utilisateur empruntent des chemins distincts

---

## Matrice de flux

| Source | Destination | Protocole / Port | Autorisé | Règle |
|---|---|---|---|---|
| LAN 10 `192.168.10.0/24` | LAN, DMZ, WAN | Tout | ✅ | #1 |
| LAN 20 `192.168.20.0/24` | LAN, DMZ | Tout | ✅ | #2 |
| LAN 20 `192.168.20.0/24` | WAN | Tout | ❌ | #2 |
| LAN 30 `192.168.30.0/24` | LAN | Tout | ✅ | #3 |
| LAN 30 `192.168.30.0/24` | DMZ, WAN | Tout | ❌ | #3 |
| DMZ `172.16.0.0/24` | WAN | Tout | ✅ | #4 |
| DMZ `172.16.0.0/24` | LAN | Tout | ❌ | #4 |
| Tous | SIEM `192.168.20.100` | TCP/5044 | ✅ | #5 |
| Tous | SIEM `192.168.20.100` | TCP/5601 | ✅ | #5 |
| WAN | WEB DMZ `172.16.0.25` | TCP/443 — TCP/80 | ✅ | #6 |
| WAN | Conférence `172.16.0.63` | TCP/8080 — UDP/10000 | ✅ | #6 |
| Client SSH `10.0.1.0/30` | SSH `192.168.20.10` | TCP/22 | ✅ | #7 |
| Client OpenVPN `10.0.2.0/30` | OpenVPN `192.168.20.10` | UDP/1194 | ✅ | #8 |
| Tous | NTP `10.0.4.1` | UDP/123 | ✅ | #9 |
| LAN 20 `192.168.20.0/24` | Firewall `10.1.2.1` | TCP/443 — TCP/22 | ✅ | #10 |

---

## Règles détaillées

### #1 — LAN 10 accès total
Le réseau Users est le moins restreint. Il accède aux autres LANs, à la DMZ et au WAN.  
**Justification** : réseau des postes utilisateurs standards nécessitant un accès complet aux services internes et Internet.

### #2 — LAN 20 accès LAN et DMZ uniquement
Le réseau Info héberge le SIEM et le serveur SSH/OpenVPN. L'accès WAN direct est bloqué.  
**Justification** : les serveurs d'administration ne doivent pas initier de connexions sortantes vers Internet. L'accès entrant depuis le WAN est géré par des règles spécifiques (SSH, OpenVPN).

### #3 — LAN 30 accès inter-LAN uniquement
Le réseau Serveur héberge le DHCP et le WEB interne. Aucun accès DMZ ni WAN.  
**Justification** : les serveurs d'infrastructure internes n'ont pas vocation à communiquer avec l'extérieur.

### #4 — DMZ accès WAN, pas LAN
Les serveurs DMZ peuvent sortir vers Internet (mises à jour, NTP) mais ne peuvent pas initier de connexions vers le LAN.  
**Justification** : si un serveur DMZ est compromis, l'attaquant ne doit pas pouvoir atteindre le réseau interne.

### #5 — Logs vers SIEM
Tous les équipements (LAN et DMZ) envoient leurs logs au SIEM Wazuh sur TCP/5044.  
Le dashboard Wazuh est accessible sur TCP/5601 depuis le LAN.  
**Justification** : visibilité complète sur l'activité réseau pour la détection d'incidents.

### #6 — Services WEB et Conférence accessibles depuis WAN
Le serveur WEB (`172.16.0.25`) et le serveur Conférence (`172.16.0.63`) sont exposés sur Internet.  
Ports autorisés : TCP/443, TCP/80, TCP/8080, UDP/10000.  
**Justification** : ces services sont destinés aux utilisateurs externes.

### #7 — Accès SSH depuis WAN
Seul le sous-réseau `10.0.1.0/30` est autorisé à se connecter en SSH au serveur `192.168.20.10`.  
**Justification** : restriction de l'accès SSH à un client WAN identifié — évite les tentatives de brute-force depuis tout Internet.

### #8 — Accès OpenVPN depuis WAN
Seul le sous-réseau `10.0.2.0/30` est autorisé à établir un tunnel OpenVPN vers `192.168.20.10` sur UDP/1194.  
**Justification** : même principe que SSH — accès restreint à un client dédié.

### #9 — Synchronisation NTP
Tous les équipements du réseau synchronisent leur horloge via le serveur NTP `10.0.4.1` sur UDP/123.  
**Justification** : la cohérence horaire est indispensable pour la corrélation des logs dans Wazuh.

### #10 — Administration du Firewall
Seul le réseau LAN 20 (`192.168.20.0/24`) peut administrer le Firewall via HTTPS (TCP/443) et SSH (TCP/22).  
**Justification** : l'administration réseau est centralisée sur le VLAN dédié à l'équipe Info/Sécu.

---

## Localisation des ACL

| ACL | Équipement | Interface | Direction | Filtre |
|---|---|---|---|---|
| `ACL_WAN_IN` | Routeur Internet | G1/0 | in | Trafic WAN → NAT |
| `ACL-NAT-OUTSIDE` | Routeur NAT | G2/0 | in | Trafic Internet → interne |
| `ACL-NAT-INSIDE` | Routeur NAT | Fa0/0 | in | Trafic interne → WAN |
| `ACL-FROM-LAN` | Firewall | G1/0 | in | Trafic LAN → FW |
| `ACL-FROM-DMZ` | Firewall | G2/0 | in | Trafic DMZ → FW |
| `ACL_LAN10_IN` | SWL3 CœurReseau | SVI Vlan10 | in | Trafic VLAN 10 |
| `ACL_LAN20_IN` | SWL3 CœurReseau | SVI Vlan20 | in | Trafic VLAN 20 |
| `ACL_LAN30_IN` | SWL3 CœurReseau | SVI Vlan30 | in | Trafic VLAN 30 |

---

> Les configurations complètes de chaque ACL sont disponibles dans [`/configs/`](../configs/)  
> Dernière mise à jour : avril 2026
