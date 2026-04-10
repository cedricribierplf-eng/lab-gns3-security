# Configurations réseau

Index des configurations de tous les équipements du lab. Chaque fichier contient les interfaces, routes statiques et ACL appliquées.

---

## Équipements

| Fichier | Équipement | Rôle | IP(s) |
|---|---|---|---|
| [`routeur-internet.md`](./routeur-internet.md) | Routeur Internet | Simulation WAN + clients externes | `200.0.0.1` / `10.0.X.X` |
| [`routeur-nat.md`](./routeur-nat.md) | Routeur NAT | Translation d'adresses WAN↔LAN | `200.0.0.2` / `10.1.2.2` |
| [`firewall.md`](./firewall.md) | Firewall | Filtrage LAN / DMZ / WAN | `10.1.1.2` / `10.1.2.1` / `172.16.0.254` |
| [`coeurreseau-swl3.md`](./coeurreseau-swl3.md) | Switch L3 CœurReseau | Routage inter-VLAN + ACL inter-LAN | `10.1.1.1` / SVI `.254` |
| [`sw1.md`](./sw1.md) | SW1 | Accès VLAN 10 — Users | `192.168.10.0/24` |
| [`sw2.md`](./sw2.md) | SW2 | Accès VLAN 20 — Info / SIEM | `192.168.20.0/24` |
| [`sw3.md`](./sw3.md) | SW3 | Accès VLAN 30 — Serveur | `192.168.30.0/24` |

---

## Topologie résumée

```
[WAN]
  └── Routeur Internet (200.0.0.1)
        └── Routeur NAT (200.0.0.2 / 10.1.2.2)
              └── Firewall (10.1.2.1 / 10.1.1.2 / 172.16.0.254)
                    ├── DMZ (172.16.0.0/24)
                    │     ├── DNS       172.16.0.53
                    │     ├── WEB       172.16.0.25
                    │     └── Conférence 172.16.0.63
                    └── SWL3 CœurReseau (10.1.1.1)
                          ├── SW1 — VLAN 10 — Users    192.168.10.0/24
                          ├── SW2 — VLAN 20 — Info     192.168.20.0/24
                          └── SW3 — VLAN 30 — Serveur  192.168.30.0/24
```

---

## ACL — vue d'ensemble

| Équipement | ACL | Interface | Direction |
|---|---|---|---|
| Routeur Internet | `ACL_WAN_IN` | G1/0 | in |
| Routeur NAT | `ACL-NAT-OUTSIDE` | G2/0 | in |
| Routeur NAT | `ACL-NAT-INSIDE` | Fa0/0 | in |
| Firewall | `ACL-FROM-LAN` | G1/0 | in |
| Firewall | `ACL-FROM-DMZ` | G2/0 | in |
| SWL3 CœurReseau | `ACL_LAN10_IN` | G0/1 | in |
| SWL3 CœurReseau | `ACL_LAN20_IN` | G0/2 | in |
| SWL3 CœurReseau | `ACL_LAN30_IN` | G0/3 | in |

---

> Les configurations sont extraites du lab GNS3 en cours de déploiement.  
> Dernière mise à jour : avril 2026
