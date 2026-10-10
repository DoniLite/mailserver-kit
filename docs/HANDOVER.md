# Livraison au client

À remplir et remettre une fois le déploiement validé (`./mailctl check` sans [FAIL], mail-tester ≥ 9/10).

## Paramètres de configuration des clients mail

| | |
|---|---|
| Serveur IMAP (réception) | `mail.client.com`, port **993**, SSL/TLS |
| Serveur SMTP (envoi) | `mail.client.com`, port **465** SSL/TLS (ou 587 STARTTLS) |
| Identifiant | l'adresse complète, `user@client.com` |
| Mot de passe | celui communiqué à la création du compte |
| Webmail | `https://mail.client.com` ou `https://webmail.client.com` |

## À conserver côté prestataire

- Accès SSH au VPS, répertoire du kit, `.env`.
- `./mailctl backup` planifié (ex. cron quotidien) et copie hors serveur du dossier `backups/`.
- Renouvellement du certificat : `sudo ./mailctl cert autorenew` (mode Docker nu).
- Mises à jour : `DMS_IMAGE_TAG` figé, montée de version testée avant chaque passage.

## Support de premier niveau

| Symptôme | Vérification |
|---|---|
| Mails en spam chez Gmail/Outlook | `./mailctl check` : PTR, SPF, DKIM, DMARC ; IP présente sur une blacklist ? |
| Impossible de se connecter | certificat expiré (`./mailctl cert renew`), IP bannie par Fail2ban : `docker exec mailserver setup fail2ban` |
| Mails sortants refusés | port 25 sortant bloqué par l'hébergeur |
