# Construire un premier homelab de A a Z

Guide pratique pour monter un homelab maison simple, fiable et evolutif avec Proxmox VE, ZFS, Ubuntu Server, Docker Compose, sauvegardes et acces distant securise.

Ce guide vise un premier serveur personnel, pas une plateforme d'entreprise. Les choix privilegient la clarte, la recuperation facile et l'apprentissage progressif.

## Architecture cible

```text
Serveur physique
└── Proxmox VE 9.x (bare-metal, Debian 13/Trixie)
    ├── Stockage ZFS sur l'hote Proxmox
    ├── VM Ubuntu Server 24.04 LTS principale
    │   └── Docker + Docker Compose
    │       ├── Portainer
    │       ├── Nginx Proxy Manager
    │       ├── AdGuard Home ou Pi-hole
    │       ├── Uptime Kuma
    │       ├── Vaultwarden
    │       ├── Jellyfin
    │       ├── Syncthing
    │       └── Gitea
    ├── VM Home Assistant
    └── VM Linux de test
```

## Documents

1. [Materiel recommande](docs/01-hardware.md)
2. [Proxmox, ZFS et VM Ubuntu](docs/02-proxmox.md)
3. [Docker Compose et services](docs/03-docker.md)
4. [Securite minimale](docs/04-security.md)
5. [Reseau, DNS et acces distant](docs/05-network.md)
6. [Backups et restauration](docs/06-backups.md)
7. [Maintenance et depannage](docs/07-maintenance.md)

## Parcours recommande

1. Choisir le materiel et valider les disques avant installation.
2. Installer Proxmox VE sur le SSD/NVMe.
3. Creer le miroir ZFS sur les disques de donnees.
4. Creer la VM Ubuntu principale avec un disque systeme et un disque de donnees.
5. Monter le disque de donnees Ubuntu sous `/srv/data`.
6. Installer Docker Engine et Docker Compose dans Ubuntu.
7. Installer Portainer, Nginx Proxy Manager, DNS local et monitoring.
8. Mettre en place les backups avant les services sensibles.
9. Installer Vaultwarden, Jellyfin, Syncthing, Gitea ou autres services selon les besoins.
10. Documenter les IP, ports, volumes, backups et procedures de restauration.

## Choix structurants

- Installe Proxmox directement sur le serveur physique.
- Garde l'hote Proxmox minimal: pas de Docker directement sur Proxmox pour un premier setup.
- Fais tourner les services applicatifs dans une VM Ubuntu dediee.
- Mets le pool ZFS sur l'hote Proxmox, puis expose le stockage a la VM sous forme de disque virtuel.
- Monte le disque de donnees dans Ubuntu sous `/srv/data`.
- Place les stacks Docker sous `/opt/docker`.
- Utilise Tailscale pour l'acces distant au debut.
- N'ouvre aucun port Internet tant que les sauvegardes, mises a jour, mots de passe et restaurations ne sont pas maitrises.

## Versions recommandees en 2026

- Proxmox VE 9.x pour une nouvelle installation.
- Ubuntu Server 24.04 LTS pour la VM Docker principale, car elle reste stable et largement documentee.
- Ubuntu Server 26.04 LTS peut etre choisi plus tard, apres validation de compatibilite avec tes images Docker et tes habitudes d'administration.
- Docker Engine installe depuis le depot officiel Docker, avec le plugin `docker-compose-plugin`.

## Resume rapide

Configuration conseillee pour commencer:

- Petite tour ou desktop business evolutif.
- CPU Intel i5/i7 8e generation ou plus recent, ou AMD Ryzen 5/7 equivalent.
- 32 Go de RAM minimum, 64 Go si le budget le permet.
- 1 SSD/NVMe pour Proxmox et les disques systeme des VM.
- 2 HDD NAS CMR en miroir ZFS pour les donnees.
- 1 disque USB externe pour les backups.
- 1 UPS line-interactive.
- Tailscale pour l'acces distant.
- Uptime Kuma pour le monitoring simple.

## Sources officielles utiles

- Proxmox VE 9.0 est base sur Debian 13 Trixie: https://www.proxmox.com/en/about/company-details/press-releases/proxmox-virtual-environment-9-0
- Depots Proxmox VE: https://pve.proxmox.com/pve-docs/pve-package-repos-plain.html
- Installation Docker Engine sur Ubuntu: https://docs.docker.com/engine/install/ubuntu/
- Cycle de vie Ubuntu: https://ubuntu.com/about/release-cycle
- Installation Tailscale Linux: https://tailscale.com/docs/install/linux
