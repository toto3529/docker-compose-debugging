# Docker Compose Debugging - Phase 5

---

## Contexte

Cette phase a pour objectif de corriger et fiabiliser plusieurs stacks Docker Compose
présentant des dysfonctionnements au démarrage ou à l’exécution.

Chaque exercice consiste à :

- analyser les erreurs à partir des logs Docker,
- identifier les causes racines,
- appliquer des correctifs conformes aux bonnes pratiques,
- documenter les solutions de manière claire et factuelle.

Le périmètre couvre des stacks applicatives variées, représentatives de cas réels
rencontrés en environnement de développement ou de pré-production.

---

## Objectifs

- Diagnostiquer des erreurs Docker Compose à partir des logs
- Corriger des problèmes d’initialisation, de dépendances et de configuration
- Appliquer des bonnes pratiques Docker (réseau, sécurité, healthchecks)
- Produire des configurations fonctionnelles et reproductibles
- Documenter chaque correction de manière professionnelle

---

## Structure du dépôt

Le dépôt est organisé par exercice, chaque dossier contenant :

- un docker-compose.yml corrigé
- un README dédié décrivant les problèmes et les solutions
- un fichier .env.example (le .env réel n’est jamais versionné)

Structure simplifiée :

```text

docker-compose-debugging/
├── exo-1-wordpress-mysql/
│   └── README.md
├── exo-2-nextcloud-postgres-redis/
│   └── README.md
├── exo-3-mattermost-postgres/
│   └── README.md
├── exo-4-elk/
│   └── README.md
└── README.md
```

---

## Exercices réalisés

EXERCICE 1 — WordPress + MySQL

- Problèmes d’initialisation MySQL (mot de passe root manquant)
- Connexions prématurées WordPress ↔ MySQL
- Absence de healthchecks
- Exposition inutile de la base de données

Correctifs principaux :

- Externalisation des secrets via .env
- Ajout d’un healthcheck MySQL
- depends_on avec condition service_healthy
- Réseau Docker isolé

---

EXERCICE 2 — Nextcloud + PostgreSQL + Redis

- Erreur “Internal Server Error” au démarrage
- Initialisation Nextcloud incomplète (installed: false)
- Secrets codés en dur
- Dépendances non garanties
- Exposition inutile de PostgreSQL

Correctifs principaux :

- Variables Nextcloud et PostgreSQL externalisées
- Ajout de Redis avec healthcheck
- depends_on condition service_healthy
- Réseau Docker dédié
- Ports minimisés

---

EXERCICE 3 — Mattermost + PostgreSQL

- Erreur de connexion PostgreSQL liée au SSL
- Version PostgreSQL incompatible (13.x au lieu de 14.x)
- Démarrage Mattermost avant disponibilité de la base

Correctifs principaux :

- Désactivation SSL via sslmode=disable
- Mise à jour PostgreSQL vers la version 14
- Healthcheck PostgreSQL
- depends_on condition service_healthy
- Réseau Docker dédié

---

EXERCICE 4 — ELK (Elasticsearch + Logstash + Kibana + Filebeat)

- Erreurs de bind mount (fichier vs dossier)
- Variable LS_HEAP_SIZE manquante
- Ingestion de logs non visible initialement
- Contraintes spécifiques Docker Desktop Windows

Correctifs principaux :

- Recréation des fichiers de configuration Logstash et Filebeat
- Externalisation des heap sizes et ports
- Healthchecks Elasticsearch / Kibana / Logstash
- Adaptation Filebeat (root + strict.perms=false)
- Validation de l’ingestion via indices Elasticsearch

---

## Bonnes pratiques appliquées

- Utilisation systématique de fichiers .env et .env.example
- Suppression de l’exposition inutile des services internes
- Réseaux Docker dédiés par exercice
- Healthchecks pour fiabiliser les dépendances
- depends_on avec condition service_healthy
- Historique Git clair, commits atomiques par exercice
- Documentation concise, factuelle et orientée diagnostic

---

## Lancement des exercices

Chaque exercice peut être lancé indépendamment.

Depuis le dossier de l’exercice concerné :

```bash
docker compose up -d
```

Les vérifications spécifiques sont détaillées dans le README de chaque exercice.

---

## Notes

- Les fichiers .env ne sont jamais versionnés.
- Les statuts healthy peuvent nécessiter quelques secondes au premier démarrage.
- Certains ajustements sont spécifiques à Docker Desktop Windows et sont documentés le cas échéant.
