# 05 - Reseau, DNS et acces distant

## Plan d'adressage simple

Exemple:

| Equipement | IP |
|---|---|
| Routeur | `192.168.1.1` |
| Proxmox | `192.168.1.10` |
| VM Ubuntu Docker | `192.168.1.20` |
| VM Home Assistant | `192.168.1.30` |
| VM Linux test | `192.168.1.40` |

Utilise des reservations DHCP sur le routeur quand c'est possible. C'est souvent plus simple qu'une configuration statique manuelle sur chaque machine.

## Domaine local

Evite `.local`, car il est utilise par mDNS et peut creer des conflits.

Prefere:

```text
home.arpa
```

Exemples:

- `pve1.home.arpa`
- `portainer.home.arpa`
- `uptime.home.arpa`
- `vaultwarden.home.arpa`
- `jellyfin.home.arpa`

## DNS local

AdGuard Home ou Pi-hole peuvent servir a:

- Bloquer certaines pubs et trackers.
- Resoudre des noms internes.
- Centraliser le DNS maison.

Deploiement prudent:

- Installer le service.
- Tester depuis un seul ordinateur.
- Ajouter quelques entrees locales.
- Valider que la resolution Internet fonctionne.
- Seulement ensuite, configurer le routeur pour distribuer ce DNS.

Garde toujours un plan B:

- DNS du routeur.
- `1.1.1.1`.
- `8.8.8.8`.
- Acces direct par IP aux interfaces critiques.

## Reverse proxy

Nginx Proxy Manager permet de remplacer:

```text
http://192.168.1.20:3001
```

par:

```text
https://uptime.home.arpa
```

Pour un usage interne:

- Cree une entree DNS locale vers l'IP de la VM Docker.
- Cree un proxy host dans Nginx Proxy Manager.
- Garde les interfaces admin accessibles seulement depuis LAN ou Tailscale.

## Tailscale

Tailscale est recommande pour l'acces distant initial.

Installation dans Ubuntu:

```bash
curl -fsSL https://tailscale.com/install.sh | sh
sudo tailscale up
```

Verifier:

```bash
tailscale status
tailscale ip
```

Bonnes pratiques:

- Installe Tailscale sur la VM Ubuntu Docker au debut.
- Ajoute-le aussi sur ton ordinateur et ton telephone.
- Desactive l'expiration de cle pour les serveurs stables si necessaire depuis l'admin console Tailscale.
- N'ouvre pas Proxmox sur Internet.

## Port forwarding

Evite le port forwarding au debut.

Avant d'en ouvrir un:

- Confirme que le service est a jour.
- Confirme que l'authentification est forte.
- Confirme que les backups sont testes.
- Confirme que tu sais fermer le port rapidement.
- Documente pourquoi ce port existe.

## Segmentation reseau

Les VLANs ne sont pas necessaires au premier jour.

Tu peux y venir plus tard pour separer:

- Serveurs.
- IoT.
- Invites.
- Clients personnels.
- Services exposes.

Ne commence pas par les VLANs si tu n'as pas encore de backups, monitoring et restauration.
