# 07 — Surveillance des services

Node Exporter et cAdvisor couvrent l'hôte et les conteneurs (CPU, RAM, disque, réseau). Ils ne disent rien de l'état **fonctionnel** des services : une base peut être saturée en connexions, ou un site renvoyer des erreurs 502, avec un conteneur parfaitement « Up ». Des exporteurs spécialisés complètent la collecte.

| Niveau | Exporteur | Ce qu'il mesure | Statut |
|---|---|---|---|
| Hôte | Node Exporter | CPU, RAM, disque, réseau, charge | ✅ déployé |
| Conteneurs | cAdvisor | Ressources consommées par conteneur | ✅ déployé |
| Base de données | postgres_exporter | Connexions, transactions, verrous, cache | ✅ [08 — PostgreSQL](08-database.md) |
| Disponibilité externe | Blackbox Exporter | Réponse HTTP/HTTPS, expiration TLS, ports TCP | ✅ [09 — Blackbox](09-blackbox.md) |

## 1. postgres_exporter

Pour chaque instance PostgreSQL : connexions actives, transactions, verrous, deadlocks, taille des bases, efficacité du cache. Les métriques disponibles dépendent des droits du rôle utilisé (`pg_monitor`).

Procédure complète : [08 — Surveillance de PostgreSQL](08-database.md).

## 2. Blackbox Exporter

Sonde les sites de l'extérieur, comme un utilisateur : code HTTP, temps de réponse par phase (DNS, TLS, connexion), expiration du certificat TLS. Sites sondés : `hakilox.com`, `djigitech.site`.

Procédure complète : [09 — Blackbox Exporter](09-blackbox.md).

## Référence

- [postgres_exporter](https://github.com/prometheus-community/postgres_exporter)
- [blackbox_exporter](https://github.com/prometheus/blackbox_exporter)
- [Liste des exporteurs](https://prometheus.io/docs/instrumenting/exporters/)
