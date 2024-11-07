# Base de données (Développeur)

## Utilisation & Mise à jour des modèles

La base de données se base sur les modèles (dans /api_service/models/) pour créer son schèma grâce à `flask-migrate` qui va modifier et suivre les modifications au fur et à mesure qu'on modifie les models, pour lancer la mise à jour de la base de données il y un script bash `update_base.sh` qui va lancer `flask-migrate`.

S'il y a un soucis avec la migration :

- Supprimer toutes la table `alembric_version`
- Relancer le script de migration
- Lancer le script de parsing

**Le parsing prendra un certain temps à se terminer puisuq'il y a une assez grosse quantité de données**

## La structure de la base de données

La base de données contient 7 tables :

- Alembic_version
- alert_mail
- extenso_raw
- extenso_day
- extenso_week
- settings
- stations

### 1.1 Table Alembic_version

Cette table uniquement le numero de version généré par `flask-migrate`, cette table est généré automatiquement lors de la création de la base de données via les modèles.

Elle permet de suivre les mises à jours et de pouvoir revenir à ce numero de version si nécessaire commme l'outil `git`


### 1.2 Table [alert_mail]

Cette table est composé de 7 colonnes:

```
_______________________________________________________________________________________________________________________________________________________
|   id   |   mail   |   inactivity_alert   |   speed_alert   |   extenso_alert   |   max_inactivity   |   max_speed   |   max_extenso  |   stations   |
|-----------------------------------------------------------------------------------------------------------------------------------------------------|
|  int   | varchar  |         bool         |       bool      |        bool       |         int        |     float     |      float     |   varchar    |
```

Cette table permet de stocker les utilisateurs abonnées à une alarme qui est vérifié lors du parsing. Les utilisateurs enregistrés sont recupérés pour verifier si la limite qu'ils ont défini a été dépassée.

Ils peuvent s'abonner à deux types de données, les données extenso et velocity qui sont caractérisés par les colonnes `speed_alert` et `extenso_alert`, si l'une des deux valeurs est `TRUE` alors la valeur correspondante (soit `max_speed` ou `max_extenso`) pour vérifier si la valeur parsé a été dépassée.

L'utilisateur peut s'abonner aux deux types d'alertes `extenso` et `speed`, dans ce cas il y aura des valeurs dans les deux colonnes `max_speed` et `extenso`.

De plus, l'utilisateur peut également choisir les stations sur lequelles il souhaite recevoir ces alertes.

L'abonnement à ces alertes ainsi que la géstion des utilisateurs abonnés se passe directemet dans le panel administrateur.

Si toutes les conditions sont réunies, le mail de l'utilisateur est récupéré et le mail personnalisé est créé directement dans service de parsing.

__Détail des données contenues dans les colonnes:__

- **id** : Primary Key - Identifiant Unique
- **mail** : VarChar UNIQUE - String contenant l'adresse mail
- **inactivity_alert**: Bool - Si l'utilisateur souhaite s'abooner à l'alerte d'inactivité de station
- **speed_alert** : Bool - Si l'utilisateur souhaite s'abonner à l'alerte type `speed`
- **extenso_alert** : Bool - Si l'utilisateur souhaite s'abonner à l'alerte type `extenso`
- **max_inactivity** : Int - Si l'utilisateur est a `inactivity_alert` en `TRUE` alors la valeur est récupéré (nombre de jours)
- **max_speed** : Float - Si l'utilisateur est a `speed_alert` en `TRUE` alors la valeur est récupéré
- **max_extenso** : Float - Si l'utilisateur est a `extenso_alert` en `TRUE` alors la valeur est récupéré
- **stations** : VarChar - Liste python sous forme de char pour les alertes sur des stations en particulier

### 1.3 Table [stations]

Cette table est composé de 8 colonnes:

```
__________________________________________________________________________________________________________
|   id   |   station   |   comments   |   lat   |   lon   |   last_update   |   enabled   |    status    |
|--------------------------------------------------------------------------------------------------------|
|  int   |   varchar   |   varchar    |  float  |  float  |    timestamp    |    bool     | station_enum |
|--------------------------------------------------------------------------------------------------------|
|                                                                                                ^       |
|                                                                                                |       |
|                                                      Enum('On-Going', 'Completed', name='station_enum')|
```

