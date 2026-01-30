# Exercice 3 — Mattermost + PostgreSQL

---

## Contexte

Déployer un stack Mattermost avec une base de données PostgreSQL.
Le docker-compose.yml fourni ne permettait pas à l’application de démarrer correctement.

L’objectif est d’obtenir une instance Mattermost fonctionnelle, conforme aux exigences
de compatibilité PostgreSQL et aux bonnes pratiques Docker.

---

## Problèmes rencontrés

Les dysfonctionnements ont été identifiés via l’analyse des logs Docker
(docker compose logs mattermost, docker compose logs postgres).

1. Erreur de connexion PostgreSQL (SSL)

- Les logs Mattermost indiquaient :
  pq: SSL is not enabled on the server
- La configuration par défaut tentait une connexion SSL alors que PostgreSQL
  n’était pas configuré pour l’accepter.

2. Version PostgreSQL incompatible

- Après correction du paramètre SSL, Mattermost échouait avec l’erreur :
  minimum Postgres version requirements not met
- Version détectée : PostgreSQL 13.x
- Version requise par Mattermost : PostgreSQL 14.0 minimum

3. Dépendances non garanties

- Mattermost pouvait démarrer avant que PostgreSQL soit prêt,
  entraînant des erreurs au démarrage.

4. Exposition inutile de la base de données

- Le port PostgreSQL était exposé sur l’hôte sans nécessité fonctionnelle.

---

## Correctifs appliqués

- Externalisation des variables sensibles dans un fichier .env :
  - paramètres PostgreSQL
  - paramètres Mattermost (MM\_\*)

- Correction de la chaîne de connexion PostgreSQL :
  - ajout de ?sslmode=disable dans la datasource Mattermost

- Mise à jour de l’image PostgreSQL :
  - passage à postgres:14 pour respecter les prérequis Mattermost

- Ajout d’un healthcheck PostgreSQL :
  - utilisation de pg_isready

- Mise en place de depends_on avec condition service_healthy :
  - Mattermost attend PostgreSQL avant démarrage

- Création d’un réseau Docker dédié :
  - exo3_net

- Suppression de l’exposition du port PostgreSQL sur l’hôte

---

## Bonnes pratiques mises en œuvre

- Réseau Docker isolé
  - Tous les services communiquent via le réseau exo3_net.

- Gestion sécurisée des secrets
  - Aucune variable sensible n’est codée en dur dans le docker-compose.yml.
  - Fourniture d’un fichier .env.example pour la reproductibilité.

- Compatibilité logicielle
  - Respect strict des versions minimales requises par Mattermost.

- Dépendances fiables
  - Utilisation de healthchecks combinés à depends_on condition service_healthy.

- Surface d’exposition minimale
  - Seul le port Mattermost est exposé sur l’hôte.
  - PostgreSQL reste accessible uniquement sur le réseau Docker interne.

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
POSTGRES_DB=mattermost
POSTGRES_USER=mattermost
POSTGRES_PASSWORD=change_me
POSTGRES_HOST=postgres

MM_USERNAME=admin
MM_PASSWORD=change_me
MM_DBNAME=mattermost
MM_SQLSETTINGS_DATASOURCE=postgres://mattermost:change_me@postgres:5432/mattermost?sslmode=disable
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

- PostgreSQL en état healthy
- Mattermost en cours d’exécution

Accès à l’interface Mattermost :

- URL : http://localhost:8065
- Interface accessible en mode desktop et mobile

---

## Notes

- Le statut healthy de PostgreSQL peut prendre quelques secondes au premier démarrage.
- Mattermost nécessite une version minimale de PostgreSQL 14 pour fonctionner correctement.
