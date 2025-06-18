# Instructions de Déploiement - Plateforme de Simulation de Ferme Virtuelle

Ce document fournit les instructions détaillées pour déployer la plateforme de simulation de ferme virtuelle dans un environnement de production.

## Table des matières

1. [Prérequis](#1-prérequis)
2. [Déploiement du Frontend](#2-déploiement-du-frontend)
3. [Déploiement du Backend](#3-déploiement-du-backend)
4. [Configuration de la Base de Données](#4-configuration-de-la-base-de-données)
5. [Configuration des Services Externes](#5-configuration-des-services-externes)
6. [Sécurité et SSL](#6-sécurité-et-ssl)
7. [Vérification du Déploiement](#7-vérification-du-déploiement)
8. [Maintenance](#8-maintenance)

## 1. Prérequis

### 1.1 Compétences requises

- Connaissances en administration système Linux
- Familiarité avec Docker et les conteneurs
- Compréhension des bases de données PostgreSQL
- Expérience avec les serveurs web (Nginx, Apache)

### 1.2 Infrastructure nécessaire

- Serveur pour le backend (recommandé : 2 CPU, 4 Go RAM minimum)
- Service d'hébergement statique pour le frontend
- Base de données PostgreSQL avec extension PostGIS
- Noms de domaine pour le frontend et l'API backend
- Certificats SSL

### 1.3 Logiciels requis

- Docker et Docker Compose
- Node.js 16+ (pour le build du frontend)
- Python 3.9+ (pour le backend)
- Git

## 2. Déploiement du Frontend

### 2.1 Construction du build de production

```bash
# Cloner le dépôt
git clone https://github.com/votre-organisation/ferme-virtuelle.git
cd ferme-virtuelle/frontend

# Installer les dépendances
npm install

# Configurer les variables d'environnement
cp .env.example .env.production
# Éditer .env.production avec les valeurs appropriées
# REACT_APP_API_URL=https://api.votre-domaine.com
# REACT_APP_MAPBOX_TOKEN=votre_token_mapbox

# Construire l'application
npm run build
```

### 2.2 Déploiement sur un service d'hébergement statique

#### Option 1 : Netlify

1. Créez un compte sur [Netlify](https://www.netlify.com/)
2. Depuis le tableau de bord, cliquez sur "New site from Git"
3. Connectez votre dépôt Git et configurez :
   - Build command: `npm run build`
   - Publish directory: `build`
4. Configurez les variables d'environnement dans Settings > Build & deploy > Environment
5. Déployez le site

#### Option 2 : Vercel

1. Créez un compte sur [Vercel](https://vercel.com/)
2. Depuis le tableau de bord, importez votre projet Git
3. Configurez le projet :
   - Framework Preset: Create React App
   - Build Command: `npm run build`
   - Output Directory: `build`
4. Configurez les variables d'environnement dans Settings > Environment Variables
5. Déployez le site

#### Option 3 : Serveur web traditionnel (Nginx)

1. Transférez les fichiers du répertoire `build` vers votre serveur web
2. Configurez Nginx :

```nginx
server {
    listen 80;
    server_name votre-domaine.com www.votre-domaine.com;
    
    root /chemin/vers/build;
    index index.html;
    
    location / {
        try_files $uri $uri/ /index.html;
    }
    
    # Configuration pour le cache et la compression
    location ~* \.(js|css|png|jpg|jpeg|gif|ico)$ {
        expires 1y;
        add_header Cache-Control "public, max-age=31536000";
    }
    
    gzip on;
    gzip_types text/plain text/css application/json application/javascript text/xml application/xml application/xml+rss text/javascript;
}
```

3. Redémarrez Nginx : `sudo systemctl restart nginx`

## 3. Déploiement du Backend

### 3.1 Utilisation de Docker (recommandé)

1. Naviguez vers le répertoire du backend :
```bash
cd ferme-virtuelle/backend
```

2. Créez un fichier `.env` pour les variables d'environnement :
```
FLASK_ENV=production
SECRET_KEY=votre_clé_secrète_très_longue_et_aléatoire
DATABASE_URI=postgresql://utilisateur:mot_de_passe@hôte:port/nom_base
OPENWEATHERMAP_API_KEY=votre_clé_api
CORS_ORIGINS=https://votre-domaine-frontend.com
```

3. Construisez et démarrez les conteneurs avec Docker Compose :
```bash
docker-compose -f docker-compose.prod.yml up -d
```

### 3.2 Déploiement manuel

1. Créez un environnement virtuel Python :
```bash
python -m venv venv
source venv/bin/activate  # Sur Windows : venv\Scripts\activate
```

2. Installez les dépendances :
```bash
pip install -r requirements.txt
```

3. Configurez les variables d'environnement :
```bash
export FLASK_ENV=production
export SECRET_KEY=votre_clé_secrète
export DATABASE_URI=postgresql://utilisateur:mot_de_passe@hôte:port/nom_base
export OPENWEATHERMAP_API_KEY=votre_clé_api
export CORS_ORIGINS=https://votre-domaine-frontend.com
```

4. Exécutez l'application avec Gunicorn :
```bash
gunicorn -w 4 -b 0.0.0.0:5000 "src.main:create_app()"
```

5. Configurez Nginx comme proxy inverse :
```nginx
server {
    listen 80;
    server_name api.votre-domaine.com;
    
    location / {
        proxy_pass http://localhost:5000;
        proxy_set_header Host $host;
        proxy_set_header X-Real-IP $remote_addr;
        proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
        proxy_set_header X-Forwarded-Proto $scheme;
    }
}
```

6. Redémarrez Nginx : `sudo systemctl restart nginx`

### 3.3 Déploiement sur un service PaaS

#### Option : Heroku

1. Créez une application Heroku :
```bash
heroku create votre-app-backend
```

2. Ajoutez l'add-on PostgreSQL :
```bash
heroku addons:create heroku-postgresql:hobby-dev
```

3. Configurez les variables d'environnement :
```bash
heroku config:set FLASK_ENV=production
heroku config:set SECRET_KEY=votre_clé_secrète
heroku config:set OPENWEATHERMAP_API_KEY=votre_clé_api
heroku config:set CORS_ORIGINS=https://votre-domaine-frontend.com
```

4. Déployez l'application :
```bash
git push heroku main
```

## 4. Configuration de la Base de Données

### 4.1 PostgreSQL avec PostGIS

1. Installez PostgreSQL et PostGIS :
```bash
sudo apt update
sudo apt install postgresql postgresql-contrib postgis
```

2. Créez une base de données et un utilisateur :
```bash
sudo -u postgres psql

CREATE DATABASE ferme_virtuelle;
CREATE USER ferme_user WITH ENCRYPTED PASSWORD 'mot_de_passe_sécurisé';
GRANT ALL PRIVILEGES ON DATABASE ferme_virtuelle TO ferme_user;
\c ferme_virtuelle
CREATE EXTENSION postgis;
```

3. Configurez l'accès distant si nécessaire (dans `postgresql.conf` et `pg_hba.conf`)

### 4.2 Initialisation de la base de données

1. Exécutez les migrations depuis le backend :
```bash
cd backend
source venv/bin/activate  # Sur Windows : venv\Scripts\activate
flask db upgrade
```

2. Chargez les données initiales (si nécessaire) :
```bash
flask seed-db
```

## 5. Configuration des Services Externes

### 5.1 OpenWeatherMap API

1. Créez un compte sur [OpenWeatherMap](https://openweathermap.org/)
2. Obtenez une clé API
3. Configurez la clé dans les variables d'environnement du backend

### 5.2 Services Cartographiques

#### Option 1 : Mapbox

1. Créez un compte sur [Mapbox](https://www.mapbox.com/)
2. Obtenez un token d'accès
3. Configurez le token dans les variables d'environnement du frontend

#### Option 2 : OpenStreetMap

1. Configurez le frontend pour utiliser les tuiles OpenStreetMap
2. Respectez les [conditions d'utilisation](https://operations.osmfoundation.org/policies/tiles/)

## 6. Sécurité et SSL

### 6.1 Certificats SSL avec Let's Encrypt

1. Installez Certbot :
```bash
sudo apt install certbot python3-certbot-nginx
```

2. Obtenez et configurez les certificats :
```bash
sudo certbot --nginx -d votre-domaine.com -d www.votre-domaine.com
sudo certbot --nginx -d api.votre-domaine.com
```

3. Configurez le renouvellement automatique :
```bash
sudo systemctl status certbot.timer
```

### 6.2 Configuration de sécurité

1. Configurez les en-têtes de sécurité dans Nginx :
```nginx
add_header X-Content-Type-Options nosniff;
add_header X-Frame-Options SAMEORIGIN;
add_header X-XSS-Protection "1; mode=block";
add_header Content-Security-Policy "default-src 'self'; script-src 'self' 'unsafe-inline' 'unsafe-eval'; style-src 'self' 'unsafe-inline'; img-src 'self' data: https:; font-src 'self' data:; connect-src 'self' https://api.votre-domaine.com https://api.openweathermap.org https://api.mapbox.com;";
```

2. Configurez le pare-feu :
```bash
sudo ufw allow 80/tcp
sudo ufw allow 443/tcp
sudo ufw enable
```

## 7. Vérification du Déploiement

### 7.1 Tests de base

1. Vérifiez que le frontend est accessible : `https://votre-domaine.com`
2. Vérifiez que l'API backend répond : `https://api.votre-domaine.com/api/health`
3. Testez le processus d'inscription et de connexion
4. Vérifiez la création et la visualisation de fermes

### 7.2 Surveillance initiale

1. Vérifiez les logs du backend :
```bash
docker-compose -f docker-compose.prod.yml logs -f backend
```

2. Surveillez les performances du serveur :
```bash
htop
```

## 8. Maintenance

### 8.1 Sauvegardes

1. Configurez des sauvegardes régulières de la base de données :
```bash
# Créez un script de sauvegarde
cat > /usr/local/bin/backup-db.sh << 'EOF'
#!/bin/bash
BACKUP_DIR="/chemin/vers/backups"
TIMESTAMP=$(date +"%Y%m%d_%H%M%S")
pg_dump -U ferme_user -h localhost ferme_virtuelle | gzip > "$BACKUP_DIR/ferme_virtuelle_$TIMESTAMP.sql.gz"
find "$BACKUP_DIR" -type f -name "ferme_virtuelle_*.sql.gz" -mtime +30 -delete
EOF

# Rendez-le exécutable
chmod +x /usr/local/bin/backup-db.sh

# Ajoutez-le au crontab pour une exécution quotidienne
(crontab -l 2>/dev/null; echo "0 2 * * * /usr/local/bin/backup-db.sh") | crontab -
```

### 8.2 Mises à jour

1. Pour mettre à jour le frontend :
```bash
cd frontend
git pull
npm install
npm run build
# Redéployez les fichiers build
```

2. Pour mettre à jour le backend :
```bash
cd backend
git pull
source venv/bin/activate  # Sur Windows : venv\Scripts\activate
pip install -r requirements.txt
flask db upgrade
# Redémarrez le service
```

### 8.3 Monitoring

1. Configurez un service de monitoring comme Prometheus + Grafana ou Datadog
2. Mettez en place des alertes pour les problèmes critiques
3. Surveillez régulièrement les logs et les performances

---

Ces instructions fournissent un guide complet pour déployer la plateforme de simulation de ferme virtuelle dans un environnement de production. Adaptez-les selon vos besoins spécifiques et votre infrastructure.
