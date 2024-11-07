# Configuration de l'application

Cette partie traite les configuration à effectuer avant le premier lancement de l'application.
Les configuration données concernent la base de données, les docker compose ainsi que l'attrbution des ports.

## 1. Configuration des ports

La verison de déploiement et développement sont séparés, donc il est important qu'il n'y ait pas de conflit entre ces deux versions.

### 1.1 Configuration docker compose

La configuration des ports se fait directement dans le docker compose, cependant **il ne faut pas changer les ports de sortie docker** auquel cas il pourrait il y avoir un risque que le service en question ne fonctionne plus.

Exemple:
```
services:
  db_hydro:
    ports:
      - (Port souhaité):5432 # (Port d'entrée):(Port de sortie)
  api_hydro:
    ports:
      - (Port souhaité):5000 # (Port d'entrée):(Port de sortie)
```

## 2. Configuration de la base de données

### 2.1 Emplacement de stockage

L'emplacement de stockage est défini dans le docker compose il y a un pour la version de développement et une autre pour la version de production.

Pour changer l'emplacement il suffit de mettre à jour dans le service `db_hydro` / `db_hydro_dev` le volume souhaité :

```
services:
  db_hydro_dev:
    image: postgres:14.4-alpine
    volumes:
      - (Emplacement souhaité):/var/lib/postgresql/data
```

### 2.2 Structure de la base de données (Premier lancement)

Lors de la création de la base de données par le service db_hydro, la base de données n'a aucune structure, pour lui fournir la structure nécessaire un script bash (`update_base.sh`) est présent dans le dossier `api_service/` qui permet de lui fournir le modèle à la base de données.

Pour lancer le script il faut que le docker soit lancé, pour lancer le container il suffit de spécifier le docker-compose souhaité :

```bash
sudo docker compose -f docker-compose-prod.yml up --build --force-recreate -d
```

Avant de lancer le script de structure pour la base de données, il faut créer un `TYPE ENUM` pour qu'il puisse correctement créer toutes les tables.

Il suffit de lancer cette query dans la base de données :

```sql
CREATE TYPE station_enum AS ENUM ('On-Going', 'Completed');
```

Une fois terminé on pourra lancer le script `update_base.sh`, pour le lancer :

```bash
sudo docker exec -it backend_hydro-api_hydro-1 /bin/bash update_base.sh
```

Normalement une fois lancé la base de données devrait contenir la strcutre.

### 2.3 Parsing des données (Premier lancement)

Pré-requis : [Environnement correctment configuré](environnement.md)

Informations: **Le script de parsing est configuré dans le docker pour se lancer tous les jours à 12h (heure UTC)**

Après avoir correctement structuré la base de données on peut lancer le parsing,

⚠️ **Le premier parsing sera assez long dû à la très grosse quantité de données à traiter**

Le script pour lancer le parsing se trouve dans `/parsing_service/start.sh` il va lancer le script de parsing.

Pour lancer le script on peut le lancer de la même façon que `update_base.sh`.

```
sudo docker exec -it backend_hydro-api_hydro_dev-1 /bin/bash -c "cd /parsing_service/ && ./start.sh"
```

Une fois le parsing terminé les données seront dans la base de données.

### 2.4 Gestion de données

Pour la gestion de données (stations, données hydro, etc) les modifications sont possibles dans le panel admin, on peut désactiver / activer les stations et définir si une plage de données sont affichables ou pas dans l'application hydro.

La modification d'informations d'une stations se déroule également dans le panel administrateur.

