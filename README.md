# Setting up Media Management Services on Synology NAS with Docker and Portainer

## Introduction

Setting up media management services on your Synology NAS can greatly enhance your media consumption experience. In this guide, we'll walk through the installation process for Sonarr, Radarr, Lidarr, and other essential services using Docker and Portainer. The instructions assume that you have already installed Portainer on your Synology NAS.

## Prerequisites

Install Portainer: Follow the guide How to Install Portainer on Your Synology NAS to set up Portainer:
https://mariushosting.com/how-to-install-portainer-on-your-synology-nas/

## Create a Dedicated User and Group:
Utilize the DSM Control Panel to create a dedicated user and group for the Docker containers.

## SSH Connection for User Information:
Connect to your NAS using SSH to obtain the user information. Use the command:
```
id User
```
(replace "User" with the newly created user). Note the values for UID and GID returned by the command.


## Create Necessary Folders:
In the shared docker folder, create the following directory structure:
```
docker/
  |
  |_portainer/
  |_overseerr/
  |_lidarr/
  |_radarr/
  |_sonarr/
  |_prowlarr/
  |_flaresolverr/
  |_rdtclient/
      |
      |_downloads/
            |
            |_lidar/
            |_radarr/
            |_sonarr/
```
Create a multimedia folder with the structure (or adapt the docker-compose file to your need):
```
multimedia/
  |
  |_films/
  |_series/
  |_musiques/
```

## Docker Compose File:
Create a new stack file under Portainer's Stack section with the following content. Replace PGID and PUID with the values obtained from the SSH command.
```
version: '3.3'
services:
    overseerr:
        container_name: overseerr
        volumes:
            - /volume1/docker/overseerr:/config
        image: ghcr.io/linuxserver/overseerr
        restart: always
        logging:
            driver: json-file
            options:
                max-size: 10m
        ports:
            - '5055:5055'
        environment:
            - PGID=100
            - PUID=1028
    lidarr:
        container_name: lidarr
        volumes:
            - /volume1/docker/lidarr:/config
            - /volume1/multimedias:/multimedias
            - /volume1/docker/rdtclient/downloads:/downloads
        image: ghcr.io/linuxserver/lidarr:develop
        restart: always
        logging:
            driver: json-file
            options:
                max-size: 10m
        ports:
            - '8686:8686'
        environment:
            - PGID=100
            - PUID=1028
    radarr:
        container_name: radarr
        volumes:
            - /volume1/docker/radarr:/config
            - /volume1/multimedias:/multimedias
            - /volume1/docker/rdtclient/downloads:/downloads
        image: ghcr.io/linuxserver/radarr:develop
        restart: always
        logging:
            driver: json-file
            options:
                max-size: 10m
        ports:
            - '7878:7878'
        environment:
            - PGID=100
            - PUID=1028
    sonarr:
        container_name: sonarr
        volumes:
            - /volume1/docker/sonarr:/config
            - /volume1/multimedias:/multimedias
            - /volume1/docker/rdtclient/downloads:/downloads
        image: ghcr.io/linuxserver/sonarr:develop
        restart: always
        logging:
            driver: json-file
            options:
                max-size: 10m
        ports:
            - '8989:8989'
        environment:
            - PGID=100
            - PUID=1028
    prowlarr:
        container_name: prowlarr
        volumes:
            - /volume1/docker/prowlarr:/config
        image: ghcr.io/linuxserver/prowlarr:develop
        restart: always
        logging:
            driver: json-file
            options:
                max-size: 10m
        environment:
            - PGID=100
            - PUID=1028
        ports:
            - '9696:9696'
    flaresolverr:
        container_name: flaresolverr
        volumes:
            - /volume1/docker/flaresolverr:/config
        image: ghcr.io/flaresolverr/flaresolverr:latest
        restart: always
        logging:
            driver: json-file
            options:
                max-size: 10m
        ports:
            - '8191:8191'
        environment:
            - PGID=100
            - PUID=1028
    rdtclient:
        container_name: rdtclient
        volumes:
            - /volume1/docker/rdtclient/downloads:/data/downloads
            - /volume1/docker/rdtclient:/data/db
        image: rogerfar/rdtclient
        restart: always
        logging:
            driver: json-file
            options:
                max-size: 10m
        ports:
            - '6500:6500'
        environment:
            - PGID=100
            - PUID=1028
```
If you don't want some services like jellyfin, simply delete jellyfin section (Radarr(Movies) or Sonarr(Series) or Lidarr(Music) and Rdtclient(download manager); Prowlarr(indexers) are important)

## Configure Rdtclient:
Configuration details for Rdtclient will be provided later in this guide.
....

## Radarr Configuration:
### Configure Prowlarr:
Obtain the Radarr API key from Settings → General → API key.
Radarr API Key
In Prowlarr, navigate to Settings → Apps and add the Radarr configuration to synchronize indexers. (You can also add some indexers into prowlarr)