Cette table stocke toutes les information des stations comme la lat, lon, dernière mise à jour, commentaires, etc.

La colonne status est un Enum, elle n'est pas créé par défaut dans la base de données, s'il y a un problème avec la migration de `flask-migrate` il faut regarder la séction 2.2 de la [configuration](config.md) poru créer le `TYPE ENUM` avec de lancer la migration.

La station est une dépendance pour toutes les tables `extenso_` puisqu'ils utilisent des ForeignKey qui pointent sur cette table.

__Détail des données contenues dans les colonnes:__

- **id** : Primary Key - Identifiant Unique
- **station** : VarChar - Le nom de la station
- **comments** : VarChar - Contient des remarques ajoutés par l'administrateur (visibles sur la carte)
- **lat** : Float - Latitude
- **lon** : Float - Longitude
- **last_update** : Timestamp - Dernière mise à jour (Géré par le service de parsing)
- **enabled** : bool - Si la station est active ou pas (Gestion sur le panel admin)
- **status** : station_enum : Status de la station (Soit `On-Going` ou `Completed`)


### 1.3 Table [extenso_raw]

Cette table est composé de 7 colonnes:

```
____________________________________________________________________________________________________
|   id   |   timestamp   |   station_id  |   display   |   displacement   |   speed  |   station   |
|--------------------------------------------------------------------------------------------------|
|  int   |   timestamp   |      int      |     bool    |      float       |  float   |   varchar   |
|--------------------------------------------------------------------------------------------------|
|                                ^                                                          ^      |
|                                |                                                          |      |
|                     ForeignKey('stations.id')                      ForeignKey('stations.station')|
```

Cette table stocke les données brutes (sans échantillonage) qui sont dans les fichiers `.csv`

Elle permet de filtrer de plusieurs façons notamment grâce au stockage de l'id et du nom de la station.

Elle a également des options administrateur qui servent à la géstion de données notammet avec la colonne `display` qui est une option qui est soit `TRUE` soit `FALSE` et permet d'afficher ou pas la donnée en question au cas où celle-ci est faussé ou incorrecte.

__Détail des données contenues dans les colonnes:__

- **id** : Primary Key - Identifiant Unique
- **timestamp** : Timestamp - Date & Heure de la donnée
- **station_id** : Int - Identifiant de la station (lié directement avec `stations.id`)
- **display** : Bool - Permet d'afficher ou pas cette donnée (panel administrateur)
- **displacement** : Float - Contient la valeur récupéré dans le `.csv`
- **speed** : Float - Contient la valeur récupéré dans le `.csv`
- **stations** : VarChar - Contient le nom de la station (lié directement avec `stations.station`)


### 1.4 Table [extenso_day]


Cette table est composé de 11 colonnes:

```
_________________________________________________________________________________________________________________________________________________________________________________
|   id   |   station_id   |   timestamp   |   displacement  |   speed  |   displacement_max   |   displacement_min  |   speed_max   |   speed_min   |   station   |   display   |
|-------------------------------------------------------------------------------------------------------------------------------------------------------------------------------|
|  int   |       int      |   timestamp   |      float      |  float   |         float        |        float        |     float     |     float     |   varchar   |     bool    |
|-------------------------------------------------------------------------------------------------------------------------------------------------------------------------------|
|                 ^                                                                                                                                        ^                    |
|                 |                                                                                                                                        |                    |
|      ForeignKey('stations.id')                                                                                                              ForeignKey('stations.station')    |
```

Cette table stocke les valeurs échantillonés avec une plage de **24h** elle est basé sur la table `extenso_raw`, ainsi la parsing calcule la moyenne par jour pour les colonnes `displacement` et `speed`, il réucpére également les valeurs enregistrés maxiumum et minimum pour les colonnes `_max` et `_min`.

L'option display est également dans cette table pour pouvoir avoir une géstion complète des données à afficher sur l'application extenso

__Détail des données contenues dans les colonnes:__

