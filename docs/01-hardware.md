# 01 - Materiel recommande

## Objectif

Le materiel doit etre assez puissant pour apprendre Proxmox, Docker, ZFS et quelques services maison, sans devenir bruyant, cher ou complexe. Pour un premier homelab, la fiabilite et la recuperation sont plus importantes que la performance maximale.

## Configuration cible

| Element | Recommandation | Raison |
|---|---|---|
| Format | Petite tour ou desktop business evolutif | Meilleur compromis silence, prix, disques internes et maintenance |
| CPU | Intel i5/i7 8e gen ou plus recent, ou AMD Ryzen 5/7 | Suffisant pour plusieurs VM et services Docker |
| RAM | 32 Go minimum, 64 Go ideal | Proxmox, VM Ubuntu, cache ZFS, Home Assistant et VM de test |
| Disque systeme | SSD/NVMe 500 Go a 1 To | VM reactives, mises a jour rapides, logs et caches plus fluides |
| Donnees | 2 HDD NAS CMR en miroir ZFS | Redondance simple et cout raisonnable par To |
| Backup | Disque USB externe au moins egal aux donnees critiques | Copie separee du stockage principal |
| UPS | 700 a 1000 VA line-interactive | Arret propre et protection contre les coupures |
| Reseau | Ethernet 1 Gbit/s | Simple, stable et suffisant pour debuter |

## Formats de machine

### Mini-PC

Avantages:

- Tres silencieux.
- Faible consommation electrique.
- Bon choix pour Proxmox, Docker et quelques VM legeres.

Limites:

- Peu ou pas d'emplacements pour des HDD 3.5 pouces.
- Refroidissement moins tolerant sous charge continue.
- Extension plus limitee.

### Desktop business usage

Exemples: Dell OptiPlex, HP EliteDesk, Lenovo ThinkCentre.

Avantages:

- Excellent rapport prix/fiabilite en occasion.
- Pieces faciles a trouver.
- RAM et SSD souvent faciles a remplacer.
- Bruit et consommation raisonnables.

Limites:

- Certains formats SFF ont peu d'espace pour des disques 3.5 pouces.
- Alimentation parfois proprietaire.

### Petite tour

Avantages:

- Meilleur choix si tu veux 2 HDD internes.
- Plus simple a refroidir.
- Plus evolutive.
- Meilleure option pour un vrai miroir ZFS interne.

Limites:

- Plus volumineuse.
- Peut consommer un peu plus selon les composants.

## CPU

Bon point de depart:

- Intel i5-8500, i5-9500, i5-10500, i5-11500 ou plus recent.
- Intel i7 equivalent si bonne occasion.
- AMD Ryzen 5 5600G, Ryzen 7 5700G ou equivalents.

Notes:

- Intel Quick Sync peut aider Jellyfin pour le transcodage video.
- Le transcodage n'est pas necessaire si tes clients lisent les fichiers en direct.
- Pour apprendre, 4 a 6 coeurs modernes suffisent largement.

## RAM

Allocation typique avec 32 Go:

- Proxmox: 4 a 8 Go disponibles pour l'hote.
- VM Ubuntu Docker: 8 a 16 Go.
- VM Home Assistant: 2 a 4 Go.
- VM Linux de test: 2 a 8 Go.
- Reste: cache ZFS et marge.

64 Go est plus confortable si tu veux ajouter Immich, Paperless-ngx, plusieurs bases de donnees ou plusieurs VM de test.

## Stockage

### Disque systeme

Utilise un SSD ou NVMe pour:

- Proxmox.
- Les ISO.
- Les disques systeme des VM.
- Les caches et mises a jour.

### Disques de donnees

Recommandation simple:

- 2 disques HDD NAS CMR identiques.
- Miroir ZFS sur l'hote Proxmox.
- Donnees exposees a Ubuntu via un disque virtuel Proxmox, pas via un montage direct de `/tank`.

Exemples de familles adaptees:

- WD Red Plus.
- Seagate IronWolf.
- Toshiba N300.

## CMR vs SMR

Prends des disques CMR pour ZFS.

Pourquoi:

- Ecritures plus previsibles.
- Meilleur comportement lors des resilver/reconstructions.
- Moins de variations de performance sous charge longue.

Evite les disques SMR pour un miroir ZFS qui tourne en continu.

## UPS

Un UPS est fortement recommande.

Il protege contre:

- Coupures courtes.
- Micro-coupures.
- Arrets brutaux pendant des ecritures disque.
- Risques de corruption apres panne electrique.

Choix pratique:

- APC ou Eaton line-interactive.
- 700 a 1000 VA pour une petite machine maison.
- Si possible, modele compatible USB pour declencher un arret propre.

## Regle d'achat simple

Si tu veux un setup robuste sans trop reflechir:

- Petite tour business ou workstation compacte.
- 32 ou 64 Go RAM.
- 1 NVMe 1 To.
- 2 HDD NAS CMR de meme capacite.
- 1 disque USB externe.
- 1 UPS.
