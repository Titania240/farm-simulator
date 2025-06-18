# Documentation Technique - Plateforme de Simulation de Ferme Virtuelle

## 1. Introduction

Cette documentation technique détaille l'architecture, les technologies et les composants de la plateforme de simulation de ferme virtuelle. Elle est destinée aux développeurs et administrateurs système qui souhaitent comprendre, maintenir ou étendre le système.

## 2. Architecture Globale

La plateforme suit une architecture client-serveur moderne avec séparation claire entre le frontend et le backend :

### 2.1 Vue d'ensemble

```
┌─────────────────────────────────────────────────────────────────┐
│                        CLIENT (NAVIGATEUR)                       │
│                                                                 │
│  ┌─────────────────────────────────────────────────────────┐   │
│  │                    APPLICATION REACT                     │   │
│  │                                                         │   │
│  │  ┌─────────────┐  ┌─────────────┐  ┌─────────────────┐  │   │
│  │  │    Pages    │  │ Composants  │  │     Hooks &     │  │   │
│  │  │             │  │             │  │     Context     │  │   │
│  │  └─────────────┘  └─────────────┘  └─────────────────┘  │   │
│  │                                                         │   │
│  │  ┌─────────────┐  ┌─────────────┐  ┌─────────────────┐  │   │
│  │  │  Services   │  │    Store    │  │  Utilitaires &  │  │   │
│  │  │   (API)     │  │   (Redux)   │  │    Helpers      │  │   │
│  │  └─────────────┘  └─────────────┘  └─────────────────┘  │   │
│  │                                                         │   │
│  │  ┌─────────────────────┐  ┌───────────────────────────┐  │   │
│  │  │  Module Cartographie│  │ Module Visualisation Data │  │   │
│  │  │  (Leaflet/Mapbox)   │  │      (Recharts/D3)        │  │   │
│  │  └─────────────────────┘  └───────────────────────────┘  │   │
│  └─────────────────────────────────────────────────────────┘   │
└─────────────────────────────────────────────────────────────────┘
                               ▲
                               │
                               ▼
┌─────────────────────────────────────────────────────────────────┐
│                         SERVEUR (BACKEND)                        │
│                                                                 │
│  ┌─────────────────────────────────────────────────────────┐   │
│  │                  API FLASK (PYTHON)                      │   │
│  │                                                         │   │
│  │  ┌─────────────┐  ┌─────────────┐  ┌─────────────────┐  │   │
│  │  │   Routes    │  │ Contrôleurs │  │   Middleware    │  │   │
│  │  │             │  │             │  │                 │  │   │
│  │  └─────────────┘  └─────────────┘  └─────────────────┘  │   │
│  │                                                         │   │
│  │  ┌─────────────────────────────────────────────────┐    │   │
│  │  │              Services Métier                     │    │   │
│  │  │                                                 │    │   │
│  │  │  ┌─────────────┐  ┌─────────────────────────┐  │    │   │
│  │  │  │ Gestion des │  │ Moteur de Simulation    │  │    │   │
│  │  │  │ Utilisateurs│  │ Agricole                │  │    │   │
│  │  │  └─────────────┘  └─────────────────────────┘  │    │   │
│  │  │                                                 │    │   │
│  │  │  ┌─────────────┐  ┌─────────────────────────┐  │    │   │
│  │  │  │ Gestion des │  │ Intégration Données     │  │    │   │
│  │  │  │ Cultures    │  │ Externes                │  │    │   │
│  │  │  └─────────────┘  └─────────────────────────┘  │    │   │
│  │  └─────────────────────────────────────────────────┘    │   │
│  └─────────────────────────────────────────────────────────┘   │
│                               ▲                                 │
│                               │                                 │
│                               ▼                                 │
│  ┌─────────────────────────────────────────────────────────┐   │
│  │                     BASE DE DONNÉES                      │   │
│  │                                                         │   │
│  │  ┌─────────────────────┐      ┌─────────────────────┐   │   │
│  │  │     MongoDB         │  OU  │   PostgreSQL        │   │   │
│  │  │     (NoSQL)         │      │   + PostGIS         │   │   │
│  │  └─────────────────────┘      └─────────────────────┘   │   │
│  └─────────────────────────────────────────────────────────┘   │
└─────────────────────────────────────────────────────────────────┘
                               ▲
                               │
                               ▼
┌─────────────────────────────────────────────────────────────────┐
│                       SERVICES EXTERNES                          │
│                                                                 │
│  ┌─────────────┐  ┌─────────────┐  ┌─────────────┐  ┌────────┐  │
│  │ OpenWeather │  │ OpenStreetMap│  │ Google Maps│  │ Autres │  │
│  │     API     │  │     API     │  │    API     │  │  APIs  │  │
│  └─────────────┘  └─────────────┘  └─────────────┘  └────────┘  │
└─────────────────────────────────────────────────────────────────┘
```

### 2.2 Flux de données

1. L'utilisateur interagit avec l'interface React dans le navigateur
2. Les requêtes sont envoyées à l'API Flask via des appels HTTP
3. Le backend traite les requêtes, interagit avec la base de données et les services externes
4. Les réponses sont renvoyées au frontend pour affichage
5. Les mises à jour en temps réel utilisent des requêtes périodiques ou WebSockets

