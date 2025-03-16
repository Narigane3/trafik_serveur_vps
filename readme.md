# Configuration du reouteur serveur via Traefik

[![Deploy to VPS](https://github.com/Narigane3/trafik_serveur_vps/actions/workflows/deploy.yml/badge.svg)](https://github.com/Narigane3/trafik_serveur_vps/actions/workflows/deploy.yml)

## Table des matières
- [Description](#description)
- [Information](#information)
- [Requis](#requis)
- [Installation](#installation)
- [Configuration des services](#configuration-des-services)
- [Configuration MDP pour le dashboard de Traefik](#configuration-mdp-pour-le-dashboard-de-traefik)
- [Accès à l'interface Dashboard de Traefik](#accès-à-linterface-dashboard-de-traefik)
- [Accès au logs de Traefik](#accès-au-logs-de-traefik)

## Description
Ce projet permet de configurer un serveur VPS en tant que routeur pour rediriger les requêtes entrantes vers les différents services docker. Il utilise Traefik comme reverse proxy et permet de gérer les différents services via une interface web. 

Il s'occupe également de générer automatiquement les certificats SSL via Let's Encrypt pour chaque service.

> **Note:** Nobilier pas de faire pointer votre nom de domaine depuis votre serveur DNS vers l'adresse IP de votre serveur VPS

# Information
Le dashboard de Traefik est accessible via le domain `${TRAEFIK_DOMAIN}` et le port `${TRAEFIK_PROT}` (ex: `traefik.domaine.com:8080`)

## Requis
- Docker
- Docker-compose
- Un serveur VPS
- Un nom de domaine
- apache2-utils (pour générer les mots de passe) `sudo apt install apache2-utils`

## Installation
1. Se connecter en ssh sur le serveur VPS doté de docker et docker-compose
2. Se rendre dans le dossier du projet ou le cloner sur le serveur `git clone {url}`
3. Créer un network docker pour Traefik `docker network create traefik_network` (si ce n'est pas déjà fait)
4. Créer un mot de passe pour l'interface de Traefik `htpasswd -nb admin password` et copier le résultat remplacer `admin`par le nom d'utilisateur souhaité et `password` par le mot de passe souhaité
4. Créer le fichier `.env` via la commande `cp .env.example .env` et le remplir avec les informations nécessaires (`DOMAIN`, `TRAEFIK_DOMAIN`,  `BASIC_AUTH`, `EMAIL`)
5. Lancer le serveur Traefik via la commande `docker-compose up -d`

## Configuration des services
Pour ajouter un service à Traefik, il suffit de rajouter les labels suivants dans le fichier `docker-compose.yml` du service:
```yml
labels:
  - "traefik.enable=true"
  - "traefik.http.routers.{NOM_DU_SERVICE}.rule=Host(`{NOM_DU_SERVICE}.${DOMAIN}`)"
  - "traefik.http.routers.{NOM_DU_SERVICE}.entrypoints=websecure"
  - "traefik.http.routers.{NOM_DU_SERVICE}.tls=true"
  - "traefik.http.routers.{NOM_DU_SERVICE}.tls.certresolver=letsencrypt"
  - "traefik.http.services.{NOM_DU_SERVICE}.loadbalancer.server.port={PORT_DU_SERVICE}"
  # Redirect to https
  - "traefik.http.middlewares.myapp-https-redirect.redirectscheme.scheme=https"
```
sans oublier d'inclure le service dans le réseau `traefik_network`:
```yml
   networks:
      - traefik_network
networks:
  traefik_network:
    external: true
```

## Configuration MDP pour le dashboard de Traefik
Le mot de passe pour l'interface de Traefik est généré via la commande `htpasswd -nb admin password` et doit être ajouté dans le fichier `.env` sous la forme `BASIC_AUTH='admin:password'`

> **Note:** si la commande `htpasswd` n'est pas reconnue, il faut installer le paquet `apache2-utils` via la commande `sudo apt install apache2-utils`


## Accès à l'interface Dashboard de Traefik
L'interface de Traefik est accessible via le domain `${TRAEFIK_DOMAIN}` en HTTPS (ex: `traefik.domaine.com/dashboard/#/`)

## Accès au logs de Traefik
Les access logs de Traefik sont activer et sont accessibles dans le logs rep `./logs/access.log`
