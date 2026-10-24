# Déploiement

Ce dépôt n'a plus de déploiement automatique : il n'y a plus de workflow GitHub Actions. Le déploiement se fait à la main
sur le serveur, depuis le dossier où le dépôt est cloné (`~/files-management-app-with-api` sur le serveur Byrsthos).

## Déployer une nouvelle version

```bash
ssh <utilisateur>@<serveur>
cd ~/files-management-app-with-api
git fetch origin
git status                      # aucune modification locale attendue
git pull --ff-only origin main
docker compose up -d --build    # reconstruit et redémarre backend et frontend
docker compose ps               # les deux conteneurs sont "running"
docker compose logs --tail=50 backend
```

Le fichier `.env` reste sur le serveur (jamais versionné) : à créer une seule fois à partir de `.env.example`
(`JWT_SECRET`, `ADMIN_PASSWORD`, `DOMAIN`, `BACK_PORT`, `FRONT_PORT`, `PROTOCOL`).

Les fichiers et la base (`uploads/` et `data/` à la racine du dépôt, montés dans le backend) sont sur le serveur et ne sont pas touchés par un
déploiement. Les sauvegardes sont faites par les scripts `scripts/backup/` du dépôt byrsthos-athletics.

## Vérifier

```bash
curl -I https://<domaine>/public/<id-d-un-fichier-public>
# X-Content-Type-Options: nosniff sur toutes les réponses de fichiers ;
# pour un SVG : Content-Security-Policy "default-src 'none'; style-src 'unsafe-inline'; sandbox"
# et Content-Disposition: attachment (les <img src> des sites continuent d'afficher les drapeaux et logos)
```

## Revenir en arrière

```bash
cd ~/files-management-app-with-api
git log --oneline -5
git checkout <commit-précédent>
docker compose up -d --build
```

## Traefik

Le `docker-compose.yml` contient les labels Traefik (point d'entrée `websecure`, résolveur `letsencrypt`, réseau
Docker partagé avec Traefik) :

- frontend : `https://<domaine>` (port interne 3000) ;
- backend : `https://api.<domaine>` et `https://<domaine>/public` (port interne 3001).