## 3. Technologies Utilisées

### 3.1 Frontend

- **Framework principal** : React.js avec TypeScript
- **Gestion d'état** : Context API (pour le MVP, extensible à Redux)
- **Routage** : React Router
- **Styles** : Tailwind CSS
- **Cartographie** : Leaflet.js (prévu)
- **Visualisation de données** : Recharts (prévu)
- **Formulaires** : Formik avec Yup (validation)
- **Requêtes API** : Axios ou Fetch API

### 3.2 Backend

- **Framework principal** : Flask (Python)
- **ORM** : SQLAlchemy
- **Authentification** : JWT (JSON Web Tokens)
- **Validation de données** : Marshmallow
- **Gestion des requêtes** : Flask-RESTful
- **CORS** : Flask-CORS

### 3.3 Base de données

- **Système principal** : PostgreSQL avec extension PostGIS (recommandé pour les données géospatiales)
- **Alternative** : MongoDB (pour une approche NoSQL)

### 3.4 Services externes

- **Données météorologiques** : OpenWeatherMap API
- **Cartographie** : OpenStreetMap ou Google Maps API
- **Géocodage** : Nominatim ou Google Geocoding API

## 4. Structure du Projet

### 4.1 Frontend (React)

```
frontend/
├── public/                 # Fichiers statiques publics
├── src/
│   ├── assets/             # Images, icônes, etc.
│   ├── components/         # Composants réutilisables
│   │   ├── ui/             # Composants d'interface utilisateur
│   │   ├── forms/          # Composants de formulaire
│   │   ├── maps/           # Composants cartographiques
│   │   └── charts/         # Composants de visualisation
│   ├── hooks/              # Hooks React personnalisés
│   ├── pages/              # Composants de page
│   │   ├── Home.tsx        # Page d'accueil
│   │   ├── Login.tsx       # Page de connexion
│   │   ├── Register.tsx    # Page d'inscription
│   │   ├── Dashboard.tsx   # Tableau de bord
│   │   ├── FarmCreation.tsx # Création de ferme
│   │   └── FarmView.tsx    # Visualisation de ferme
│   ├── services/           # Services d'API et utilitaires
│   │   ├── api.ts          # Client API
│   │   ├── auth.ts         # Service d'authentification
│   │   └── weather.ts      # Service météo
│   ├── context/            # Contextes React
│   │   ├── AuthContext.tsx # Contexte d'authentification
│   │   └── FarmContext.tsx # Contexte de ferme
│   ├── types/              # Définitions de types TypeScript
│   ├── utils/              # Fonctions utilitaires
│   ├── App.tsx             # Composant racine
│   ├── index.tsx           # Point d'entrée
│   └── ...
├── package.json            # Dépendances et scripts
├── tsconfig.json           # Configuration TypeScript
└── ...
```

### 4.2 Backend (Flask)

```
backend/
├── venv/                   # Environnement virtuel Python
├── src/
│   ├── models/             # Modèles de données
│   │   ├── __init__.py
│   │   ├── user.py         # Modèle utilisateur
│   │   ├── farm.py         # Modèle ferme
│   │   ├── parcel.py       # Modèle parcelle
│   │   ├── crop.py         # Modèle culture
│   │   ├── soil.py         # Modèle type de sol
│   │   ├── weather.py      # Modèle données météo
│   │   └── simulation.py   # Modèle simulation
│   ├── routes/             # Routes API
│   │   ├── __init__.py
│   │   ├── auth.py         # Routes d'authentification
│   │   ├── farms.py        # Routes de gestion des fermes
│   │   ├── parcels.py      # Routes de gestion des parcelles
│   │   ├── crops.py        # Routes de gestion des cultures
│   │   ├── simulations.py  # Routes de simulation
│   │   └── weather.py      # Routes météo
│   ├── services/           # Services métier
│   │   ├── __init__.py
│   │   ├── auth_service.py # Service d'authentification
│   │   ├── farm_service.py # Service de gestion des fermes
│   │   ├── weather_service.py # Service météo
│   │   └── simulation_service.py # Service de simulation
│   ├── static/             # Fichiers statiques
│   ├── main.py             # Point d'entrée de l'application
│   └── ...
├── requirements.txt        # Dépendances Python
└── ...
```

## 5. Modèles de Données

### 5.1 Schéma de Base de Données

Le schéma complet est détaillé dans le document `schema_database_api.md`. Voici un résumé des principales entités :

- **Utilisateur** : Informations d'authentification et profil
- **Ferme** : Exploitation agricole virtuelle
- **Parcelle** : Subdivision de la ferme avec géométrie
- **Type de Sol** : Caractéristiques du sol
- **Culture** : Catalogue des cultures disponibles
- **Culture Parcelle** : Association entre parcelle et culture
- **Simulation** : Paramètres et résultats de simulation
- **Donnée Météo** : Données météorologiques historiques et prévisionnelles
- **Événement** : Actions sur les parcelles (irrigation, fertilisation, etc.)

### 5.2 Relations

