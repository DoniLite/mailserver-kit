# Coolify et Dokploy

Les deux panels exécutent Traefik sur 80/443 et gèrent les certificats Let's Encrypt. Les ports mail (25, 465, 587, 993) sont publiés directement par le conteneur.

Principe : le webmail est servi sur **le même nom que `MAIL_HOSTNAME`**. Traefik émet donc le certificat de `mail.client.com`, que docker-mailserver relit dans `acme.json` (`SSL_DOMAIN`) et recharge à chaque renouvellement.

## Étapes

1. Créer une application **Docker Compose** depuis ce dépôt Git, fichier `deploy/panel.compose.yaml`.
2. Variables d'environnement dans l'UI :
   ```
   MAIL_HOSTNAME=mail.client.com
   POSTMASTER_ADDRESS=postmaster@client.com
   TZ=Europe/Paris
   ACME_JSON=/data/coolify/proxy/acme.json        # Dokploy : /etc/dokploy/traefik/dynamic/acme.json
   ```
   Optionnel : `ENABLE_CLAMAV`, `POSTFIX_MESSAGE_SIZE_LIMIT`, `DMS_IMAGE_TAG`.
3. Assigner le domaine `mail.client.com` au service **webmail** (port 80) dans l'UI, avec HTTPS activé.
4. Créer le `A` DNS vers le VPS et déployer. Attendre que Traefik ait émis le certificat (vérifier dans le navigateur) ; redémarrer le conteneur `mailserver` s'il a démarré avant.
5. Administration, depuis le shell du VPS (le conteneur est nommé `mailserver`) :
   ```bash
   docker exec -it mailserver setup email add admin@client.com
   docker exec -it mailserver setup config dkim domain client.com
   ```
   Le script `mailctl` fonctionne aussi pour `add`, `dkim`, `dns`, `list`, `alias` : le cloner sur le VPS et renseigner `MAIL_DOMAIN`, `MAIL_HOSTNAME`, `POSTMASTER_ADDRESS` dans un `.env` (inutile pour le reste).

## Points à vérifier sur place

- Le chemin réel de `acme.json` varie selon la version du panel : `docker inspect` du conteneur Traefik (montages) donne la réponse.
- Si le panel renomme le conteneur, définir `DMS_CONTAINER` dans `.env`.
- Les sauvegardes passent par les volumes nommés (`mail-data`, `mail-state`, `config`) ; `mailctl backup` ne couvre que le mode Docker nu.
