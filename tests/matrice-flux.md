# Matrice de flux — Tests ACL

Document de validation des politiques de filtrage.  
Chaque flux est testé depuis les postes clients simulés dans GNS3 et le résultat consigné ici.

---

## Légende

| Symbole | Signification |
|---|---|
| ✅ | Flux autorisé — testé et fonctionnel |
| ❌ | Flux bloqué — testé et conforme |
| ⏳ | Non encore testé |
| ⚠️ | Résultat inattendu — à investiguer |

---

## Flux inter-LAN

| Source | Destination | Protocole | Attendu | Résultat | Commande de test |
|---|---|---|---|---|---|
| LAN 10 `192.168.10.1` | LAN 20 `192.168.20.1` | ICMP | ✅ | ⏳ | `ping 192.168.20.1` |
| LAN 10 `192.168.10.1` | LAN 30 `192.168.30.1` | ICMP | ✅ | ⏳ | `ping 192.168.30.1` |
| LAN 20 `192.168.20.1` | LAN 10 `192.168.10.1` | ICMP | ✅ | ⏳ | `ping 192.168.10.1` |
| LAN 20 `192.168.20.1` | LAN 30 `192.168.30.1` | ICMP | ✅ | ⏳ | `ping 192.168.30.1` |
| LAN 30 `192.168.30.1` | LAN 10 `192.168.10.1` | ICMP | ✅ | ⏳ | `ping 192.168.10.1` |
| LAN 30 `192.168.30.1` | LAN 20 `192.168.20.1` | ICMP | ✅ | ⏳ | `ping 192.168.20.1` |

---

## Flux LAN → DMZ

| Source | Destination | Protocole | Attendu | Résultat | Commande de test |
|---|---|---|---|---|---|
| LAN 10 `192.168.10.1` | DNS `172.16.0.53` | UDP/53 | ✅ | ⏳ | `nslookup domain 172.16.0.53` |
| LAN 10 `192.168.10.1` | WEB `172.16.0.25` | TCP/443 | ✅ | ⏳ | `curl https://172.16.0.25` |
| LAN 20 `192.168.20.1` | WEB `172.16.0.25` | TCP/443 | ✅ | ⏳ | `curl https://172.16.0.25` |
| LAN 30 `192.168.30.1` | WEB `172.16.0.25` | TCP/443 | ❌ | ⏳ | `curl https://172.16.0.25` |
| LAN 30 `192.168.30.1` | DNS `172.16.0.53` | UDP/53 | ❌ | ⏳ | `nslookup domain 172.16.0.53` |

---

## Flux LAN → WAN

| Source | Destination | Protocole | Attendu | Résultat | Commande de test |
|---|---|---|---|---|---|
| LAN 10 `192.168.10.1` | WAN `200.0.0.1` | ICMP | ✅ | ⏳ | `ping 200.0.0.1` |
| LAN 20 `192.168.20.1` | WAN `200.0.0.1` | ICMP | ❌ | ⏳ | `ping 200.0.0.1` |
| LAN 30 `192.168.30.1` | WAN `200.0.0.1` | ICMP | ❌ | ⏳ | `ping 200.0.0.1` |
| DMZ `172.16.0.25` | WAN `200.0.0.1` | ICMP | ✅ | ⏳ | `ping 200.0.0.1` |

---

## Flux WAN → DMZ

| Source | Destination | Protocole / Port | Attendu | Résultat | Commande de test |
|---|---|---|---|---|---|
| Client WEB `10.0.3.2` | WEB `172.16.0.25` | TCP/443 | ✅ | ⏳ | `curl https://172.16.0.25` |
| Client WEB `10.0.3.2` | WEB `172.16.0.25` | TCP/80 | ✅ | ⏳ | `curl http://172.16.0.25` |
| Client WEB `10.0.3.2` | Conférence `172.16.0.63` | TCP/8080 | ✅ | ⏳ | `curl https://172.16.0.63:8080` |
| Client WEB `10.0.3.2` | Conférence `172.16.0.63` | UDP/10000 | ✅ | ⏳ | `nc -u 172.16.0.63 10000` |
| Client WEB `10.0.3.2` | LAN 10 `192.168.10.1` | ICMP | ❌ | ⏳ | `ping 192.168.10.1` |

