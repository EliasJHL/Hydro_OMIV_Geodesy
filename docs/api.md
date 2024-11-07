# Documentation de l'API

Cette API permet d'interagir avec l'application hydro. Elle fournit des routes RESTful pour effectuer des opérations CRUD sur les données.

La base de l'URL dépend de la configuration, par défaut l'url de base est :

```
http://omiv-geodesy.u-strasbg.fr/hydro-rest/
```

## Structure

La structue de l'API est la suivante :

```txt
api_service
    |-- update_base.sh
    |-- app.py
    |-- api/
    |   |-- routes/
    |   |   `-- # Dossier contenant toutes les routes API
    |   |-- base.py
    |   `-- create_app.py
    |
    |-- config/
    |   `-- config.yml # Fichier de configuration pour le parsing
    |
    |-- core/
    |   `-- database.py
    |
    |-- crud/
    |   `-- # Dossier contenant toutes les opérations CRUD utilisés par l'API
    |
    |-- migrations/
    |   `-- # Dossier créé par flask-migrate
    |
    `-- models/
       `-- # Dossier contenant tous les modèles pour la composition de la base de données
```

Lors du lancement de l'API, le fichier `app.py` va lancer touts les fichiers nécessaires notamment `api/create_app.py` qui va créer l'app flask et qui va lancer les vérification de la base de données.

La vérification de la base de données va créer la base de données si celle-ci n'existe pas avec la fonction `check_database_exists()`.

## Routes API  

Toutes les routes commencent par `/api/` pour correctement différencier les requêtes envoyés à l'API

### 1. Routes stations

Les routes station utilisent le fichier crud : **api_service/crud/stations.py**

**a. Récupération des stations**

```bash
GET /api/get-stations/
or
GET /api/get-stations/<int:station_id>
or
GET /api/get-stations/<string:station_name>
```

Cette requête permet de récupérer soit toutes les stations soit une station en particulier pour obtenir une information la concernant.

Exemple de réponse:

```json
[
    {
        "comments":null,
        "enabled":true,
        "id":1,
        "last_update":"2024-09-26 23:50:00",
        "lat":45.4674286,
        "lon":5.9310367,
        "station":"GRAN1",
        "status":"On-Going"
    },
    {
        "comments":null,
        "enabled":true,
        "id":2,
        "last_update":"2024-09-26 23:50:00",
        "lat":45.4674749,
        "lon":5.9310013,
        "station":"GRAN2",
        "status":"On-Going"
    }
]
```


**b. Création d'une station**

```txt
POST /api/create-station?station_name=test&comments=test&lat=0&lon=0&enabled=true
```

Cette reqûete est utilisé dans le panel administrateur pour pourvoir créer une station.

Elle prends comme arguements les paramètres suivants :

```py
station_name = # Nom de la station
comments = # Remarque
lat = # Latitude
lon = # Longitude
enabled = #Si elle est active
```

### 2. Routes Système

Les routes système utilisent le fichier crud : **api_service/crud/system.py**

**a. Téléchargement des données**

```
GET /api/download-data/
```

Cette route permet de télécharger les données brutes venant de la table `hydro_raw` dans la base de données, on peut choisir plusieures stations

Elle prends comme arguements les paramètres suivants :

```py
date_start = # Date de début (plage de données) sinon Date minimale connue
date_end = # Date de fin (plage de données) sinon Date maximale connue
stations_list = # Liste des stations
```

Exemple de réponse:

```txt
Fichier.zip
  |-- GRAN1.csv
  |-- GRAN2.csv
  `-- GRAN3.csv
```

**b. Mise à jour du modèle de base de données (Non recommandé)**

```
GET /api/update-models/
```

Cette requête permet de lancer le script `update_base.sh` sans devoir rentrer dans le docker, cependant en cas de problème les messages d'erreurs ne sont pas complètements transmis.

Exemple de réponse:

```
(Si OK)
{
    'message': 'Models updated successfully'
}

(Sinon)
{
    'error': f'Error updating models : [Message d'erreur]'
}
```

**c. Mise à jour des coordonnées d'une station (uniquement via panel admin)**

```bash
GET /api/update-coordinates/<int:station_id>?lat=<latitude>&lon=<longitude>
```

Cette reqête permet de mettre à jour les coordonnées d'une station via le panel admin.

Elle prends comme arguements les paramètres suivants :

```py
lon = # Longitude
lat = # LAtitude
```


**d. Test du host SMTP**

```bash
GET /api/test/send-mail/?host=smtp.example.com&port=587&user=test@example.com&password=secret&to=recipient@example.com
```

Cette requête est uniquement pour la panel administrateur, elle permet de tester un host (avec les identifiants, etc) avant de le mettre en tant que paramètre sur la base de données.

Cette fonctionnalité a comme unique objectif de vérifier les identifiants pour que lors du parsing les alertes soient bien envoyés aux abonnées.

Elle prends comme arguements les paramètres suivants :

```py
host = # Host SMTP
port = # Port SMTP
username = # Username SMTP
password = # Mot de passe SMTP
to = # Destinataire de test où le mail de test sera envoyé
```

### 3. Routes settings (uniquement panel admin)

#### PAS ENCORE IMPLEMENTÉ

### 4. Routes alert_mail

```
GET /api/alert-mail
or
GET /api/alert-mail/<string:mail>
```

Cette reqûete permet de récupérer soit une liste contenant tous les abonnées utilisé dans le parsing soit les détails d'un utilisateur pour voir ses abonnements aux alertes.


### 5. Routes hydro

Pour la visulisation des données hydro et velocity il n'y a qu'une seule route qui permet de tout visualiser seuls les arguements permettent de selectionner le niveau d'échantillonage et fonctionnent comme des filtres.

**a. Récupération des données brutes hydro**

```
GET /api/get-hydro/
or
GET /api/get-hydro/<int:station_id>
```

Elle prends comme arguements les paramètres suivants :

```py
date_start = # Date de début (plage de données) sinon Date minimale connue
date_end = # Date de fin (plage de données) sinon Date maximale connue
light = # Le niveau d'échantillonage (0 / 1 / 2)
# 0 -> Données brutes
# 1 -> Echantillonage journalier
# 2 -> Echantillonage hebdomadaire
type = # Type soit "displacement" soit "speed" sinon les deux types
```

