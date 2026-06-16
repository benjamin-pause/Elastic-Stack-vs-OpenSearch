# Elastic-Stack-vs-OpenSearch
Déploiement comparatif avec Docker   Projet réalisé en stage BTS SIO SISR — Juin 2026  Objectif : déployer et comparer deux solutions de supervision et d'analyse de logs open source
## Contexte

Dans le cadre d'une mission de stage, deux stacks de supervision ont été déployées en parallèle sur un serveur Linux avec Docker, afin d'évaluer leurs fonctionnalités, leur facilité de déploiement et leur pertinence pour un usage en entreprise (SIEM, monitoring, analyse de logs).

---

## Architecture

```
┌─────────────────────────────────────────────────────────────┐
│                        Docker Host (Linux)                  │
└──────────────┬──────────────────────────┬───────────────────┘
               │                          │
   ┌───────────▼───────────┐  ┌───────────▼───────────┐
   │     Elastic Stack      │  │    OpenSearch Stack    │
   │                        │  │                        │
   │  Elasticsearch :9200   │  │  OpenSearch     :9201  │
   │  Kibana        :5601   │  │  OS Dashboards  :5602  │
   │  Fleet Server  :8220   │  │                        │
   │  Filebeat              │  │                        │
   │  Metricbeat            │  │                        │
   │  Auditbeat             │  │                        │
   └────────────────────────┘  └────────────────────────┘
         Réseau : elastic_net        Réseau : opensearch_net
```

Les deux stacks sont **totalement isolées** (réseaux Docker distincts, volumes séparés, ports différents). Aucune communication entre elles n'est prévue : c'est voulu pour garantir une comparaison indépendante.

---

## Ports utilisés

| Service               | Port hôte | Rôle                          |
|-----------------------|-----------|-------------------------------|
| Elasticsearch         | 9200      | API REST Elastic               |
| Kibana                | 5601      | Interface web Elastic          |
| Fleet Server          | 8220      | Gestion agents Elastic         |
| OpenSearch            | 9201      | API REST OpenSearch            |
| OpenSearch Dashboards | 5602      | Interface web OpenSearch       |

---

## Structure du projet

```
elastic-vs-opensearch/
├── elastic/
│   └── docker-compose.yml      # Stack complète : ES + Kibana + Fleet + Beats
├── opensearch/
│   └── docker-compose.yml      # Stack : OpenSearch + Dashboards
└── README.md                   # Ce fichier
```

---

## Démarrage rapide

### Elastic Stack
```bash
cd elastic/
ELASTIC_PASSWORD=votreMotDePasse KIBANA_PASSWORD=votreMotDePasse docker compose up -d
```
Interface Kibana : http://localhost:5601 (login : `elastic` / mot de passe défini)

### OpenSearch Stack
```bash
cd opensearch/
OPENSEARCH_PASSWORD='Admin@1234!' docker compose up -d
```
Interface Dashboards : http://localhost:5602 (login : `admin` / mot de passe défini)

---

## Comparaison fonctionnelle

| Critère                        | Elastic Stack 8.x            | OpenSearch 2.x               |
|--------------------------------|------------------------------|------------------------------|
| **Origine**                    | Elastic NV                   | Fork AWS (depuis Elastic 7.10)|
| **Licence**                    | SSPL (non libre) + Basic     | Apache 2.0 (100% open source)|
| **Interface web**              | Kibana                       | OpenSearch Dashboards        |
| **Sécurité par défaut**        | TLS activé dès l'install     | Plugin Security (configurable)|
| **Gestion des agents**         | Fleet Server + Elastic Agent | Intégration tierce (ex: Beats)|
| **Composants de collecte**     | Filebeat, Metricbeat, Auditbeat | Compatible Beats (si version compatible) |
| **Alerting**                   | Kibana Alerts (licence Basic+)| Alerting intégré (gratuit)   |
| **SIEM**                       | Elastic Security (licence)   | Security Analytics (gratuit) |
| **Facilité de déploiement**    | Plus complexe (certificats)  | Plus simple à démarrer       |
| **Documentation**              | Très complète (officielle)   | Bonne (communauté AWS)       |
| **Communauté**                 | Très large                   | En croissance                |

---

## Observations issues du déploiement

**Elastic Stack**
- La sécurité (TLS, authentification) est activée par défaut sur les versions récentes, ce qui complique le démarrage initial mais est un avantage en production.
- Fleet Server simplifie énormément la gestion centralisée des agents dans un réseau d'entreprise.
- Les Beats (Filebeat, Metricbeat, Auditbeat) couvrent tous les besoins de collecte sans configuration lourde.

**OpenSearch**
- Démarrage plus simple et plus rapide grâce à la configuration moins stricte par défaut.
- La licence Apache 2.0 est un avantage majeur pour les entreprises qui veulent éviter les restrictions SSPL.
- L'interface OpenSearch Dashboards est très proche de Kibana (fork direct), donc la courbe d'apprentissage est nulle si on connaît déjà Kibana.

---

## Conclusion

Pour un usage **SIEM en entreprise avec gestion d'agents et collecte avancée**, Elastic Stack reste plus mature et outillé.  
Pour un usage **open source sans contraintes de licence ou un déploiement rapide**, OpenSearch est une alternative solide et entièrement gratuite.

---

## Compétences mobilisées

- Administration Linux (Docker, Docker Compose)
- Supervision et analyse de logs (Elasticsearch, OpenSearch)
- Configuration de la sécurité réseau (isolation Docker, ports, TLS)
- Analyse comparative de solutions techniques
- Documentation technique (Markdown, schémas ASCII)
