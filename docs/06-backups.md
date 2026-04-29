# 06 - Backups et restauration

## Principe

RAID, ZFS mirror et snapshots ne sont pas des backups.

Ils ne protegent pas suffisamment contre:

- Suppression accidentelle.
- Mauvaise commande.
- Ransomware.
- Corruption logique.
- Vol.
- Incendie.
- Panne complete du serveur.

## Regle 3-2-1

Objectif:

- 3 copies des donnees importantes.
- 2 types de supports differents.
- 1 copie hors machine, idealement hors site.

Version homelab simple:

- Original sur le serveur.
- Backup local sur disque USB.
- Copie cloud ou disque stocke ailleurs pour les documents et photos critiques.

## Ce qu'il faut sauvegarder

Priorite haute:

- Documents personnels.
- Photos.
- Donnees Vaultwarden.
- Configurations Docker.
- Bases de donnees applicatives.
- Notes d'administration.

Priorite moyenne:

- Config Jellyfin.
- Config Gitea.
- Config Syncthing.
- Exports de Home Assistant.

Priorite basse:

- Medias recuperables.
- Caches.
- Images Docker.
- Fichiers temporaires.

## Backups Proxmox

Proxmox peut sauvegarder les VM.

Recommandation:

- Backup hebdomadaire de la VM Ubuntu Docker.
- Backup avant gros changement.
- Retention limitee pour eviter de remplir le stockage.
- Copie reguliere vers un disque externe.

Points a verifier:

- Le job de backup se termine sans erreur.
- La taille du backup est coherente.
- Tu connais la procedure de restauration.

## Backups Docker

Sauvegarde:

- `/opt/docker`
- `/srv/data`
- Les dumps de bases de donnees si un service utilise PostgreSQL, MariaDB ou SQLite critique.

Pour les services avec base de donnees, un simple backup de fichiers peut etre insuffisant si la base est active. Preferer un dump applicatif ou arreter temporairement le service pendant le backup si necessaire.

## Snapshots ZFS

Les snapshots sont utiles pour revenir rapidement en arriere.

Exemple:

```bash
zfs snapshot tank/vmdata@avant-update
```

Mais:

- Ils vivent sur le meme pool.
- Ils ne remplacent pas un backup externe.
- Ils peuvent etre perdus si le serveur ou le pool est perdu.

## Disque USB externe

Bon usage:

- Brancher le disque seulement pendant le backup.
- L'ejecter proprement.
- Le stocker separement du serveur.
- Alterner deux disques si possible.

Pourquoi:

- Un disque branche en permanence est plus expose aux erreurs, surtensions et ransomwares.

## Tests de restauration

Un backup non teste est une hypothese.

Test mensuel minimal:

- Restaurer un fichier.
- Restaurer un dossier de config Docker dans un emplacement temporaire.
- Restaurer une VM de test ou verifier qu'un backup Proxmox est exploitable.
- Noter la date du test.

## Strategie simple recommandee

- Snapshots locaux avant changements importants.
- Backups Proxmox hebdomadaires.
- Backup de `/opt/docker` et `/srv/data`.
- Disque USB externe au moins mensuel.
- Copie cloud ou hors site pour documents et photos critiques.
- Test de restauration mensuel.

## Checklist backup

- Les donnees critiques sont identifiees.
- Les services critiques sont arretes ou dumpes correctement.
- Les backups sont stockes hors du serveur.
- La retention est configuree.
- La restauration a ete testee.
- Les secrets necessaires a la restauration sont accessibles.
