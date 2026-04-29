# 07 - Maintenance et depannage

## Routine mensuelle

- Verifier `zpool status`.
- Lancer ou verifier le scrub ZFS.
- Verifier SMART des disques.
- Appliquer les mises a jour Proxmox.
- Appliquer les mises a jour Ubuntu.
- Mettre a jour quelques stacks Docker, pas tout aveuglement.
- Verifier Uptime Kuma.
- Tester une restauration.
- Verifier l'espace disque.
- Documenter les changements.

## Verification ZFS

Etat du pool:

```bash
zpool status
zpool list
zfs list
```

Scrub:

```bash
zpool scrub tank
zpool status tank
```

Garde environ 20 pourcent d'espace libre sur un pool ZFS. Un pool trop plein devient moins performant et plus penible a administrer.

## SMART

Installer:

```bash
apt install -y smartmontools
```

Verifier un disque:

```bash
smartctl -a /dev/sda
```

Adapte `/dev/sda` au bon disque. Utilise `lsblk` et les informations de modele/serie pour eviter les erreurs.

## Espace disque

Commandes utiles:

```bash
df -h
zfs list
docker system df
du -sh /opt/docker/* 2>/dev/null
du -sh /srv/data/* 2>/dev/null
```

Ca aide a separer:

- Systeme Ubuntu plein.
- Pool ZFS plein.
- Images Docker inutilisees.
- Donnees applicatives trop grosses.
- Logs ou caches anormaux.

## Mise a jour d'une stack Docker

Procedure prudente:

```bash
cd /opt/docker/nom-du-service
docker compose pull
docker compose up -d
docker compose logs -f
```

Si le service est critique:

- Fais un backup avant.
- Lis les notes de version.
- Mets a jour un service a la fois.
- Valide le fonctionnement apres redemarrage.

## Conteneur qui ne demarre pas

Verifier:

```bash
docker compose ps
docker compose logs -f
```

Causes frequentes:

- Port deja utilise.
- Permission incorrecte sur un volume.
- Variable d'environnement manquante.
- Image mise a jour avec changement incompatible.
- Chemin de volume inexistant.

## Service inaccessible

Verifier:

```bash
ss -tulpn
docker ps
docker compose logs -f
```

Points a controler:

- Le conteneur tourne.
- Le port est publie.
- Le service ecoute sur le bon port.
- L'IP de la VM est correcte.
- Le DNS local pointe au bon endroit.
- Le reverse proxy cible le bon port.
- Le firewall ne bloque pas.

## DNS casse

Si AdGuard ou Pi-hole casse la resolution:

- Reviens temporairement au DNS du routeur.
- Teste l'acces direct par IP.
- Verifie que le port `53/tcp` et `53/udp` sont actifs.
- Verifie que le routeur distribue la bonne IP DNS.
- Ne modifie pas tout le LAN avant d'avoir valide sur un seul client.

## Pool ZFS degrade

Premiere commande:

```bash
zpool status
```

Procedure generale:

- Ne redemarre pas en boucle.
- Identifie le disque defectueux avec modele et numero de serie.
- Verifie les cables et l'alimentation.
- Remplace le disque si necessaire.
- Lance le resilver selon la procedure ZFS.
- Surveille jusqu'a retour a l'etat `ONLINE`.

## Documentation a maintenir

Garde un fichier ou carnet avec:

- IP des machines.
- Noms DNS.
- Ports importants.
- Emplacement des volumes Docker.
- Plan de backup.
- Procedure de restauration.
- Date des mises a jour majeures.
- Changements materiels.

## Erreurs classiques

- Confondre miroir ZFS et backup.
- Tout installer en une journee sans tester.
- Exposer Proxmox sur Internet.
- Exposer Vaultwarden sans backup teste.
- Acheter des disques SMR pour ZFS.
- Oublier l'UPS.
- Ne pas documenter les changements.
- Mettre a jour tous les conteneurs sans backup.
- Utiliser des permissions trop larges pour corriger vite.
- Ne jamais tester la restauration.
