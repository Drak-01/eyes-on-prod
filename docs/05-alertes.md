# 05 — Alertes

> Statut : **en cours** — Alertmanager est déployé mais pas encore relié à Prometheus, et aucun récepteur n'est configuré.

## Chaîne d'alerte

```mermaid
flowchart LR
    R[Règles d'alerte<br/>prometheus/rules/*.yml] --> P[Prometheus<br/>évaluation toutes les 15 s]
    P -- "alertes actives" --> AM[Alertmanager<br/>groupement, déduplication, silences]
    AM --> N[E-mail / Slack / Telegram ...]
```

| Élément | Responsabilité |
|---|---|
| Prometheus | Évalue les expressions PromQL et déclenche l'alerte après la durée `for` |
| Alertmanager | Groupe, déduplique, applique les silences et route vers le bon récepteur |

## État actuel

`alertmanager/alertmanager.yml` :

```yaml
route:
  receiver: default

receivers:
  - name: default
```

Le récepteur `default` n'a aucune intégration : les alertes reçues ne sont envoyées nulle part.

## Travaux restants

### 1. Relier Prometheus à Alertmanager

Ajouter dans `prometheus/prometheus.yml` :

```yaml
alerting:
  alertmanagers:
    - static_configs:
        - targets: ['alertmanager:9093']

rule_files:
  - /etc/prometheus/rules/*.yml
```

Et monter le dossier des règles dans le service `prometheus` :

```yaml
      - ./prometheus/rules:/etc/prometheus/rules:ro
```

### 2. Règles d'alerte de base

`prometheus/rules/node.yml` (proposition) :

| Alerte | Expression | Durée | Sévérité |
|---|---|---|---|
| `TargetDown` | `up == 0` | 2m | critical |
| `HighCpuUsage` | `100 - avg by (instance) (rate(node_cpu_seconds_total{mode="idle"}[5m])) * 100 > 90` | 10m | warning |
| `HighMemoryUsage` | `(1 - node_memory_MemAvailable_bytes / node_memory_MemTotal_bytes) * 100 > 90` | 5m | warning |
| `DiskAlmostFull` | `(1 - node_filesystem_avail_bytes{fstype!~"tmpfs\|overlay"} / node_filesystem_size_bytes) * 100 > 85` | 5m | warning |

### 3. Configurer un récepteur

Choisir le canal (e-mail, Slack, Telegram, webhook) et ajouter la configuration correspondante dans `alertmanager.yml`. Les secrets (mot de passe SMTP, token) ne doivent pas être versionnés.

## Vérification

```bash
docker compose run --rm --entrypoint promtool prometheus \
  check rules /etc/prometheus/rules/node.yml
docker compose exec prometheus wget -qO- http://localhost:9090/api/v1/rules
docker compose exec prometheus wget -qO- http://localhost:9090/api/v1/alerts
```

Test de bout en bout : arrêter `node-exporter` (`docker compose stop node-exporter`), attendre 2 minutes, vérifier la réception de `TargetDown`, puis le redémarrer.

## Référence

- [Alerting rules](https://prometheus.io/docs/prometheus/latest/configuration/alerting_rules/)
- [Configuration Alertmanager](https://prometheus.io/docs/alerting/latest/configuration/)
