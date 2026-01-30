# Exercice 1 — WordPress + MySQL

## Contexte

Déployer un stack WordPress pour l’équipe marketing. Le `docker-compose.yml` fourni ne fonctionnait pas :

- MySQL ne démarrait pas correctement
- WordPress avait des erreurs de connexion à la base
- La configuration n’appliquait pas les bonnes pratiques attendues

---

## Problèmes rencontrés

1. **MySQL ne démarrait pas** car la configuration ne définissait pas de mot de passe root.
   - L’image officielle `mysql:8.0` exige une initialisation correcte (ex: `MYSQL_ROOT_PASSWORD`).
2. **Risque de connexion WordPress ↔ MySQL instable** : `depends_on` ne garantit pas que MySQL soit prêt (uniquement l’ordre de démarrage).
3. **Sécurité / exposition inutile** : la base MySQL n’a pas besoin d’être exposée sur la machine hôte.
4. **Secrets dans le compose** : mots de passe en dur au lieu d’utiliser un fichier `.env`.

---

## Correctifs appliqués

- Ajout de la variable requise pour MySQL :
  - `MYSQL_ROOT_PASSWORD` (via `.env`)
- Alignement des paramètres WordPress sur les variables d’environnement :
  - `WORDPRESS_DB_HOST`, `WORDPRESS_DB_USER`, `WORDPRESS_DB_PASSWORD`, `WORDPRESS_DB_NAME`
- Réinitialisation du volume MySQL après correction (afin de réappliquer l’initialisation avec les nouvelles variables) :
  - `docker compose down -v` puis `docker compose up -d`

---

## Bonnes pratiques mises en œuvre

- **Réseau Docker isolé**
  - Création d’un réseau dédié `exo1_net` et rattachement des services (WordPress / MySQL / phpMyAdmin) à ce réseau.
- **Sécurisation via `.env`**
  - Les variables sensibles ne sont pas codées en dur dans le `docker-compose.yml`.
  - Fourniture d’un `.env.example` pour faciliter la reproduction.
- **Healthcheck MySQL**
  - Ajout d’un `healthcheck` MySQL pour détecter l’état “prêt” (healthy).
- **Dépendances propres**
  - WordPress et phpMyAdmin attendent que MySQL soit `healthy` via `depends_on` + `condition: service_healthy`.
- **Sécurité des ports**
  - Suppression de l’exposition du port MySQL sur l’hôte (MySQL reste accessible uniquement sur le réseau Docker interne).

---

## Livrables

- docker-compose.yml — configuration corrigée et fonctionnelle
- .env.example — variables d’environnement attendues
- .env — utilisé localement, non versionné

---

## Configuration

Créer un fichier `.env` dans ce dossier à partir de `.env.example`.

Variables attendues :

```env
# Database
MYSQL_DATABASE=wordpress
MYSQL_USER=wordpress
MYSQL_PASSWORD=change_me
MYSQL_ROOT_PASSWORD=change_me

# WordPress
WORDPRESS_DB_HOST=mysql
```

---

## Lancement

Depuis le dossier de l’exercice :

```bash
docker compose up -d
```

---

## Vérifications

### Accès aux services

- WordPress : http://localhost:8080
- phpMyAdmin : http://localhost:8081

### Vérifier l’état MySQL

```bash
docker inspect --format='{{.State.Health.Status}}' $(docker compose ps -q mysql)
```

Résultat attendu :

```bash
healthy
```

---

## Notes

- Le statut healthy peut prendre quelques secondes à apparaître lors du premier démarrage.
- Les tables WordPress sont créées uniquement après la finalisation de l’installation WordPress.