- Un utilisateur peut avoir plusieurs fermes
- Une ferme contient plusieurs parcelles
- Une parcelle a un type de sol
- Une parcelle peut avoir plusieurs cultures (via Culture Parcelle)
- Une ferme peut avoir plusieurs simulations
- Une ferme a des données météo associées

## 6. API REST

### 6.1 Authentification

- `POST /api/auth/register` : Inscription
- `POST /api/auth/login` : Connexion
- `POST /api/auth/logout` : Déconnexion

### 6.2 Gestion des Fermes

- `GET /api/fermes` : Liste des fermes de l'utilisateur
- `POST /api/fermes` : Création d'une ferme
- `GET /api/fermes/{ferme_id}` : Détails d'une ferme
- `PUT /api/fermes/{ferme_id}` : Mise à jour d'une ferme
- `DELETE /api/fermes/{ferme_id}` : Suppression d'une ferme

### 6.3 Gestion des Parcelles

- `GET /api/fermes/{ferme_id}/parcelles` : Liste des parcelles d'une ferme
- `POST /api/fermes/{ferme_id}/parcelles` : Création d'une parcelle
- `GET /api/parcelles/{parcelle_id}` : Détails d'une parcelle
- `PUT /api/parcelles/{parcelle_id}` : Mise à jour d'une parcelle
- `DELETE /api/parcelles/{parcelle_id}` : Suppression d'une parcelle

### 6.4 Gestion des Cultures

- `GET /api/cultures` : Catalogue de cultures
- `POST /api/parcelles/{parcelle_id}/cultures` : Plantation d'une culture
- `GET /api/parcelles/{parcelle_id}/cultures` : Cultures d'une parcelle

### 6.5 Simulation

- `POST /api/fermes/{ferme_id}/simulations` : Lancement d'une simulation
- `GET /api/simulations/{simulation_id}` : Statut d'une simulation
- `GET /api/simulations/{simulation_id}/resultats` : Résultats d'une simulation

### 6.6 Données Météorologiques

- `GET /api/fermes/{ferme_id}/meteo` : Données météo historiques
- `GET /api/fermes/{ferme_id}/meteo/previsions` : Prévisions météo

## 7. Sécurité

### 7.1 Authentification

- Utilisation de JWT (JSON Web Tokens) pour l'authentification stateless
- Stockage sécurisé des mots de passe avec hachage bcrypt
- Expiration des tokens après une période définie

### 7.2 Autorisation

- Middleware de vérification des droits d'accès
- Contrôle d'accès basé sur les rôles (RBAC)
- Vérification de propriété des ressources

### 7.3 Protection des Données

- Validation des entrées côté client et serveur
- Protection contre les injections SQL via ORM
- Protection CSRF
- Configuration CORS appropriée

## 8. Performance et Scalabilité

### 8.1 Optimisations Frontend

- Code splitting pour chargement à la demande
- Mise en cache des données fréquemment utilisées
- Lazy loading des composants et images
- Compression des assets

### 8.2 Optimisations Backend

- Mise en cache avec Redis (prévu)
- Pagination des résultats
- Indexation optimisée de la base de données
- Traitement asynchrone pour les calculs intensifs

## 9. Déploiement

### 9.1 Environnements

- **Développement** : Local avec serveurs de développement
- **Test** : Environnement de staging
- **Production** : Environnement de production

### 9.2 Infrastructure Recommandée

- **Frontend** : Hébergement statique (Netlify, Vercel, AWS S3)
- **Backend** : Conteneurisation avec Docker, déploiement sur AWS, Google Cloud ou Azure
- **Base de Données** : Services gérés (AWS RDS, MongoDB Atlas)

## 10. Extension VR/AR

L'architecture a été conçue pour permettre une extension future vers la réalité virtuelle et augmentée :

### 10.1 Points d'Extension

- API REST complète accessible pour les clients VR/AR
- Format de données standardisé pour la représentation 3D
- Endpoints dédiés pour les données géospatiales

### 10.2 Technologies Recommandées

- **Web VR** : A-Frame ou React 360
- **VR Native** : Unity ou Unreal Engine
- **AR** : AR.js ou WebXR

## 11. Maintenance et Évolution

### 11.1 Gestion des Versions

- Utilisation de Git pour le contrôle de version
- Branches feature/develop/master
- Versionnage sémantique

### 11.2 Tests

- Tests unitaires avec Jest (frontend) et Pytest (backend)
- Tests d'intégration
- Tests end-to-end avec Cypress ou Playwright

### 11.3 Monitoring

- Logging centralisé
- Surveillance des performances
- Alertes en cas d'anomalies

## 12. Conclusion

Cette documentation technique fournit une vue d'ensemble de l'architecture et des composants de la plateforme de simulation de ferme virtuelle. Elle servira de référence pour le développement, la maintenance et l'évolution future du système.

Pour plus de détails sur des aspects spécifiques, consultez les documents complémentaires :
- `schema_database_api.md` : Détails des schémas de base de données et des APIs
- `guide_utilisateur.md` : Guide d'utilisation de la plateforme
- `instructions_deploiement.md` : Instructions détaillées pour le déploiement
