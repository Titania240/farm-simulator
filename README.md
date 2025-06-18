# README - Plateforme de Simulation de Ferme Virtuelle

## Présentation du Projet

Bienvenue dans le projet de plateforme interactive de simulation de ferme virtuelle. Cette application web permet aux utilisateurs (étudiants, agriculteurs, passionnés d'environnement) de créer, gérer et simuler des exploitations agricoles virtuelles basées sur des données réelles (géospatiales, climatiques, pédologiques, etc.).

## Structure du Livrable

Ce livrable contient tous les éléments nécessaires pour comprendre, déployer et faire évoluer la plateforme :

### Documentation

- `analyse_besoins.md` : Analyse détaillée des besoins fonctionnels et non-fonctionnels
- `architecture_globale.md` : Description de l'architecture technique globale
- `schema_database_api.md` : Définition des schémas de base de données et des APIs
- `prototype_mvp.md` : Plan de développement du prototype minimal viable
- `documentation_technique.md` : Documentation technique complète
- `guide_utilisateur.md` : Guide d'utilisation destiné aux utilisateurs finaux
- `instructions_deploiement.md` : Instructions détaillées pour le déploiement
- `guide_validation.md` : Guide pour la validation du prototype
- `todo.md` : Liste des tâches réalisées pendant le développement

### Code Source

- `/frontend` : Application React avec TypeScript et Tailwind CSS
- `/backend` : API Flask avec SQLAlchemy et support PostgreSQL/PostGIS

## Fonctionnalités Principales

- Visualisation d'une carte interactive (basée sur OpenStreetMap ou Google Maps)
- Sélection et configuration de parcelles de terrain virtuelles
- Choix et configuration de cultures avec paramètres agricoles
- Simulation des rendements en fonction des données météorologiques et du type de sol
- Tableau de bord pour suivre l'évolution de la ferme
- Système d'authentification et de gestion des comptes utilisateurs

## Technologies Utilisées

### Frontend
- React.js avec TypeScript
- Tailwind CSS pour le design responsive
- Leaflet.js pour la cartographie interactive
- Recharts pour la visualisation de données

### Backend
- Flask (Python) pour l'API REST
- SQLAlchemy comme ORM
- PostgreSQL avec PostGIS pour les données géospatiales
- JWT pour l'authentification

### Services Externes
- OpenWeatherMap API pour les données météorologiques
- OpenStreetMap/Google Maps pour les données cartographiques

## Démarrage Rapide

Pour une installation rapide en environnement de développement :

1. **Frontend** :
   ```bash
   cd frontend/farm-simulator
   npm install
   npm run dev
   ```

2. **Backend** :
   ```bash
   cd backend/farm-simulator-api
   source venv/bin/activate  # Sur Windows : venv\Scripts\activate
   pip install -r requirements.txt
   flask run
   ```

Pour un déploiement en production, veuillez consulter le document `instructions_deploiement.md`.

## Extension Future

La plateforme a été conçue pour être extensible vers :
- La réalité virtuelle (VR)
- La réalité augmentée (AR)
- L'intégration de modèles de simulation agricole plus avancés
- Des fonctionnalités collaboratives et sociales

## Contact

Pour toute question ou assistance concernant ce projet, veuillez contacter l'équipe de développement.

---

© 2025 Plateforme de Simulation de Ferme Virtuelle - Tous droits réservés