---

## Flux WAN → Services d'accès distant

| Source | Destination | Protocole / Port | Attendu | Résultat | Commande de test |
|---|---|---|---|---|---|
| Client SSH `10.0.1.2` | SSH `192.168.20.10` | TCP/22 | ✅ | ⏳ | `ssh user@192.168.20.10` |
| Client OpenVPN `10.0.2.2` | OpenVPN `192.168.20.10` | UDP/1194 | ✅ | ⏳ | Connexion client OpenVPN |
| Client WEB `10.0.3.2` | SSH `192.168.20.10` | TCP/22 | ❌ | ⏳ | `ssh user@192.168.20.10` |

---

## Flux → SIEM

| Source | Destination | Protocole / Port | Attendu | Résultat | Commande de test |
|---|---|---|---|---|---|
| LAN 10 `192.168.10.1` | SIEM `192.168.20.100` | TCP/5044 | ✅ | ⏳ | `nc -zv 192.168.20.100 5044` |
| LAN 20 `192.168.20.1` | SIEM `192.168.20.100` | TCP/5044 | ✅ | ⏳ | `nc -zv 192.168.20.100 5044` |
| LAN 30 `192.168.30.1` | SIEM `192.168.20.100` | TCP/5044 | ✅ | ⏳ | `nc -zv 192.168.20.100 5044` |
| DMZ `172.16.0.25` | SIEM `192.168.20.100` | TCP/5044 | ✅ | ⏳ | `nc -zv 192.168.20.100 5044` |
| LAN 20 `192.168.20.1` | Dashboard `192.168.20.100` | TCP/5601 | ✅ | ⏳ | `curl http://192.168.20.100:5601` |

---

## Flux NTP

| Source | Destination | Protocole / Port | Attendu | Résultat | Commande de test |
|---|---|---|---|---|---|
| LAN 10 `192.168.10.1` | NTP `10.0.4.1` | UDP/123 | ✅ | ⏳ | `ntpdate -q 10.0.4.1` |
| LAN 20 `192.168.20.1` | NTP `10.0.4.1` | UDP/123 | ✅ | ⏳ | `ntpdate -q 10.0.4.1` |
| LAN 30 `192.168.30.1` | NTP `10.0.4.1` | UDP/123 | ✅ | ⏳ | `ntpdate -q 10.0.4.1` |
| DMZ `172.16.0.25` | NTP `10.0.4.1` | UDP/123 | ✅ | ⏳ | `ntpdate -q 10.0.4.1` |

---

## Administration Firewall

| Source | Destination | Protocole / Port | Attendu | Résultat | Commande de test |
|---|---|---|---|---|---|
| LAN 20 `192.168.20.1` | Firewall `10.1.2.1` | TCP/443 | ✅ | ⏳ | `curl https://10.1.2.1` |
| LAN 20 `192.168.20.1` | Firewall `10.1.2.1` | TCP/22 | ✅ | ⏳ | `ssh admin@10.1.2.1` |
| LAN 10 `192.168.10.1` | Firewall `10.1.2.1` | TCP/22 | ❌ | ⏳ | `ssh admin@10.1.2.1` |
| LAN 30 `192.168.30.1` | Firewall `10.1.2.1` | TCP/22 | ❌ | ⏳ | `ssh admin@10.1.2.1` |

---

## Simulation d'attaque — Phase 6

> Section à compléter lors de la Phase 6 du projet.

| Scénario | Vecteur | Détection Wazuh | Résultat |
|---|---|---|---|
| Scan de ports depuis WAN | `nmap -sS 200.0.0.2` | ⏳ | ⏳ |
| Brute-force SSH depuis mauvais client | `hydra` depuis `10.0.3.2` | ⏳ | ⏳ |
| Tentative de pivot DMZ → LAN | Shell depuis `172.16.0.25` | ⏳ | ⏳ |
| Flood ICMP inter-VLAN | `ping -f` depuis LAN 30 | ⏳ | ⏳ |

---

> Les résultats seront mis à jour au fur et à mesure des phases de test.  
> Dernière mise à jour : avril 2026
