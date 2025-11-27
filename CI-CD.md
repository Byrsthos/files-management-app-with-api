# 🚀 CI/CD Configuration

Guide pour configurer le déploiement automatique avec GitHub Actions et Traefik.

## 📋 Configuration des Secrets GitHub

Dans votre repository GitHub, allez dans **Settings > Secrets and variables > Actions** et ajoutez ces secrets :

### Secrets de connexion serveur
```
HOST=your-server-ip-or-domain
USERNAME=your-ssh-username
SSH_PRIVATE_KEY=your-ssh-private-key
PORT=22
```

### Secrets d'environnement
```
JWT_SECRET=your-super-secure-256-bit-secret-key
ADMIN_PASSWORD=YourSecurePassword123!
DOMAIN=your-domain.com
BACK_PORT=3001
FRONT_PORT=3000
PROTOCOL=https
```

## 🔧 Configuration Traefik

Le docker-compose.yml inclut déjà les labels Traefik :

### Frontend
- **URL principale** : `https://your-domain.com`
- **Port interne** : 3000
- **SSL** : Certificat automatique Let's Encrypt

### Backend  
- **URL API** : `https://api.your-domain.com` ou `https://your-domain.com/api`
- **URL Publique** : `https://your-domain.com/public`
- **Port interne** : 3001
- **SSL** : Certificat automatique Let's Encrypt

## 🏗️ Structure du Pipeline CI/CD

Le workflow `.github/workflows/deploy.yml` s'exécute automatiquement sur chaque push vers `main` :

1. **Build** : Installation des dépendances et build des applications
2. **Deploy** : Connexion SSH au serveur et déploiement
3. **Environment** : Création du fichier `.env` à partir des secrets GitHub
4. **Docker** : Reconstruction et redémarrage des conteneurs

## 📁 Structure des fichiers

```
.github/
└── workflows/
    └── deploy.yml          # Pipeline CI/CD

docker-compose.yml          # Configuration Docker avec labels Traefik
.gitignore                 # Exclut package-lock.json
```

## ⚙️ Configuration serveur

### 1. Préparer le serveur

```bash
# Installer Docker (inclut Docker Compose v2)
curl -fsSL https://get.docker.com | sh
sudo usermod -aG docker $USER

# Cloner le repository
git clone https://github.com/your-username/your-repo.git
cd your-repo
```

### 2. Configuration Traefik

Assurez-vous que Traefik est configuré avec :
- Point d'entrée `websecure` sur le port 443
- Résolveur de certificats `letsencrypt`
- Network Docker partagé

### 3. Première exécution

Le pipeline créera automatiquement le fichier `.env` et déploiera l'application.

## 🔄 Déploiement automatique

- **Trigger** : Push sur la branche `main`
- **Durée** : ~3-5 minutes
- **Actions** : Build → Test → Deploy → Restart

## ✅ Vérification

Après déploiement, vérifiez :

```bash
# Status des conteneurs
docker compose ps

# Logs des services
docker compose logs -f

# Test des endpoints
curl https://your-domain.com
curl https://your-domain.com/api/health
curl https://your-domain.com/public/{file-id}
```

## 🚨 Troubleshooting

### Échec de connexion SSH
- Vérifiez les secrets `HOST`, `USERNAME`, `SSH_PRIVATE_KEY`
- Assurez-vous que la clé SSH est au format PEM

### Erreurs de build
- Vérifiez les logs dans l'onglet Actions de GitHub
- Les dépendances sont installées avec `npm install` (pas `npm ci`)

### Problèmes Traefik
- Vérifiez que les labels correspondent à votre configuration Traefik
- Assurez-vous que `DOMAIN` est correctement défini