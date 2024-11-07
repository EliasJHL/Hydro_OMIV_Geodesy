# Front-End | RShiny

## Installation R & Shiny

La version de **R** utilisé ici est la version `R Package Version` et la version de Shiny est la `Shiny Package Version`

Pour installer la dernière version de R

```bash
$ deb http://cloud.r-project.org/bin/linux/debian buster-cran40/
```

Pour installer la dernière version de Shiny, on compile depuis les sources :

```bash 
https://github.com/rstudio/shiny-server/wiki/Building-Shiny-Server-from-Source
```

## Installation des packages R

Les packages utilisés sont issues de CRAN en grande majorité cependant certains d'eux sont issues de GitHub

###  1. Préparation des outils nécessaires

Tous les packages nécessaires se trouvent dans le fichier `DESCRIPTION`, pour correctement installer les bonnes version vous pouvez utiliser le package `devtools`. Vous pouvez l'installer via cette commande : 

```R
install.packages("devtools")
```

Si vous venez à ajouter des packages ou autres configurations il sera important que tout soit spécifié dans le fichier `DESCRIPTION` qui est généré par *golem*

### 2. Installation des packages

Pour installer les dépendances listés vous pouvez utiliser `devtools::install_deps()` il installera tous les packages à la bonne version.

```R
devtools::install_deps()
```

### 3. Vérification de l'intégrité de l'application

Vous pouvez utiliser `devtools::check()` pour vérifier que tout est correctement installé et configuré

```R
devtools::check()
```

-----

# Back-End | Docker

## Installation de docker

Le docker contient les deux services, `backend_hydro-api_extenso-1` qui contient toute la partie API et `backend_hydro-db_extenso-1` qui contient la base de données et la gestion des connexions.

- Pré-requis : Configuration de l'environnement (voir [configuration env](environnement.md))

## Composition & Structure

    | 
    |-- backend_extenso/
    |   |-- Dockerfile
    |   |-- docker-compose-dev.yml  #Version de développement utilisé dans /dev/
    |   |-- docker-compose-prod.yml  #Version de production utilisé dans /omiv/
    |   `-- api_service/
    |         |-- app.py
    |         |-- db_extenso_omiv.py
    |         `-- parsing_scripts/
    |             |-- start.sh
    |             `-- [Scripts de parsing]
    |-- .gitlab-ci.yml
    `-- start.sh

- `backend_extenso/` -> Contient tout le back-end de l'application
- `.gitlab-ci.yml` -> GitLab CI pour l'auto-déploiement (version développement & version de production)
- `start.sh` -> Script de lancement de la version de production uniquement

## Installation de Docker

Dans l'application actuelle la version utilisé est la `Docker version 20.10.5+dfsg1, build 55c4c88`.

Il est recommandé d'installer Docker Engine au lieu de Docker io qui peut ne pas être compatible avec l'application.

Pour l'installation, la documentation docker est disponible directement via [ce lien](https://docs.docker.com/engine/install/).