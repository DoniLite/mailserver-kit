# mailserver-kit

Serveur mail complet prêt à déployer chez un client, basé sur [docker-mailserver](https://docker-mailserver.github.io/docker-mailserver/latest/) (Postfix, Dovecot, Rspamd, Fail2ban) avec webmail Roundcube optionnel.

Une fiche client (`.env`) + un script (`mailctl`) : même procédure sur n'importe quel VPS.

| Environnement du client | Fichier compose | Certificat TLS |
|---|---|---|
| Docker seul (aucun panel) | `compose.yaml` | `mailctl cert http` ou `cloudflare` |
| Coolify ou Dokploy | `deploy/panel.compose.yaml` | repris de Traefik, voir [docs/PANELS.md](docs/PANELS.md) |

## Prérequis chez le client

- VPS Linux, **2 Go de RAM minimum** (4 Go avec ClamAV), Docker + Compose v2.
- **Port 25 sortant ouvert** chez l'hébergeur (certains, comme AWS ou GCP, le bloquent par défaut, d'autres sur demande).
- Possibilité de définir le **reverse DNS (PTR)** de l'IP vers `mail.<domaine>`. Sans lui, Gmail et Outlook rejettent ou classent en spam.
- Accès à la zone DNS du domaine.
- Ports ouverts : 25, 465, 587, 993 (+ 80 pour `cert http`). Docker contourne `ufw` pour les ports publiés ; filtrer côté pare-feu du fournisseur.
- Sur Coolify/Dokploy, ni 25, 465, 587 ni 993 ne sont pris par le panel (seuls 80/443 le sont).

## Déploiement (Docker nu)

```bash
git clone <ce-repo> /opt/mail && cd /opt/mail
./mailctl init client.com            # écrit .env (hostname par défaut : mail.client.com)
$EDITOR .env                         # ajuster : TZ, ClamAV, taille max, profil webmail...
```

1. **DNS** : créer l'enregistrement `A` `mail.client.com` vers l'IP du VPS, et le PTR chez l'hébergeur.
2. **Certificat** : `./mailctl cert http` (port 80 libre) ou `./mailctl cert cloudflare` (renseigner `CF_API_TOKEN`). Puis `sudo ./mailctl cert autorenew`. Pour un test : `./mailctl cert selfsigned`. Un certificat existant se dépose dans `data/certbot/live/<hostname>/{fullchain,privkey}.pem`.
3. **Démarrage** : `./mailctl up`.
4. **Premier compte et DKIM** :
   ```bash
   ./mailctl add admin@client.com
   ./mailctl dkim client.com
   ```
5. **Records DNS** : `./mailctl dns` affiche MX, SPF, DKIM, DMARC prêts à copier ; `./mailctl check` vérifie le tout (SPF/DKIM/DMARC/PTR et ports).

Plusieurs domaines sur un même serveur : `./mailctl dkim autre.com`, `./mailctl dns autre.com`, puis `./mailctl add user@autre.com`.

## Webmail (Roundcube)

Mettre `COMPOSE_PROFILES=webmail` dans `.env` puis `./mailctl up`. Roundcube écoute sur `127.0.0.1:8080` et se publie via un reverse proxy, par exemple Caddy :

```
webmail.client.com {
    reverse_proxy 127.0.0.1:8080
}
```

## Commandes courantes

```
./mailctl add|del|passwd <email>    ./mailctl alias add contact@client.com admin@client.com
./mailctl quota <email> 5G          ./mailctl list
./mailctl logs | status | update    ./mailctl backup
```

`update` tire la nouvelle image : pour une offre vendue, fixer `DMS_IMAGE_TAG` sur une version testée et ne la monter qu'après vérification.

## Ce qui est configuré par défaut

- Rspamd (anti-spam, DKIM, SPF, DMARC) ; OpenDKIM, OpenDMARC, policyd-spf, Amavis, SpamAssassin et Postgrey désactivés.
- `SPOOF_PROTECTION=1`, Fail2ban, TLS moderne, pas de relais ouvert (`PERMIT_DOCKER=none`).
- Quotas Dovecot, spam rangé en Junk, apprentissage bayésien via le dossier Junk.
- IMAP 993 (TLS implicite), soumission 465/587. L'IMAP clair (143) et le POP3 ne sont pas publiés.

Réglages avancés : [docs/CUSTOMIZE.md](docs/CUSTOMIZE.md). Livraison au client : [docs/HANDOVER.md](docs/HANDOVER.md).

## Structure

```
compose.yaml  deploy/panel.compose.yaml  mailserver.env  .env.example  mailctl
templates/rspamd/override.d/   config Rspamd montée dans DMS (signature DKIM multi-domaines)
docs/                          PANELS, CUSTOMIZE, HANDOVER
data/                          (non versionné) mails, comptes, clés DKIM, certificats
```

## Licence

MIT. docker-mailserver est sous licence MIT également.
