## Home Media Server
This is a Media Stack for your Home Server using Docker Compose.

- [Sonarr](https://sonarr.tv/) is a PVR for Usenet and BitTorrent users. It can monitor multiple RSS feeds for new episodes of your favorite shows and will grab, sort and rename them. It can also be configured to automatically upgrade the quality of files already downloaded when a better quality format becomes available.
- [Radarr](https://radarr.video/) is like Sonarr, but for movies.
- [Bazarr](https://www.bazarr.media/) is a companion application to Sonarr and Radarr that manages and downloads subtitles based on your requirements.
- [Prowlarr](https://prowlarr.com/) is an indexer manager/proxy built on the popular *arr .net/reactjs base stack to integrate with your various PVR apps. Prowlarr supports management of both Torrent Trackers and Usenet Indexers. It integrates seamlessly with Lidarr, Mylar3, Radarr, Readarr, and Sonarr offering complete management of your indexers with no per app Indexer setup required.
- [Recyclarr](https://github.com/recyclarr/recyclarr) is a command-line application that will automatically synchronize recommended settings from the [TRaSH guides](https://trash-guides.info/) to your Sonarr/Radarr instances.
- [Jellyfin](https://jellyfin.org/) enables you to collect, manage, and stream your media.
- [Jellyseerr](https://github.com/Fallenbagel/jellyseerr) is a free and open source software application for managing requests for your media library. It is a fork of Overseerr built to bring support for Jellyfin & Emby media servers.

## Setup
Create an .env file replacing the examples from .env.example with the desired values.

The $HOSTNAME property will be used to create the [Traefik Proxy](https://doc.traefik.io/traefik/) routes. For example: http://traefik.home.

## AdGuard Home

[AdGuard Home](https://github.com/AdguardTeam/AdGuardHome) is a privacy protection center for you and your devices, it will be our private DNS which will redirect the requests to our server.

For configuring the DNS you will need to:
1. Reserve an address on your router for the server.
2. Configure the reserved address as your primary DNS provider on your router.

:warning: Before running for the first time you will need to update adguardhome/docker-compose.yml exposing the port 3000.

Start the containers running the command `docker compose up -d` in your terminal from the root directory of this project. This will create the initial config files.

Access http://localhost:3000 and set your password. You can remove the exposed port after the initial configuration.

Configure the filtering rewrites for your $HOSTNAME:
1. Open adguardhome/config/AdGuardHome.yaml.
2. Update the config to the desired $HOSTNAME and local IP address. Leave the wildcard *. since we will use subdomains to redirect to the multiple applications that we will run. For example http://sonarr.home.

```
filtering:
  rewrites:
    - domain: '*.home'
      answer: Local server IP address, for example 192.168.1.100
```
## qBittorent

1. Open the container logs to get the generated password with `sudo docker logs qbittorrent`
2. Access torrent.$HOSTNAME and login with the credentials.

### Web UI
- Update the password.
- Enable `Bypass authentication for clients in whitelisted IP subnets` and add `10.11.0.0/16`

### Downloads
- Set the Default Save Path to `/data/downloads`.

### Additional configuration
- Check [TRaSH Guides](https://trash-guides.info/Downloaders/qBittorrent/Basic-Setup/) for additional configuration.

## Sonarr / Radarr / Readarr / Prowlarr

If you need detailed information about the *arr stack check the [Servarr Wiki](https://wiki.servarr.com/).

All *arr apps will request to configure an authentication. To disable it, once the /config directory is created you can edit the config/config.xml file with:
```
  <AuthenticationMethod>External</AuthenticationMethod>
```

- Retrieve the API keys from config/config.xml and update the .env file
  - After updating the .env run the command `docker compose up -d` again to recreate the containers that use it. This will not automatically configure the *arr apps with each other. That needs to be done through the interface.
      
- Configure the qBittorent as the Download Client in *arr apps.
  - Since the containers are in the same network you can referece each application by the container name and port. For example: http://qbittorrent:8080. This will avoid unnecessary requests to the DNS.

### Prowlarr
- Configure the Sonarr/Radarr/Readarr instances for full sync.
- Configure the providers in Prowlarr. Some of the providers will require Flaresolverr to work.
  - Follow the steps in [TRaSH Guides](https://trash-guides.info/Prowlarr/prowlarr-setup-flaresolverr/) on how to configure it.

### Bazarr
- Configure the Sonarr/Radarr instances.

## Homepage

Access http://get.$HOSTNAME/ to access the dashboard. Ex: http://get.home/.

## Portainer

If you are using [Docker Desktop](https://docs.docker.com/desktop/) you can remove the include from [docker-compose.yml](docker-compose.yml). 

Since it's not recommend by the [Servarr Docker Guide](https://wiki.servarr.com/docker-guide) for setting up docker containers, use it only to check the container status and logs through the web interface.

## OS

- [Suggested OS: Ubuntu Server](https://ubuntu.com/download/server)

- [Install Docker Engine on Ubuntu](https://docs.docker.com/engine/install/ubuntu/)
  - Do not select the Docker software during the OS setup.

- [Ubuntu: How To Free Up Port 53, Used By systemd-resolved](https://www.linuxuprising.com/2020/07/ubuntu-how-to-free-up-port-53-used-by.html)

- [Auto Mount Drive in Ubuntu Server 22.04 at Startup](https://developerinsider.co/auto-mount-drive-in-ubuntu-server-22-04-at-startup/)
