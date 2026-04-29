# 02 - Proxmox, ZFS et VM Ubuntu

## Role de Proxmox

Proxmox VE est l'hyperviseur principal. Il doit rester propre, previsible et facile a restaurer.

Il sert a:

- Gerer les VM et conteneurs LXC.
- Gerer le stockage local.
- Executer les backups de VM.
- Fournir une interface web d'administration.
- Isoler les services applicatifs du serveur physique.

Evite d'installer Docker directement sur Proxmox pour un premier homelab. Docker doit vivre dans une VM Ubuntu dediee.

## Installation de Proxmox

Pour une nouvelle installation en 2026, pars sur Proxmox VE 9.x.

Preparation:

- Telecharge l'ISO Proxmox VE depuis le site officiel.
- Ecris l'ISO sur une cle USB avec Ventoy, Rufus ou balenaEtcher.
- Branche le serveur en Ethernet.
- Branche le serveur sur l'UPS si disponible.
- Verifie dans le BIOS que le boot UEFI et la virtualisation sont actifs.

Parametres de base:

- Installer Proxmox sur le SSD/NVMe.
- Utiliser une IP fixe pour l'hote, par exemple `192.168.1.10`.
- Choisir un hostname clair, par exemple `pve1.home.arpa`.
- Activer la virtualisation dans le BIOS: Intel VT-x ou AMD-V.
- Activer IOMMU/VT-d si disponible, surtout pour un futur passthrough.

Interface web:

```text
https://192.168.1.10:8006
```

Le certificat auto-signe est normal au debut.

## Depots Proxmox

Proxmox active par defaut le depot entreprise. Sans abonnement, il faut soit le desactiver, soit accepter les erreurs `401 Unauthorized` pendant `apt update`.

### Proxmox VE 9.x

Proxmox VE 9 utilise Debian 13 Trixie et le format de depots Deb822.

Depot no-subscription:

```bash
cat > /etc/apt/sources.list.d/proxmox.sources <<'EOF'
Types: deb
URIs: http://download.proxmox.com/debian/pve
Suites: trixie
Components: pve-no-subscription
Signed-By: /usr/share/keyrings/proxmox-archive-keyring.gpg
EOF
```

Si `/etc/apt/sources.list.d/pve-enterprise.sources` existe et que tu n'as pas d'abonnement, commente ses lignes ou deplace le fichier hors de `sources.list.d`.

Ensuite:

```bash
apt update
apt full-upgrade -y
```

Verifie aussi qu'un ancien depot entreprise n'est pas encore actif dans:

```bash
ls /etc/apt/sources.list.d/
```

### Proxmox VE 8.x

Si tu installes ou maintiens encore Proxmox VE 8, les exemples `bookworm` s'appliquent. Ne melange pas `bookworm` et `trixie` sur la meme installation.

## Strategie ZFS

Le pool ZFS vit sur l'hote Proxmox.

Principe important:

- Ne monte pas directement `/tank/...` dans la VM Ubuntu pour debuter.
- Utilise le pool ZFS comme backend de stockage Proxmox.
- Ajoute un disque virtuel a la VM Ubuntu depuis ce stockage.
- Dans Ubuntu, monte ce disque sous `/srv/data`.

Cette approche garde une separation nette entre l'hyperviseur et l'OS applicatif.

## Creer un miroir ZFS

Identifie les disques:

```bash
lsblk -o NAME,SIZE,TYPE,MODEL,SERIAL
ls -l /dev/disk/by-id/
```

Utilise toujours les chemins `/dev/disk/by-id/`, pas `/dev/sdb` ou `/dev/sdc`, car ces noms peuvent changer.

Exemple:

```bash
zpool create -f \
  -o ashift=12 \
  -O compression=lz4 \
  -O atime=off \
  -O xattr=sa \
  -O acltype=posixacl \
  tank mirror \
  /dev/disk/by-id/ata-DISQUE-1 \
  /dev/disk/by-id/ata-DISQUE-2
```

