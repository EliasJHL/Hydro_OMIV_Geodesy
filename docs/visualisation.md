# Déploiement & Configuration

Le module shiny utilisé est `shinydashboard` qui permet d'avoir l'UI d'un dashboard en utilisant le package `shiny`

L'application actuelle se trouve dans `http://omiv-geodesy.u-strasbg.fr/omiv/hydro/`

## Lancement du docker

L'installation des packages shiny necessaires se fait directement dans le docker compose correspondant.

Il y a un dockerfile par application, tout est géré par le docker compose à la racine.

La différence entre les deux docker compose est l'environnement, chaque applicaiton a son url pointant sur l'API, on peut configurer dans la version de déploiement une URL différente que celle pour le développement.

```yml
# Developpement
x-environnement: &environnement
  API_URL_EXTENSO: http://omiv-geodesy.u-strasbg.fr/extenso-rest-dev/
  API_URL_GNSS: http://omiv-geodesy.u-strasbg.fr/rest/
  API_URL_TACHEO: http://omiv-geodesy.u-strasbg.fr/rest/
  API_URL_HYDRO: http://omiv-geodesy.u-strasbg.fr/hydro-rest-dev/

# Production
x-environnement: &environnement
  API_URL_EXTENSO: http://omiv-geodesy.u-strasbg.fr/extenso-rest/
  API_URL_GNSS: http://omiv-geodesy.u-strasbg.fr/rest/
  API_URL_TACHEO: http://omiv-geodesy.u-strasbg.fr/rest/
  API_URL_HYDRO: http://omiv-geodesy.u-strasbg.fr/hydro-rest/
```

Chaque application a un port attribué en local.

L'attribution des ports (version déploiement)

```
# Ports allocation
# Home 10100
# Gnss 10101
# tacheo 10102
# extenso 10103
# hydro 10105
# panel_extenso 10104
```

Pour lancer le docker compose, il faut effectuer cette commande qui va lancer le fichier `docker-compose-prod.yml`:

```
sudo docker compose -f docker-compose-prod.yml up --build --force-recreate -d
```

### Configuration Ngnix

Il faut une configuration par application pour la redirection

Voilà un exemple pour la configuration des application situés dans `http://omiv-geodesy.u-strasbg.fr/omiv/`


```
server {
        listen 80;
        server_name omiv-geodesy.u-strasbg.fr;

        location /omiv/ {
                rewrite ^/omiv/(.*)$ /$1 break;
                proxy_pass http://127.0.0.1:10100/; # Redirection vers l'application Home
                proxy_redirect / $scheme://$http_host/omiv/home;
                proxy_http_version 1.1;
                proxy_set_header Upgrade $http_upgrade;
                proxy_set_header Connection $connection_upgrade;
                proxy_buffering on;
        }

        location /omiv/home {
        	rewrite ^/omiv/home/(.*)$ /$1 break;
        	proxy_pass http://127.0.0.1:10100/; # Redirection du port configuré en local
        	proxy_redirect / $scheme://$http_host/omiv/home;
        	proxy_http_version 1.1;
        	proxy_set_header Upgrade $http_upgrade;
        	proxy_set_header Connection $connection_upgrade;
        	proxy_buffering on;
        }

        location /omiv/gnss {
                rewrite ^/omiv/gnss/(.*)$ /$1 break;
                proxy_pass http://127.0.0.1:10101/; # Redirection du port configuré en local
                proxy_redirect / $scheme://$http_host/omiv/gnss;
                proxy_http_version 1.1;
                proxy_set_header Upgrade $http_upgrade;
                proxy_set_header Connection $connection_upgrade;
                proxy_buffering on;
        }

        location /omiv/tacheo {
                rewrite ^/omiv/tacheo/(.*)$ /$1 break;
                proxy_pass http://127.0.0.1:10102/; # Redirection du port configuré en local
                proxy_redirect / $scheme://$http_host/omiv/tacheo;
                proxy_http_version 1.1;
                proxy_set_header Upgrade $http_upgrade;
                proxy_set_header Connection $connection_upgrade;
                proxy_buffering on;
        }

        location /omiv/extenso/ {
                rewrite ^/omiv/extenso/(.*)$ /$1 break;
                proxy_pass http://127.0.0.1:10103/; # Redirection du port configuré en local
                proxy_redirect / $scheme://$http_host/omiv/extenso/;
                proxy_http_version 1.1;
                proxy_set_header Upgrade $http_upgrade;
                proxy_set_header Connection $connection_upgrade;
                proxy_buffering on;
        }

        location /omiv/hyro/ {
                rewrite ^/omiv/hydro/(.*)$ /$1 break;
                proxy_pass http://127.0.0.1:10105/; # Redirection du port configuré en local
                proxy_redirect / $scheme://$http_host/omiv/hydro/;
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