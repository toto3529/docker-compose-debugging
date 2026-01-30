# Exercice 2 — Nextcloud + PostgreSQL (+ Redis)

## Contexte

Déployer un stack Nextcloud avec une base de données PostgreSQL et un service Redis.
Le docker-compose.yml fourni présentait une erreur bloquante dès l’accès à l’application.

L’objectif est d’obtenir une installation Nextcloud fonctionnelle, stable et conforme aux bonnes pratiques Docker
(isolation réseau, gestion des dépendances, sécurisation des secrets).

---

## Problèmes rencontrés

Les dysfonctionnements ont été identifiés via l’analyse des logs Docker
(docker compose logs nextcloud, docker compose logs postgres)
et par inspection de l’état de Nextcloud via le fichier status.php.

1. Internal Server Error côté Nextcloud

- Nextcloud ne disposait pas des variables nécessaires à son initialisation
  (compte administrateur, configuration de la base de données).

2. Initialisation incomplète

- L’endpoint status.php indiquait installed: false avant configuration.

3. Secrets codés en dur

- Les identifiants PostgreSQL et paramètres Nextcloud étaient définis directement
  dans le docker-compose.yml.

4. Dépendances non garanties

- Nextcloud pouvait démarrer avant PostgreSQL ou Redis, entraînant des erreurs au boot.

5. Exposition inutile de services

- PostgreSQL était exposé sur l’hôte alors qu’un accès interne au réseau Docker suffisait.

---

## Correctifs appliqués

- Externalisation des variables sensibles dans un fichier .env :
  - paramètres PostgreSQL
  - identifiants administrateur Nextcloud

- Ajout explicite des variables requises par Nextcloud :
  - NEXTCLOUD_ADMIN_USER
  - NEXTCLOUD_ADMIN_PASSWORD
  - POSTGRES_HOST
  - POSTGRES_DB
  - POSTGRES_USER
  - POSTGRES_PASSWORD

- Création d’un réseau Docker dédié :
  - exo2_net

- Ajout de healthchecks :
  - PostgreSQL (pg_isready)
  - Redis (ping)

- Mise en place de depends_on avec condition service_healthy :
  - Nextcloud attend PostgreSQL et Redis avant démarrage

- Suppression de l’exposition du port PostgreSQL sur l’hôte

---

## Bonnes pratiques mises en œuvre

- Réseau Docker isolé
  - Tous les services communiquent via le réseau exo2_net.

- Gestion sécurisée des secrets
  - Aucune variable sensible n’est stockée en dur dans le compose.
  - Fourniture d’un fichier .env.example pour la reproductibilité.

- Dépendances fiables
  - Utilisation de healthchecks combinés à depends_on condition service_healthy.

- Surface d’exposition minimale
  - Seul le port Nextcloud est exposé sur l’hôte.
  - PostgreSQL et Redis restent accessibles uniquement via le réseau interne Docker.

---

## Livrables

- docker-compose.yml — configuration corrigée et fonctionnelle
- .env.example — variables d’environnement attendues
- .env — utilisé localement, non versionné

---

## Configuration

Créer un fichier .env dans ce dossier à partir de .env.example.

Variables attendues :

```env

POSTGRES_DB=nextcloud
POSTGRES_USER=nextcloud
POSTGRES_PASSWORD=change_me
POSTGRES_HOST=postgres

NEXTCLOUD_ADMIN_USER=admin
NEXTCLOUD_ADMIN_PASSWORD=change_me
```

---

## Lancement

Depuis le dossier de l’exercice :

```bash
docker compose up -d
```

---

## Vérifications

Etat des services Docker :

```bash
docker compose ps
```

Résultat attendu :

- PostgreSQL et Redis en état healthy
- Redis opérationnel en tant que cache pour Nextcloud
- Nextcloud en cours d’exécution

Vérification de l’installation Nextcloud :

```bash
curl http://localhost:8080/status.php
```

L’endpoint status.php permet de vérifier l’état de l’installation après la configuration initiale.

Résultat attendu :
installed: true

---

## Notes

- Le statut healthy peut prendre quelques secondes lors du premier démarrage.
- L’initialisation Nextcloud est effective uniquement après la première configuration réussie.
