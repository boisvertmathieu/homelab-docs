# 03 - Docker Compose et services

## Role de Docker

Docker sert a faire tourner les services applicatifs dans la VM Ubuntu principale.

Objectif:

- Garder Proxmox propre.
- Installer les services de facon reproductible.
- Centraliser les configurations.
- Faciliter les backups des donnees persistantes.

## Installation officielle sur Ubuntu

Dans la VM Ubuntu:

```bash
sudo apt update
sudo apt install -y ca-certificates curl gnupg
sudo install -m 0755 -d /etc/apt/keyrings
curl -fsSL https://download.docker.com/linux/ubuntu/gpg | sudo gpg --dearmor -o /etc/apt/keyrings/docker.gpg
sudo chmod a+r /etc/apt/keyrings/docker.gpg
```

Ajouter le depot Docker:

```bash
echo \
  "deb [arch=$(dpkg --print-architecture) signed-by=/etc/apt/keyrings/docker.gpg] https://download.docker.com/linux/ubuntu \
  $(. /etc/os-release && echo "$VERSION_CODENAME") stable" | \
  sudo tee /etc/apt/sources.list.d/docker.list > /dev/null
```

Installer:

```bash
sudo apt update
sudo apt install -y docker-ce docker-ce-cli containerd.io docker-buildx-plugin docker-compose-plugin
```

Verifier:

```bash
sudo docker version
sudo docker compose version
```

Ajouter ton utilisateur au groupe Docker:

```bash
sudo usermod -aG docker $USER
newgrp docker
```

Attention: appartenir au groupe `docker` donne un niveau de privilege eleve sur la machine. Ne l'accorde qu'a un compte admin de confiance.

## Structure de fichiers

Structure recommandee:

```text
/opt/docker/
├── adguard/
│   ├── docker-compose.yml
│   ├── work/
│   └── conf/
├── gitea/
│   ├── docker-compose.yml
│   └── data/
├── jellyfin/
│   ├── docker-compose.yml
│   ├── config/
│   └── cache/
├── nginx-proxy-manager/
│   ├── docker-compose.yml
│   ├── data/
│   └── letsencrypt/
├── portainer/
│   ├── docker-compose.yml
│   └── data/
├── syncthing/
│   ├── docker-compose.yml
│   └── config/
├── uptime-kuma/
│   ├── docker-compose.yml
│   └── data/
└── vaultwarden/
    ├── docker-compose.yml
    └── data/
```

Creation:

```bash
sudo mkdir -p /opt/docker
sudo chown -R $USER:$USER /opt/docker
mkdir -p /opt/docker/{portainer,nginx-proxy-manager,adguard,uptime-kuma,vaultwarden,jellyfin,syncthing,gitea}
```

## Principes Compose

Un `docker-compose.yml` decrit:

- Les images.
- Les noms de conteneurs.
- Les ports publies.
- Les volumes persistants.
- Les variables d'environnement.
- La politique de redemarrage.

Exemple minimal:

```yaml
services:
  app:
    image: image:tag
    container_name: app
    restart: unless-stopped
    ports:
      - "8080:80"
    volumes:
      - ./data:/data
    environment:
      - TZ=America/Toronto
```

## Ordre d'installation conseille

1. Portainer, si tu veux une interface de gestion.
2. Nginx Proxy Manager pour des URL internes propres.
3. AdGuard Home ou Pi-hole pour le DNS local.
4. Uptime Kuma pour savoir si les services repondent.
5. Vaultwarden seulement apres avoir valide backups et securite.
6. Jellyfin, Syncthing, Gitea ou autres services selon tes besoins.

## Portainer

Usage:

- Interface web pour Docker.
- Pratique pour inspecter les conteneurs, logs et volumes.

Precautions:

- Acces LAN ou Tailscale seulement.
- Mot de passe admin fort.
- Ne pas exposer sur Internet.

```yaml
services:
  portainer:
    image: portainer/portainer-ce:latest
    container_name: portainer
    restart: unless-stopped
    ports:
      - "9443:9443"
    volumes:
      - /var/run/docker.sock:/var/run/docker.sock
      - ./data:/data
    environment:
      - TZ=America/Toronto
```

Demarrer:

```bash
cd /opt/docker/portainer
docker compose up -d
```

## Nginx Proxy Manager

Usage:

- Reverse proxy simple.
- URL internes propres.
- TLS plus facile plus tard.

Precautions:

- L'interface admin sur le port `81` doit rester privee.
- N'ouvre pas `80` et `443` vers Internet au debut.

```yaml
services:
  npm:
    image: jc21/nginx-proxy-manager:latest
    container_name: nginx-proxy-manager
    restart: unless-stopped
    ports:
      - "80:80"
      - "81:81"
      - "443:443"
    volumes:
      - ./data:/data
      - ./letsencrypt:/etc/letsencrypt
    environment:
      - TZ=America/Toronto
```

