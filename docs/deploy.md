# Déploiement développeur

Le module shiny utilisé est `shinydashboard` qui permet d'avoir l'UI d'un dashboard en utilisant le package `shiny`

L'application actuelle se trouve dans `http://dev-geodesy.u-strasbg.fr/dev/hydro/`

## Lancement du docker

L'installation des packages shiny necessaires se fait directement dans le docker compose correspondant.

Il y a un dockerfile par application, tout est géré par le docker compose à la racine.

La différence entre les deux docker compose est l'environnement, chaque applicaiton a son url pointant sur l'API, on peut configurer dans la version de déploiement une URL différente que celle pour le développement.

```yml
x-environnement: &environnement
  API_URL_EXTENSO: http://dev-geodesy.u-strasbg.fr/extenso-rest-dev/
  API_URL_GNSS: http://dev-geodesy.u-strasbg.fr/rest/
  API_URL_TACHEO: http://dev-geodesy.u-strasbg.fr/rest/
  API_URL_HYDRO: http://dev-geodesy.u-strasbg.fr/hydro-rest-dev/
```

Chaque application a un port attribué en local.

L'attribution des ports (version déploiement)

```
# Ports allocation
# Home 10000
# Gnss 10001
# tacheo 10002
# extenso 10003
# hydro 10005
# panel_extenso 10004
```

Pour lancer le docker compose, il faut effectuer cette commande qui va lancer le fichier `docker-compose-dev.yml`:

```
sudo docker compose -f docker-compose-dev.yml up --build --force-recreate -d
```

### Configuration Ngnix

Il faut une configuration par application pour la redirection

Voilà un exemple pour la configuration des application situés dans `http://dev-geodesy.u-strasbg.fr/dev/`


```
server {
        listen 80;
        server_name dev-geodesy.u-strasbg.fr;

        location /dev/ {
                rewrite ^/dev/(.*)$ /$1 break;
                proxy_pass http://127.0.0.1:10000/; # Redirection vers l'application Home
                proxy_redirect / $scheme://$http_host/dev/home;
                proxy_http_version 1.1;
                proxy_set_header Upgrade $http_upgrade;
                proxy_set_header Connection $connection_upgrade;
                proxy_buffering on;
        }

        location /dev/home {
        	rewrite ^/dev/home/(.*)$ /$1 break;
        	proxy_pass http://127.0.0.1:10000/; # Redirection du port configuré en local
        	proxy_redirect / $scheme://$http_host/dev/home;
        	proxy_http_version 1.1;
        	proxy_set_header Upgrade $http_upgrade;
        	proxy_set_header Connection $connection_upgrade;
        	proxy_buffering on;
        }

        location /dev/gnss {
                rewrite ^/dev/gnss/(.*)$ /$1 break;
                proxy_pass http://127.0.0.1:10001/; # Redirection du port configuré en local
                proxy_redirect / $scheme://$http_host/dev/gnss;
                proxy_http_version 1.1;
                proxy_set_header Upgrade $http_upgrade;
                proxy_set_header Connection $connection_upgrade;
                proxy_buffering on;
        }

        location /dev/tacheo {
                rewrite ^/dev/tacheo/(.*)$ /$1 break;
                proxy_pass http://127.0.0.1:10002/; # Redirection du port configuré en local
                proxy_redirect / $scheme://$http_host/dev/tacheo;
                proxy_http_version 1.1;
                proxy_set_header Upgrade $http_upgrade;
                proxy_set_header Connection $connection_upgrade;
                proxy_buffering on;
        }

        location /dev/extenso/ {
                rewrite ^/dev/extenso/(.*)$ /$1 break;
                proxy_pass http://127.0.0.1:10003/; # Redirection du port configuré en local
                proxy_redirect / $scheme://$http_host/dev/extenso/;
                proxy_http_version 1.1;
                proxy_set_header Upgrade $http_upgrade;
                proxy_set_header Connection $connection_upgrade;
                proxy_buffering on;
        }

        location /dev/hydro/ {
                rewrite ^/dev/hydro/(.*)$ /$1 break;
                proxy_pass http://127.0.0.1:10005/; # Redirection du port configuré en local
                proxy_redirect / $scheme://$http_host/dev/hydro/;
                proxy_http_version 1.1;
                proxy_set_header Upgrade $http_upgrade;
                proxy_set_header Connection $connection_upgrade;
                proxy_buffering on;
        }
}
```

### Accès aux logs

Pour l'accès aux logs il y a la commande `docker logs` qui permet de voir ce que le programme renvoie comme information ou message d'erreur.

Utilisation:

```bash
sudo docker logs service_name
```