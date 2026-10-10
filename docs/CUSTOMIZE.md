# Personnalisation

Référence complète : <https://docker-mailserver.github.io/docker-mailserver/latest/config/environment/>

| Besoin | Où | Réglage |
|---|---|---|
| Hostname, domaine, fuseau | `.env` | `MAIL_HOSTNAME`, `MAIL_DOMAIN`, `TZ` |
| Antivirus | `.env` | `ENABLE_CLAMAV=1` (+1,5 Go RAM) |
| Taille max d'un message | `.env` | `POSTFIX_MESSAGE_SIZE_LIMIT` (octets) |
| Version DMS | `.env` | `DMS_IMAGE_TAG` |
| Quota par boîte | commande | `./mailctl quota user@client.com 5G` |
| Greylisting | `mailserver.env` | `RSPAMD_GREYLISTING=1` |
| Rspamd sur les mails sortants | `mailserver.env` | `RSPAMD_CHECK_AUTHENTICATED=1` |
| Relais SMTP (IP du VPS blacklistée) | `data/config/postfix-relaymap.cf`, `postfix-sasl-password.cf` | voir doc DMS « Relay hosts » |
| POP3 | `mailserver.env` + port 995 dans compose | `ENABLE_POP3=1` |
| Sieve (filtres utilisateur) | `mailserver.env` + port 4190 | `ENABLE_MANAGESIEVE=1` |
| Interface web Rspamd (port 11334) | `data/config/rspamd/custom-commands.conf` | mot de passe : `docker exec -it mailserver rspamadm pw` puis `set-option-for-controller password "<hash>"` |

## Ordre de priorité

`environment:` du compose (alimenté par `.env`) l'emporte sur `mailserver.env`. Mettre les valeurs communes à tous les clients dans `mailserver.env` et les valeurs propres à un client dans `.env`.

## DKIM

Sélecteur `mail`, RSA 2048, un fichier par domaine dans `data/config/rspamd/dkim/`. `templates/rspamd/override.d/dkim_signing.conf` est monté dans le conteneur et signe tous les domaines pour lesquels une clé existe. Pour changer de sélecteur, modifier ce fichier **et** `dkim_p` dans `mailctl`.

## Vérification du serveur

- `./mailctl check` : DNS et ports.
- Envoi test vers <https://www.mail-tester.com> (objectif 9/10 ou plus).
- Journaux : `./mailctl logs`.
