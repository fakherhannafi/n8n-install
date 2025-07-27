
# Documentation Technique : Installation de n8n avec Docker Compose

## Prérequis

- **Système** : Un serveur Linux (ex : Ubuntu) avec Docker et Docker Compose installés.
- **CPU** : 2 vCPU pour un usage standard de n8n.
- **Mémoire** : Minimum 8 Go de RAM (prévoir plus pour des workflows complexes).
- **Stockage** : 512 Mo à 4 Go de SSD pour la base de données et les fichiers. Par défaut, n8n utilise SQLite (fichier `.sqlite` stocké dans `/home/node/.n8n`).
- **Persistance** : Un volume Docker doit être monté sur `/home/node/.n8n` pour conserver la base de données et la clé d’encryption.

## Fichier de configuration Docker Compose
[docker-compose.yml](docker-compose.yml).

## Remarques
- La ligne ports: - "127.0.0.1:5678:5678" lie n8n à l’interface locale. Ainsi, n8n n’est pas directement exposé au public ; seul Traefik (via le domaine configuré) est accessible depuis l’extérieur[5].
- Remplacez ${SUBDOMAIN}.${DOMAIN_NAME} par votre sous-domaine (par ex. n8n.mondomaine.com). N8N_HOST, WEBHOOK_URL, etc. doivent correspondre à cette URL publique.
- La variable N8N_PORT reste 5678 (port interne de n8n), tandis que Clever Cloud exposera ce port via le port 8080 (voir ci-dessous).
- Volumes Docker : Déclarez deux volumes pour persister les données :
- n8n_data → monté sur /home/node/.n8n : stocke la base de données SQLite et la clé d’encryption[3].
- traefik_data → monté sur /letsencrypt : stocke les certificats SSL générés par Traefik[3].
- (Optionnel) ./local-files → monté sur /files pour partager des fichiers avec n8n.

## Configuration spécifique Clever Cloud
Clever Cloud impose quelques réglages :
- Port exposé : Par défaut, l’application doit écouter sur le port 8080. Pour que le trafic HTTP/HTTPS aille à n8n (port 5678 interne), ajoutez l’environnement Clever Cloud suivant : CC_DOCKER_EXPOSED_HTTP_PORT=5678. Ainsi, les requêtes reçues seront redirigées vers le port 5678 de votre conteneur.

- Variables d’environnement : Dans l’interface Clever Cloud (ou via clever env set), définissez les variables N8N_HOST, WEBHOOK_URL, N8N_PROTOCOL, etc. selon votre domaine. Par exemple :
- N8N_HOST = n8n.mondomaine.com
- N8N_PROTOCOL = https
- WEBHOOK_URL = https://n8n.mondomaine.com/

. Ces valeurs correspondent à l’exemple du docker-compose.yml ci-dessus
- Fichier .env : Il n’est pas nécessaire de créer un fichier .env puisque vous pouvez déclarer les variables directement dans Clever Cloud.

## Lancement de n8n
Dans le répertoire contenant votre docker-compose.yml :

Cette commande télécharge les images et lance les conteneurs en arrière-plan. Pour arrêter le service :

```bash
sudo docker compose up -d
```

## Arrêt de n8n
```bash
sudo docker compose stop
```
# Mise à jour n8n
```bash
sudo docker compose pull
sudo docker compose up -d
```
## Accès à n8n
Après le démarrage, n8n sera accessible via l’URL sécurisée https://n8n.mondomaine.com (selon le sous-domaine configuré).

## Importer des workflows (templates) JSON
Une fois n8n en ligne, vous pouvez charger des templates de workflow (fichiers JSON) :
- Dans l’interface n8n, ouvrez un nouveau workflow vierge.
- Cliquez sur l’icône du menu (⋯) dans la barre supérieure et choisissez Import. Vous pouvez importer depuis un fichier local (JSON) ou de copier coller le JSON dans l’éditeur ouvert.
- Par exemple, téléchargez votre template .json, puis sélectionnez Import from File et chargez le fichier. Le contenu du workflow sera alors créé dans l’éditeur.

## Configurer les credentials
Chaque workflow peut nécessiter des credentials (identifiants) pour accéder à des services externes. Pour les préparer :
1. Dans n8n, cliquez sur le bouton Create (icône « + ») en haut à gauche, puis choisissez Credential.
2. Sélectionnez le type de service (par ex. Google Drive, Slack, HTTP Request, etc.) correspondant à votre besoin.
3. Renseignez les informations d’authentification requises (clé API, nom d’utilisateur/mot de passe, etc.) dans la fenêtre qui s’ouvre.
4. Enregistrez la credential. n8n la testera automatiquement pour vérifier son bon fonctionnement.
Répétez ces étapes pour chaque credential dont vos workflows ont besoin. Assurez-vous que les noms des credentials correspondent à ceux référencés dans vos templates.

Sources : La documentation officielle de n8n détaille la configuration Docker Compose, les prérequis matériels, l’import de workflows et la gestion des credentials[1][5][3][9][11][12][10]. Les conseils sur les ressources (CPU/RAM) proviennent également de retours d’expérience utilisateurs[2].

## References
- [1] [prerequisites](https://docs.n8n.io/embed/prerequisites/)
- [2] [hardware-for-self-hosting](https://community.n8n.io/t/hardware-for-self-hosting/30647)
- [3] [docker-compose](https://docs.n8n.io/hosting/installation/server-setups/docker-compose/)
- [4] [docker-compose](https://docs.n8n.io/hosting/installation/server-setups/docker-compose/)
- [5] [docker-compose](https://docs.n8n.io/hosting/installation/server-setups/docker-compose/)
- [6] [docker](https://www.clever-cloud.com/developers/doc/applications/docker/)
- [7] [docker-compose](https://docs.n8n.io/hosting/installation/server-setups/docker-compose/)
- [8] [docker-compose](https://docs.n8n.io/hosting/installation/server-setups/docker-compose/)
- [9] [export-import](https://docs.n8n.io/workflows/export-import/)
- [10] [add-edit-credentials](https://docs.n8n.io/credentials/add-edit-credentials/)
- [11] [add-edit-credentials](https://docs.n8n.io/credentials/add-edit-credentials/)
- [12] [add-edit-credentials](https://docs.n8n.io/credentials/add-edit-credentials/)