## AdGuard Home

Usage:

- DNS local.
- Blocage de pubs et trackers.
- Enregistrements internes comme `vault.home.arpa`.

Precautions:

- Teste d'abord sur un seul client.
- Garde un DNS de secours.
- Ne force pas tout le LAN avant validation.

```yaml
services:
  adguardhome:
    image: adguard/adguardhome:latest
    container_name: adguardhome
    restart: unless-stopped
    ports:
      - "53:53/tcp"
      - "53:53/udp"
      - "3000:3000/tcp"
    volumes:
      - ./work:/opt/adguardhome/work
      - ./conf:/opt/adguardhome/conf
    environment:
      - TZ=America/Toronto
```

## Pi-hole

Alternative a AdGuard Home.

```yaml
services:
  pihole:
    image: pihole/pihole:latest
    container_name: pihole
    restart: unless-stopped
    ports:
      - "53:53/tcp"
      - "53:53/udp"
      - "8081:80/tcp"
    environment:
      - TZ=America/Toronto
      - FTLCONF_webserver_api_password=change-me
    volumes:
      - ./etc-pihole:/etc/pihole
      - ./etc-dnsmasq.d:/etc/dnsmasq.d
```

## Uptime Kuma

Usage:

- Verification HTTP.
- Pings.
- Alertes simples.

```yaml
services:
  uptime-kuma:
    image: louislam/uptime-kuma:1
    container_name: uptime-kuma
    restart: unless-stopped
    ports:
      - "3001:3001"
    volumes:
      - ./data:/app/data
    environment:
      - TZ=America/Toronto
```

## Vaultwarden

Usage:

- Gestionnaire de mots de passe auto-heberge compatible Bitwarden.

Precautions:

- Service tres sensible.
- Pas d'exposition Internet directe au debut.
- Acces via LAN ou Tailscale.
- Backups testes avant usage reel.
- `SIGNUPS_ALLOWED=false` apres creation de ton compte.

```yaml
services:
  vaultwarden:
    image: vaultwarden/server:latest
    container_name: vaultwarden
    restart: unless-stopped
    ports:
      - "8082:80"
    volumes:
      - ./data:/data
    environment:
      - TZ=America/Toronto
      - SIGNUPS_ALLOWED=false
```

## Jellyfin

Usage:

- Serveur multimedia personnel.

Precautions:

- Les medias lourds peuvent consommer beaucoup d'espace.
- Le transcodage peut consommer CPU/GPU.
- Sauvegarde seulement les medias qui ont une vraie valeur.

```yaml
services:
  jellyfin:
    image: jellyfin/jellyfin:latest
    container_name: jellyfin
    restart: unless-stopped
    ports:
      - "8096:8096"
    volumes:
      - ./config:/config
      - ./cache:/cache
      - /srv/data/media:/media
    environment:
      - TZ=America/Toronto
```

## Syncthing

Usage:

- Synchronisation de fichiers entre appareils.

Precautions:

- La synchronisation n'est pas une sauvegarde.
- Une suppression peut etre synchronisee partout.
- Active le versioning Syncthing pour les dossiers importants.

```yaml
services:
  syncthing:
    image: lscr.io/linuxserver/syncthing:latest
    container_name: syncthing
    restart: unless-stopped
    ports:
      - "8384:8384"
      - "22000:22000/tcp"
      - "22000:22000/udp"
      - "21027:21027/udp"
    volumes:
      - ./config:/config
      - /srv/data/documents:/documents
      - /srv/data/photos:/photos
    environment:
      - PUID=1000
      - PGID=1000
      - TZ=America/Toronto
```

## Gitea

Usage:

- Git auto-heberge leger.

Precautions:

- Restreins les inscriptions.
- Sauvegarde `data`.
- Evite l'exposition publique au debut.

```yaml
services:
  gitea:
    image: gitea/gitea:latest
    container_name: gitea
    restart: unless-stopped
    ports:
      - "3000:3000"
      - "2222:22"
    volumes:
      - ./data:/data
    environment:
      - USER_UID=1000
      - USER_GID=1000
      - TZ=America/Toronto
```

## Commandes utiles

Demarrer une stack:

```bash
docker compose up -d
```

Voir les conteneurs:

```bash
docker compose ps
```

Voir les logs:

```bash
docker compose logs -f
```

Mettre a jour une stack:

```bash
docker compose pull
docker compose up -d
```

Voir l'espace Docker:

```bash
docker system df
```

Nettoyer les images inutilisees:

```bash
docker image prune -a
```

Ne lance pas de nettoyage destructif avant d'avoir compris ce qui sera supprime.
