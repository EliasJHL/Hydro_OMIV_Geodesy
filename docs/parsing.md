# Documentation du Parsing

Le parsing est un service lancé depuis le service de l'API, il contenu dans le même container, il utilise les variables d'environnement Postgres, SMTP, SFTP.

Pour le moment le script de parsing est lancé deux fois, la première fois permet de récupérer et si nécessaire calculer les données de vélocité, il remplis la table `extenso_raw`.

Le deuxième passage sert à l'échantillonage des données dans `extenso_raw` pour les mettre dans les tables `extenso_day` et `extenso_week`.

Pour lancer le parsing de façon complete un fichier bash se trouve dans le dossier /parsing_service/start.sh pour lancer correctement et de façon complète le parsing.

## Structure

La structure du service de parsing :

```
parsing_service/
    |-- requirements.txt
    |-- start.sh # Script de lancement
    `-- parsing/
        |-- config/
        |   `-- config.yml # Fichier de configuration du parsing des stations
        |-- ssh_key/
        |   `-- # Dossier contenant la clé ssh pour la connexion SFTP
        |-- main.py
        |-- configuration.py
        |-- files_list.py
        `-- parsing.py
```

Lors du lancement de parsing via le script bash `start.sh`, le fichier main.py sera lancé une permière fois, une fois terminé elle sera relancé une dernière fois.

## Environnement

Les paramètres d'environnement utilisés par le parsing sont:

```markdown
**Connexion Postgresql**

- POSTGRES_DB
- POSTGRES_USER
- POSTGRES_PASSWORD
- POSTGRES_PORT
- POSTGRES_HOST

**Connexion SFTP**

- SFTP_HOST
- SFTP_USER
- SFTP_IDENTITY_FILE
- SFTP_PATH
- SFTP_FILE_FORMAT
```

Le détail sur chaque variable d'environnemnt est disponible dans la partie [Environnement](environnement.md)

## Classes

Le service utilise 3 classes pour la communication avec la base de données, le client SFTP et la configuration des stations

### 1.1 Class Configuration

```py
class Configuration:
    def __init__(self): # Configuration des path et autres variables
        self.file_path = os.getenv("CONFIG_FILE") # Récupération dans l'environnement
        self.data = self.get_config()

    def get_config(self):  # Renvoi du contenu du fichier 'config.yml'
        try:
            with open(self.file_path, 'r') as file:
                return yaml.safe_load(file)
        except Exception as e:
            print("Erreur lors de la récupération du \
                  fichier de configuration : ", e)
            exit(84)
        
    def __get__(self, key, default=None):
        return self.data.get(key, default) # Permet de récupérer une valeur
    
    def __getitem__(self, key):
        return self.data.get(key)
```

### 1.2 Class ConnectionHost

Cette partie concerne la connexion avec le serveur distant via SFTP pour récupérer les fichier `.csv` des stations.

```py
class ConnectionHost:
    def __init__(self): # Récupération des variables d'environnement concernant le HOST
        self.host = os.getenv("SFTP_HOST")
        self.user = os.getenv("SFTP_USER")
        self.identity_file = os.getenv("SFTP_IDENTITY_FILE")
        self.path = os.getenv("SFTP_PATH")
        self.file_format = os.getenv("SFTP_FILE_FORMAT")
        self.client = None

    def __get__(self):
        return self.host, self.user, self.identity_file

    def connect_to_server(self): # Tentative de connexion au serveur
        try:
            self.client = paramiko.SSHClient()
            self.client.set_missing_host_key_policy(paramiko.AutoAddPolicy())
            pk = self.identity_file
            self.client.connect(self.host, username=self.user, key_filename=io.StringIO(pk))
        except Exception as e:
            print("Error - connect_to_server -", e)
            exit(84)

    def get_file(self, station): # Permet de récupérer la liste de tous les fichiers disponibles
        try:
            list = []
            stdin, stdout, stderr = self.client.exec_command(f"cd {self.path} && ls")
            files = stdout.readlines()
            for file in files:
                r = re.match(self.file_format, file)
                if r:
                    if r.group(2) == station:
                        return file.strip()
            return None
        except Exception as e:
            print("Error - get_file -", e)
            exit(84)
    
    def content_file(self, file): # Permet de voir le contenu d'un fichier (utilisé pour la récuperation des données)
        try:
            command = f"cat {self.path}/{file}"
            stdin, stdout, stderr = self.client.exec_command(command)
            output = stdout.readlines()
            output = ''.join(output)
            return output
        except Exception as e:
            print("Error - content_file -", e)
            exit(84)
    
    def get_stations_list(self): # Permet de récupérer la liste des stations (via le nom des fichiers)
        try:
            list_sites = []
            stdin, stdout, stderr = self.client.exec_command(f"cd {self.path} && ls")
            output = stdout.readlines()
            for site in output:
                r = re.match(self.file_format, site)
                if r:
                    list_sites.append(r.group(2))
            return list_sites
        except Exception as e:
            print("Error - get_sites_list -", e)
            exit(84)

    def disconnect(self):
        self.client.close()
```

Les stations à parser sont récupérés via le nom des fichiers.

Exemple :

```markdown
Le fichier `HYD_CLAP.csv` -> `CLAP`
```

Gestion d'erreur:

```md
Les cas qui peuvent faire arrêter le programme sont :

- Si la récupération des stations via les fichier est impossible (get_stations_list)
- S'il y a un problème avec la récupération du contenu d'un fichier (content_file)
- Si la connexion au serveur distant est impossible (connect_to_server)
- Si la récupération de la liste des fichiers est impossible (get_file)
```

La plupart de ces erreurs sont provoqués par une erreur de configuration.

### 1.3 Class ConnectionDB

Cette class permet de faire la connexion avec la base de données et d'executer des requêtes.

La connexion se fait directement avec `psycopg2`

```py

class ConnectionDB:
    def __init__(self): # Configuration via les variables d'environnement
        self.db = os.getenv("POSTGRES_DB")
        self.user = os.getenv("POSTGRES_USER")
        self.password = os.getenv("POSTGRES_PASSWORD")
        self.port = os.getenv("POSTGRES_PORT")
        self.host = os.getenv("POSTGRES_HOST")
        self.engine = None

    def connect(self): # Tentative de connexion
        try:
            self.engine = create_engine(
                f"postgresql+psycopg2://{self.user}:{self.password}@{self.host}:{self.port}/{self.db}"
            )
        except Exception as e:
            print("Erreur lors de la connexion à la base de données : ", e)
            exit(84)

    def execute(self, query, do_return): # Permet d'executer une query
        try:
            with self.engine.connect() as conn:
                result = conn.execute(text(query))
                conn.commit()
                if do_return:
                    rows = result.fetchall()
                    return rows
        except Exception as e:
            print("Error - execute -", e)
            exit(84)

    def disconnect(self):
        self.engine.dispose()

```