# 06 — Sécurité

> Statut : **à réaliser** — liste de contrôle du durcissement de la stack.

## Secrets

| Mesure | État |
|---|---|
| Identifiants Grafana dans `.env`, hors du `docker-compose.yml` | ✅ |
| `.env` en `chmod 600` | ⚠️ à vérifier sur le serveur |
| `.env` exclu du dépôt (`.gitignore`) + `.env.example` fourni | ❌ |
| Mot de passe admin Grafana fort et différent de la valeur par défaut | ⚠️ à vérifier |

## Exposition réseau

| Mesure | État |
|---|---|
| Prometheus, Alertmanager, exporteurs non publiés sur l'hôte | ✅ |
| Grafana restreint (`127.0.0.1:3008:3000` + accès via VPN/tunnel ou reverse proxy) | ❌ actuellement publié sur toutes les interfaces |
| HTTPS devant Grafana (reverse proxy ou tunnel) | ❌ |
| Pare-feu de l'hôte (ex. `ufw`) : seuls SSH, WireGuard et le point d'entrée web autorisés | ❌ |

> Docker écrit ses propres règles iptables : un port publié par Compose **contourne `ufw`**. La restriction doit se faire au niveau du `ports:` (bind sur `127.0.0.1` ou sur l'IP du VPN).

## Grafana

| Mesure | État |
|---|---|
| Inscription libre désactivée (`GF_USERS_ALLOW_SIGN_UP=false`) | ✅ |
| Accès anonyme désactivé (valeur par défaut) | ✅ |
| Comptes nominatifs avec rôle `Viewer` pour la consultation | ❌ |

## Images et conteneurs

| Mesure | État |
|---|---|
| Prometheus et Grafana épinglés par digest | ✅ |
| Alertmanager, Node Exporter, cAdvisor épinglés (actuellement `:latest`) | ❌ |
| Volumes de l'hôte montés en lecture seule (`:ro`) | ✅ |
| Mises à jour planifiées des images (veille CVE) | ❌ |

## Sauvegarde

| Élément | Méthode |
|---|---|
| Configuration (`docker/`) | Versionnée dans le dépôt Git |
| Volume `grafana_data` | Sauvegarde périodique (ou dashboards provisionnés en fichiers) |
| Volume `prom_data` | Optionnel : données reconstituables, rétention 30 j |

```bash
docker run --rm -v eyes-on-prod_grafana_data:/data -v "$PWD":/backup alpine \
  tar czf /backup/grafana_data-$(date +%F).tgz -C /data .
```