- **id** : Primary Key - Identifiant Unique
- **station_id** : Int - Identifiant de la station (lié directement avec `stations.id`)
- **timestamp** : Timestamp - Date & Heure de la donnée
- **displacement** : Float - Contient la valeur moyenne (24H)
- **speed** : Float - Contient la valeur moyenne (24H)
- **displacement_max** : Float - Contient la valeur **maximale** enregistré (24H)
- **displacement_min** : Float - Contient la valeur **minimale** enregistré (24H)
- **speed_max** : Float - Contient la valeur **maximale** enregistré (24H)
- **speed_min** : Float - Contient la valeur **minimale** enregistré (24H)
- **stations** : VarChar - Contient le nom de la station (lié directement avec `stations.station`)
- **display** : Bool - Permet d'afficher ou pas cette donnée (panel administrateur)

### 1.4 Table [extenso_week]


Cette table est composé de 11 colonnes:

```
_________________________________________________________________________________________________________________________________________________________________________________
|   id   |   station_id   |   timestamp   |   displacement  |   speed  |   displacement_max   |   displacement_min  |   speed_max   |   speed_min   |   station   |   display   |
|-------------------------------------------------------------------------------------------------------------------------------------------------------------------------------|
|  int   |       int      |   timestamp   |      float      |  float   |         float        |        float        |     float     |     float     |   varchar   |     bool    |
|-------------------------------------------------------------------------------------------------------------------------------------------------------------------------------|
|                 ^                                                                                                                                        ^                    |
|                 |                                                                                                                                        |                    |
|      ForeignKey('stations.id')                                                                                                              ForeignKey('stations.station')    |
```

Cette table stocke les valeurs échantillonés avec une plage de **7 jours** elle est basé sur la table `extenso_raw`, ainsi la parsing calcule la moyenne par jour pour les colonnes `displacement` et `speed`, il réucpére également les valeurs enregistrés maxiumum et minimum pour les colonnes `_max` et `_min`.

L'option display est également dans cette table pour pouvoir avoir une géstion complète des données à afficher sur l'application extenso

__Détail des données contenues dans les colonnes:__

- **id** : Primary Key - Identifiant Unique
- **station_id** : Int - Identifiant de la station (lié directement avec `stations.id`)
- **timestamp** : Timestamp - Date & Heure de la donnée
- **displacement** : Float - Contient la valeur moyenne (7 jours)
- **speed** : Float - Contient la valeur moyenne (7 jours)
- **displacement_max** : Float - Contient la valeur **maximale** enregistré (7 jours)
- **displacement_min** : Float - Contient la valeur **minimale** enregistré (7 jours)
- **speed_max** : Float - Contient la valeur **maximale** enregistré (7 jours)
- **speed_min** : Float - Contient la valeur **minimale** enregistré (7 jours)
- **stations** : VarChar - Contient le nom de la station (lié directement avec `stations.station`)
- **display** : Bool - Permet d'afficher ou pas cette donnée (panel administrateur)

### 1.3 Table [settings]

Cette table est composé de 7 colonnes:

```
______________________________________________________________________________________________________________________________________________________
|   id   |   smtp_host   |   smtp_port   |   smtp_user   |   smtp_password   |   sftp_host  |   sftp_user   |   sftp_identity_file   |   sftp_path   |
|----------------------------------------------------------------------------------------------------------------------------------------------------|
|  int   |    varchar    |      int      |    varchar    |      varchar      |   varchar    |    varchar    |         varchar        |    varchar    |
```

Cette table stocke les données d'envrionnement, ces données sont récupéres de la base de données au démarrage du service API, elle permet de modifier les host utilisés dans le service de parsing et aussi dans le service d'envoi de mail, elles peuvent être mises à jour dans le panel administrateur.

Quand une valeur est mise à jour, il n'est pas nécessaire de redémarrer le docker, la modification est prise en compte immédiatement.

__Détail des données contenues dans les colonnes:__

- **id** : Primary Key - Identifiant Unique
- **sftp_host** : varchar - Le host pour la récupération des fichiers `.csv`
- **sftp_user** : varchar - Le nom d'utilisateur SFTP
- **sftp_identity_file** : varchar - Le path de la clé SSH
- **sftp_path** : varchar - Le path du dossier contenant les fichiers des stations (`.csv`)

