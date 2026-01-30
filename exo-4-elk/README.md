# Exercice 4 - ELK (Elasticsearch + Logstash + Kibana + Filebeat)

---

## Contexte

Déployer une stack ELK (Elasticsearch, Logstash, Kibana, Filebeat) pour centraliser et visualiser des logs.
Le docker-compose.yml fourni ne démarrait pas correctement et l’ingestion de logs n’était pas visible dans Kibana.

L’objectif est d’obtenir une stack fonctionnelle, stable au démarrage, et capable d’ingérer des événements via Filebeat.

---

## Problèmes rencontrés

Les dysfonctionnements ont été identifiés via l’analyse des logs Docker
(docker compose logs elasticsearch, docker compose logs logstash, docker compose logs kibana, docker compose logs filebeat)
et par vérification des indices Elasticsearch depuis un conteneur (curl sur l’API).

1. Erreur de bind mount (fichier vs dossier)

- Docker montait des dossiers à la place des fichiers logstash.yml et filebeat.yml.
- Symptôme typique : erreur not a directory (conflit entre un fichier attendu et un répertoire monté).

2. Variable d’environnement manquante (Logstash)

- Docker Compose signalait LS_HEAP_SIZE non définie.
- Logstash quittait avec exit(1) à cause d’une configuration incomplète.

3. Ingestion non visible initialement

- Aucun index n’apparaissait (0 index) malgré les services démarrés.
- Sur Docker Desktop Windows, la source de logs /var/log côté conteneur ne correspondait pas à un flux exploitable
  dans ce contexte (input inadapté).

---

## Correctifs appliqués

- Recréation de vrais fichiers de configuration (pour éviter les montages de répertoires) :
  - logstash/config/logstash.yml
  - logstash/pipeline/logstash.conf
  - filebeat/filebeat.yml

- Externalisation des paramètres dans .env + .env.example :
  - ES_HEAP_SIZE=512m
  - LS_HEAP_SIZE=256m
  - KIBANA_PORT=5601

- Création d’un réseau Docker dédié :
  - exo4_net

- Exposition minimale des ports :
  - Kibana exposé sur 5601
  - Elasticsearch non exposé (tests via conteneur)

- Ajout de healthchecks :
  - Elasticsearch
  - Kibana
  - Logstash (si présent dans le compose)

- Filebeat adapté à Docker Desktop Windows :
  - exécution en root
  - commande --strict.perms=false (contourne les contraintes de permissions sur les mounts)

- Validation de l’ingestion :
  - index filebeat-8.11.0-2026.01.30 présent avec docs.count > 0

---

## Bonnes pratiques mises en œuvre

- Réseau Docker isolé
  - Tous les services communiquent via le réseau exo4_net.

- Gestion propre des paramètres
  - Utilisation de .env / .env.example pour les variables (ports, heap sizes).

- Démarrage fiable
  - Healthchecks pour valider l’état “prêt” des services critiques.

- Surface d’exposition minimale
  - Kibana uniquement exposé sur l’hôte.
  - Elasticsearch accessible uniquement depuis le réseau Docker interne.

- Compatibilité Windows / Docker Desktop
  - Ajustement Filebeat (root + strict.perms=false) pour éviter les blocages liés aux permissions.

---

## Livrables

- docker-compose.yml — configuration corrigée et fonctionnelle
- .env.example — variables d’environnement attendues
- .env — utilisé localement, non versionné
- logstash/config/logstash.yml — configuration Logstash
- logstash/pipeline/logstash.conf — pipeline Logstash
- filebeat/filebeat.yml — configuration Filebeat

---

## Configuration

Créer un fichier .env dans ce dossier à partir de .env.example.

Variables attendues :

```env
ES_HEAP_SIZE=512m
LS_HEAP_SIZE=256m
KIBANA_PORT=5601
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

- Elasticsearch en état healthy
- Kibana en état healthy
- Logstash en état healthy (si healthcheck présent)
- Filebeat en cours d’exécution

Test Elasticsearch depuis le conteneur Kibana :

```bash
docker compose exec kibana curl http://elasticsearch:9200
```

Vérification des outputs Filebeat :

```bash
docker compose exec filebeat filebeat --strict.perms=false test output
```

Vérification des indices Elasticsearch :

```bash
docker compose exec kibana curl http://elasticsearch:9200/_cat/indices?v
```

Résultat attendu :

- Présence d’un index filebeat-\* avec docs.count > 0
- Statut yellow possible en single-node (normal)

---

## Notes

- Le statut yellow est normal en single-node (absence de replicas).
- L’ingestion dépend de la source de logs disponible dans l’environnement Docker Desktop Windows.
