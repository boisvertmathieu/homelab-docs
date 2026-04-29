# 04 - Securite minimale

## Principe

Le but n'est pas de rendre un homelab parfait. Le but est d'eviter les erreurs qui transforment une machine d'apprentissage en risque pour ton reseau, tes mots de passe ou tes donnees.

## Regles prioritaires

- N'expose pas Proxmox sur Internet.
- N'expose pas Vaultwarden directement sur Internet au debut.
- Utilise Tailscale ou un VPN pour l'acces distant.
- Active 2FA sur les interfaces importantes.
- Utilise des mots de passe uniques et longs.
- Sauvegarde avant de mettre un service critique en production.
- Teste les restaurations.
- Mets a jour regulierement Proxmox, Ubuntu et les images Docker.

## Comptes et acces

### Proxmox

- Garde `root@pam` pour l'administration initiale.
- Ajoute 2FA.
- Cree un utilisateur dedie si plusieurs personnes administrent.
- Ne publie jamais `:8006` sur Internet.

### SSH

Sur Ubuntu, evite la connexion root directe.

Dans `/etc/ssh/sshd_config`:

```text
PermitRootLogin no
PasswordAuthentication no
```

Avant de desactiver l'authentification par mot de passe, verifie que ta cle SSH fonctionne.

Redemarrer SSH:

```bash
sudo systemctl restart ssh
```

## Mots de passe

Utilise un gestionnaire de mots de passe pour:

- Proxmox.
- Ubuntu admin.
- Portainer.
- Nginx Proxy Manager.
- AdGuard ou Pi-hole.
- Vaultwarden.
- Gitea.
- Comptes mail utilises pour les alertes.

Ne reutilise pas les mots de passe entre services.

## Ports

Approche recommandee:

- LAN seulement pour les interfaces admin.
- Tailscale pour acces hors maison.
- Aucun port forwarding au debut.
- Reverse proxy interne pour les noms propres.

Si un port est expose, documente:

- Le port.
- Le service.
- La raison.
- Le mode d'authentification.
- Le plan de rollback.

## Vaultwarden

Vaultwarden merite une prudence particuliere.

Avant usage reel:

- Confirme que les backups fonctionnent.
- Confirme que la restauration fonctionne.
- Desactive les inscriptions publiques.
- Active 2FA sur les comptes utilisateurs.
- Garde l'acces via LAN ou Tailscale.

Variable importante:

```yaml
environment:
  - SIGNUPS_ALLOWED=false
```

## Docker

Risques principaux:

- Le socket Docker donne beaucoup de privileges.
- Les images `latest` peuvent changer de comportement.
- Les conteneurs exposes ajoutent de la surface d'attaque.
- Les volumes mal montes peuvent exposer trop de fichiers.

Bonnes pratiques:

- Limite les ports publies.
- Prefere les dossiers dedies par service.
- Sauvegarde avant `docker compose pull`.
- Lis les release notes des services critiques.
- Evite `privileged: true` sauf besoin clair et documente.

## Mises a jour

Cycle simple:

- Mises a jour OS une fois par mois.
- Mises a jour Docker service par service.
- Backup avant mise a jour majeure.
- Verification des logs apres mise a jour.

Proxmox:

```bash
apt update
apt full-upgrade
```

Ubuntu:

```bash
sudo apt update
sudo apt upgrade
```

Docker:

```bash
docker compose pull
docker compose up -d
docker compose logs -f
```

## Pare-feu

Pour debuter, la securite vient surtout de l'absence d'exposition Internet. Ensuite, tu peux ajouter:

- Pare-feu Proxmox.
- UFW dans la VM Ubuntu.
- Regles specifiques par service.

Exemple UFW minimal dans Ubuntu:

```bash
sudo ufw allow OpenSSH
sudo ufw allow from 192.168.1.0/24
sudo ufw enable
sudo ufw status verbose
```

Adapte le reseau `192.168.1.0/24` a ton LAN.

## Documentation de securite

Tiens a jour:

- Liste des services exposes.
- URL internes.
- Comptes admin.
- Emplacement des secrets.
- Procedure de restauration.
- Date du dernier test de backup.

Ne stocke pas les mots de passe en clair dans le depot Git.
