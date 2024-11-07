# Documentation Hydro


L’application est divisé en deux parties différentes : Le module de calcul écrit en Python hébergé sur le serveur et le module de visualisation écrit en R

Le module de calcul traite les données disponible dans des fichier, les parse et les stocke dans une base de données Postgresql

Le module de visualisation récupère ces données pour afficher des cartes et des graphiques via un serveur web, ce module permet également de télécharger les données brutes et traitées d’un site au format CSV.

## Structure du projet

    | 
    |-- app.R
    |-- DESCRIPTION
    |-- inst
    |     `-- app
    |          `-- www
    |               |-- img
    |               |    `-- # Images des sites - Format PNG
    |               |-- favicon.ico
    |               |-- main.css
    |               `-- tiles_config.json # Map tiles configuration file
    |-- R
    |   `-- # R scripts
    |-- dev
        `-- # Golem files


Pour ce qui concerne l’application, le contact est **Xavier WANNER** `x.wanner@quadrilaterre.fr` qui en est l’auteur.