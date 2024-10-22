# Configuration du reouteur serveur via Traefik

## Requis
- Docker
- Docker-compose

## Installation
1. Se connecter en ssh sur le serveur VPS (ou local) doté de docker et docker-compose
2. Se rendre dans le dossier du projet ou le cloner sur le serveur `git clone {url}`
3. Créer un network docker pour Traefik `docker network create traefik_network` (si ce n'est pas déjà fait)
4. Créer le fichier `.env` via la commande `cp .env.example .env` et le remplir avec les informations nécessaires
5. Lancer le serveur Traefik via la commande `docker-compose up -d`