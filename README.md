# Eyes on Prod — Centre de surveillance de serveurs

Stack de supervision auto-hébergée, déployée avec Docker Compose, pour collecter, visualiser et alerter sur l'état de serveurs Linux et de leurs conteneurs.

| Composant | Rôle | Image | Port interne |
|---|---|---|---|
| Prometheus | Collecte (pull) et stockage des métriques (TSDB, rétention 30 j) | `prom/prometheus` (digest épinglé) | 9090 |
| Grafana | Visualisation, tableaux de bord | `grafana/grafana:13.2.4-ubuntu` | 3000 → exposé sur **3008** |
| Alertmanager | Routage et envoi des alertes | `prom/alertmanager` | 9093 |
| Node Exporter | Métriques système de l'hôte (CPU, RAM, disque, réseau) | `prom/node-exporter` | 9100 |
| cAdvisor | Métriques par conteneur Docker | `gcr.io/cadvisor/cadvisor` | 8080 |
| postgres_exporter | Métriques PostgreSQL (connexions, transactions, verrous) | `quay.io/prometheuscommunity/postgres-exporter` | 9187 |
| Blackbox Exporter | Sondes HTTPS externes, expiration TLS | `prom/blackbox-exporter` | 9115 |

## Arborescence du dépôt

```
eyes-on-prod/
├── README.md
├── docker/                         # À copier tel quel dans ~/eyes-on-prod sur le serveur
│   ├── docker-compose.yml
│   ├── .env                        # Secrets (NE PAS versionner)
│   ├── prometheus/
│   │   ├── prometheus.yml
│   │   └── rules/blackbox.yml
│   ├── blackbox/blackbox.yml
│   ├── alertmanager/alertmanager.yml
│   └── grafana/provisioning/datasources/prometheus.yml
└── docs/
    ├── 01-architecture.md
    ├── 02-prometheus.md
    ├── 03-grafana.md
    ├── 04-vpn-wireguard.md
    ├── 05-alertes.md
    ├── 06-securite.md
    ├── 07-docker.md
    ├── 08-database.md
    ├── 09-blackbox.md
    └── assets/
```

## Démarrage rapide

Prérequis : Docker Engine ≥ 24 et le plugin `docker compose` v2 sur le serveur.

```bash
# 1. Copier le dossier docker/ sur le serveur
scp -r docker/ user@serveur:~/eyes-on-prod

# 2. Sur le serveur : créer le fichier de secrets
cd ~/eyes-on-prod
cat > .env <<'EOF'
GRAFANA_ADMIN_USER=admin
GRAFANA_ADMIN_PASSWORD=<mot-de-passe-fort>
EOF
chmod 600 .env

# 3. Lancer la stack
docker compose up -d
docker compose ps
```

Grafana est alors accessible sur `http://<ip-serveur>:3008`.

## Documentation

| # | Document | Contenu |
|---|---|---|
| 01 | [Architecture](docs/01-architecture.md) | Composants, flux, réseau, volumes |
| 02 | [Prometheus](docs/02-prometheus.md) | Installation, `prometheus.yml`, vérification |
| 03 | [Grafana](docs/03-grafana.md) | Installation, source de données, dashboards |
| 04 | [VPN WireGuard](docs/04-vpn-wireguard.md) | Supervision de serveurs distants via tunnel |
| 05 | [Alertes](docs/05-alertes.md) | Règles Prometheus et Alertmanager |
| 06 | [Sécurité](docs/06-securite.md) | Durcissement de la stack |
| 07 | [Surveillance des services](docs/07-docker.md) | Exporteurs applicatifs : PostgreSQL, Blackbox |
| 08 | [PostgreSQL](docs/08-database.md) | postgres_exporter, rôle `pg_monitor`, dashboards, alertes |
| 09 | [Blackbox Exporter](docs/09-blackbox.md) | Sondes HTTPS, expiration TLS, alertes de disponibilité |
# eyes-on-prod