Verifier:

```bash
zpool status
zpool list
zfs list
```

## Datasets recommandes

Structure simple:

```text
tank/
├── vmdata
├── backups
├── iso
└── archives
```

Tu peux aussi garder des datasets plus metier si tu stockes directement certains fichiers cote Proxmox:

```text
tank/media
tank/documents
tank/photos
```

Mais pour un premier setup, le plus propre est de laisser les donnees applicatives dans le disque virtuel de la VM Ubuntu, puis de sauvegarder ce disque avec Proxmox.

## Scrub et snapshots

Scrub mensuel:

```bash
zpool scrub tank
zpool status tank
```

Snapshot manuel:

```bash
zfs snapshot tank/vmdata@avant-gros-changement
```

Un snapshot n'est pas un backup. Il aide contre les erreurs rapides, mais il reste sur le meme serveur.

## VM Ubuntu principale

Recommandation:

| Parametre | Valeur |
|---|---|
| Nom | `ubuntu-docker` |
| OS | Ubuntu Server 24.04 LTS |
| vCPU | 4 |
| RAM | 8 a 16 Go |
| Disque systeme | 80 a 128 Go sur SSD/NVMe |
| Disque donnees | 500 Go ou plus sur stockage ZFS |
| Controleur | VirtIO SCSI |
| Reseau | VirtIO sur `vmbr0` |
| Agent | QEMU Guest Agent |

Ubuntu Server 24.04 LTS reste un choix conservateur et stable. Ubuntu 26.04 LTS peut etre choisi pour une nouvelle installation si tu acceptes une periode de validation plus recente.

## VM Home Assistant

Pour Home Assistant, prefere une VM separee plutot qu'un conteneur Docker dans la VM principale.

Pourquoi:

- Home Assistant OS gere mieux les add-ons et certaines integrations.
- Les dongles USB Zigbee, Z-Wave ou Matter sont plus simples a isoler.
- Les backups et restaurations Home Assistant restent independants du reste.
- Une panne applicative Home Assistant ne touche pas la VM Docker principale.

Allocation de depart:

| Parametre | Valeur |
|---|---|
| vCPU | 2 |
| RAM | 2 a 4 Go |
| Disque | 32 a 64 Go |
| Reseau | VirtIO sur `vmbr0` |

## Monter le disque de donnees dans Ubuntu

Dans la VM:

```bash
lsblk
```

Supposons que le disque additionnel soit `/dev/sdb`.

Partitionner:

```bash
sudo parted -s /dev/sdb mklabel gpt
sudo parted -s /dev/sdb mkpart primary ext4 0% 100%
```

Formater:

```bash
sudo mkfs.ext4 /dev/sdb1
```

Creer le point de montage:

```bash
sudo mkdir -p /srv/data
sudo blkid /dev/sdb1
```

Ajouter dans `/etc/fstab`:

```fstab
UUID=REMPLACER_PAR_UUID /srv/data ext4 defaults 0 2
```

Monter et verifier:

```bash
sudo mount -a
df -h
```

Creer les dossiers:

```bash
sudo mkdir -p /srv/data/{media,documents,photos,backups}
sudo chown -R $USER:$USER /srv/data
```

## QEMU Guest Agent

Dans Ubuntu:

```bash
sudo apt update
sudo apt install -y qemu-guest-agent
sudo systemctl enable --now qemu-guest-agent
```

Dans Proxmox:

- Ouvre la VM.
- Va dans `Options`.
- Active `QEMU Guest Agent`.
- Redemarre la VM si necessaire.

## Regles pratiques

- Sauvegarde la VM avant les gros changements.
- Ne garde pas de services critiques directement sur Proxmox.
- Ne donne pas a la VM un acces direct aux disques physiques au debut.
- Documente les VM, IP, disques, volumes et procedures de restauration.
