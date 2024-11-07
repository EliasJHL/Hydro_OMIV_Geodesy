# Configuration des environnements

[À venir] Les variables d'environnement SFTP & SMTP sont stockés directement dans la base de données pour que l'adminisateur puisse éditer les connexions ainsi que la configuration directement via le panel sans passer par l'édition du docker compose à la main. Les modifications, une fois validés, sont configurés et utilisés par les nombreux services instanatnément sans avoir besoin de redémarrer le container.

## Configuration des docker compose

Deux fichiers docker compose sont disponibles, une pour la version de développement et une autre pour la version de production.

La différence entre les deux fichiers est l'environnement défini ainsi que les ports pour qu'il puissent fonctionner simultanément sur la même machine sans intérférer l'une avec l'autre.

La configuration des base de données et des connexions est faite directement dans l'environnement contenu dans chacun des deux fichiers.

Voià un détail de l'utilisation de l'environnement au sein du docker.

```
x-environnement: &environnement
  # Environnement utilisé par le service db_extenso
  POSTGRES_DB: dev-omiv-extenso                # Base de données
  POSTGRES_USER: user                          # Utilisateur
  POSTGRES_PASSWORD: password                  # Mot de passe
  POSTGRES_PORT: 5432                          # Ne pas changer le port
  POSTGRES_HOST: db_extenso_dev                # Host -> Nom du service
  # Sftp Connection Environment
  SFTP_HOST: IP adress                         # Host SFTP
  SFTP_USER: debian                            # Utilisateur SFTP
  SFTP_IDENTITY_FILE: ./ssh_key/api_extenso    # Clé SSH privé
  SFTP_PATH: /tmp/public/EXTENSO               # Chemin des fichiers sur le serveur distant
  SFTP_FILE_FORMAT: "(\\w+)_(\\w+)\\.csv"      # Format des fichiers
```

La configuration des variables d'environnement (uniquement pour la verison de production) sont éditables directement sur le panel administrateur où il est possible de vérifier les connexions ainsi que le bon fonctionnement.
