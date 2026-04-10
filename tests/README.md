# Tests

Index des tests de validation du lab — ACL, connectivité et simulation d'attaques.

---

## Contenu

| Fichier | Description |
|---|---|
| [`matrice-flux.md`](./matrice-flux.md) | Validation de tous les flux réseau par zone |
| [`logs-wazuh/`](./logs-wazuh/) | Captures de logs et alertes Wazuh *(Phase 6)* |

---

## Phases de test

| Phase | Description | Statut |
|---|---|---|
| **Phase 6a** | Tests ACL — validation de la matrice de flux | ⏳ À venir |
| **Phase 6b** | Simulation d'attaques — scan, brute-force, pivot | ⏳ À venir |
| **Phase 6c** | Analyse des alertes Wazuh | ⏳ À venir |

---

## Méthodologie

Les tests sont effectués depuis les postes clients simulés dans GNS3 :

- **Connectivité** — `ping`, `traceroute` pour valider le routage
- **Ports** — `nc -zv` pour vérifier l'accessibilité d'un service
- **Services** — `curl`, `ssh`, `nslookup` pour tester les applications
- **Attaques** — `nmap`, `hydra` pour valider la détection Wazuh

Chaque résultat est consigné dans `matrice-flux.md` avec la commande utilisée.

---

> Dernière mise à jour : avril 2026
