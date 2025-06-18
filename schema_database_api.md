# Schémas de Base de Données et APIs - Plateforme de Simulation de Ferme Virtuelle

## 1. Introduction

Ce document définit les schémas de base de données et les interfaces de programmation (APIs) pour la plateforme de simulation de ferme virtuelle. Ces définitions sont essentielles pour assurer une communication efficace entre le frontend et le backend, ainsi que pour garantir la persistance et l'intégrité des données.

## 2. Schéma de Base de Données

Nous utiliserons PostgreSQL avec l'extension PostGIS pour bénéficier des fonctionnalités géospatiales avancées, essentielles pour notre application de simulation agricole.

### 2.1 Modèle Entité-Relation

```
┌───────────────┐       ┌───────────────┐       ┌───────────────┐
│   Utilisateur │       │     Ferme     │       │    Parcelle   │
│               │1     *│               │1     *│               │
│ id            │───────│ id            │───────│ id            │
│ email         │       │ nom           │       │ nom           │
│ mot_de_passe  │       │ description   │       │ superficie    │
│ nom           │       │ date_creation │       │ geometrie     │
│ prenom        │       │ utilisateur_id│       │ ferme_id      │
│ role          │       │               │       │ type_sol_id   │
└───────────────┘       └───────────────┘       └───────────────┘
                                                        │
                                                        │*
                                                        ▼1
┌───────────────┐       ┌───────────────┐       ┌───────────────┐
│   Simulation  │       │   Culture     │       │   Type Sol    │
│               │       │               │       │               │
│ id            │       │ id            │       │ id            │
│ date_debut    │*     *│ nom           │       │ nom           │
│ date_fin      │───────│ description   │       │ description   │
│ parametres    │       │ duree_cycle   │       │ texture       │
│ resultats     │       │ besoin_eau    │       │ ph            │
│ ferme_id      │       │ besoin_nutriments│    │ matiere_organique│
└───────────────┘       └───────────────┘       └───────────────┘
        │                       │
        │                       │
        ▼                       ▼
┌───────────────┐       ┌───────────────┐       ┌───────────────┐
│ Donnee Meteo  │       │ Culture Parcelle│     │ Evenement     │
│               │       │               │       │               │
│ id            │       │ id            │       │ id            │
│ date          │       │ parcelle_id   │       │ type          │
│ temperature   │       │ culture_id    │       │ date          │
│ precipitation │       │ date_semis    │       │ description   │
│ humidite      │       │ date_recolte  │       │ parcelle_id   │
│ vent          │       │ rendement     │       │ culture_id    │
│ ferme_id      │       │ etat          │       │               │
└───────────────┘       └───────────────┘       └───────────────┘
```

### 2.2 Description des Tables

#### 2.2.1 Utilisateur

Stocke les informations relatives aux utilisateurs de la plateforme.

| Champ        | Type         | Description                                      |
|--------------|--------------|--------------------------------------------------|
| id           | UUID         | Identifiant unique de l'utilisateur              |
| email        | VARCHAR(255) | Adresse email (unique)                           |
| mot_de_passe | VARCHAR(255) | Mot de passe hashé                               |
| nom          | VARCHAR(100) | Nom de famille                                   |
| prenom       | VARCHAR(100) | Prénom                                           |
| role         | ENUM         | Rôle (étudiant, agriculteur, admin)              |
| date_creation| TIMESTAMP    | Date de création du compte                       |

#### 2.2.2 Ferme

Représente une exploitation agricole virtuelle créée par un utilisateur.

| Champ         | Type         | Description                                     |
|---------------|--------------|--------------------------------------------------|
| id            | UUID         | Identifiant unique de la ferme                   |
| nom           | VARCHAR(255) | Nom de la ferme                                  |
| description   | TEXT         | Description détaillée                            |
| date_creation | TIMESTAMP    | Date de création                                 |
| utilisateur_id| UUID         | Référence à l'utilisateur propriétaire           |
| localisation  | GEOGRAPHY    | Coordonnées géographiques du centre de la ferme  |
| superficie    | FLOAT        | Superficie totale en hectares                    |

#### 2.2.3 Parcelle

Représente une subdivision de la ferme, avec ses caractéristiques propres.

| Champ       | Type         | Description                                      |
|-------------|--------------|--------------------------------------------------|
| id          | UUID         | Identifiant unique de la parcelle                |
| nom         | VARCHAR(255) | Nom de la parcelle                               |
| superficie  | FLOAT        | Superficie en hectares                           |
| geometrie   | GEOMETRY     | Forme géométrique de la parcelle (polygone)      |
| ferme_id    | UUID         | Référence à la ferme                             |
| type_sol_id | UUID         | Référence au type de sol                         |

#### 2.2.4 Type Sol

Définit les différents types de sols disponibles dans la simulation.

| Champ            | Type         | Description                                 |
|------------------|--------------|---------------------------------------------|
| id               | UUID         | Identifiant unique du type de sol           |
| nom              | VARCHAR(100) | Nom du type de sol                          |
| description      | TEXT         | Description détaillée                       |
| texture          | VARCHAR(50)  | Texture (sableux, argileux, limoneux, etc.) |
| ph               | FLOAT        | Niveau de pH                                |
| matiere_organique| FLOAT        | Pourcentage de matière organique            |
| capacite_retention| FLOAT       | Capacité de rétention d'eau                 |

#### 2.2.5 Culture

Catalogue des cultures disponibles pour la simulation.

| Champ             | Type         | Description                                |
|-------------------|--------------|-------------------------------------------|
| id                | UUID         | Identifiant unique de la culture          |
| nom               | VARCHAR(255) | Nom de la culture                         |
| description       | TEXT         | Description détaillée                     |
| duree_cycle       | INTEGER      | Durée du cycle en jours                   |
| besoin_eau        | FLOAT        | Besoin en eau (mm/jour)                   |
| besoin_nutriments | JSONB        | Besoins en nutriments (N, P, K, etc.)     |
| temperature_min   | FLOAT        | Température minimale de croissance (°C)   |
| temperature_max   | FLOAT        | Température maximale de croissance (°C)   |
| rendement_potentiel| FLOAT       | Rendement potentiel (tonnes/hectare)      |

#### 2.2.6 Culture Parcelle

Association entre une parcelle et une culture, avec les détails de plantation.

| Champ         | Type         | Description                                    |
|---------------|--------------|------------------------------------------------|
| id            | UUID         | Identifiant unique                             |
| parcelle_id   | UUID         | Référence à la parcelle                        |
| culture_id    | UUID         | Référence à la culture                         |
| date_semis    | DATE         | Date de semis                                  |
| date_recolte  | DATE         | Date prévue/effective de récolte               |
| rendement     | FLOAT        | Rendement réel ou estimé (tonnes/hectare)      |
| etat          | ENUM         | État (semé, en croissance, récolté, etc.)      |
| parametres    | JSONB        | Paramètres spécifiques (densité, profondeur...)| 

#### 2.2.7 Simulation

Enregistre les simulations effectuées par les utilisateurs.

| Champ        | Type         | Description                                     |
|--------------|--------------|------------------------------------------------|
| id           | UUID         | Identifiant unique de la simulation            |
| date_debut   | TIMESTAMP    | Date de début de la simulation                 |
| date_fin     | TIMESTAMP    | Date de fin de la simulation                   |
| parametres   | JSONB        | Paramètres de la simulation                    |
| resultats    | JSONB        | Résultats de la simulation                     |
| ferme_id     | UUID         | Référence à la ferme                           |

#### 2.2.8 Donnee Meteo

Stocke les données météorologiques historiques et prévisionnelles.

| Champ         | Type         | Description                                    |
|---------------|--------------|------------------------------------------------|
| id            | UUID         | Identifiant unique                             |
| date          | DATE         | Date de la donnée météo                        |
| temperature_min| FLOAT       | Température minimale (°C)                      |
| temperature_max| FLOAT       | Température maximale (°C)                      |
| precipitation | FLOAT        | Précipitations (mm)                            |
| humidite      | FLOAT        | Humidité relative (%)                          |
| vent          | FLOAT        | Vitesse du vent (km/h)                         |
| ensoleillement| FLOAT        | Heures d'ensoleillement                        |
| ferme_id      | UUID         | Référence à la ferme                           |

#### 2.2.9 Evenement

Enregistre les événements survenant sur les parcelles (irrigation, fertilisation, etc.).

| Champ        | Type         | Description                                     |
|--------------|--------------|------------------------------------------------|
| id           | UUID         | Identifiant unique                             |
| type         | ENUM         | Type d'événement (irrigation, fertilisation...) |
| date         | TIMESTAMP    | Date de l'événement                            |
| description  | TEXT         | Description détaillée                          |
| parametres   | JSONB        | Paramètres spécifiques à l'événement           |
| parcelle_id  | UUID         | Référence à la parcelle                        |
| culture_id   | UUID         | Référence à la culture (optionnel)             |

## 3. API REST

L'API REST permettra la communication entre le frontend et le backend. Elle suivra les principes RESTful et utilisera le format JSON pour les échanges de données.

### 3.1 Authentification

#### 3.1.1 Inscription

```
POST /api/auth/register
```

**Corps de la requête :**
```json
{
  "email": "utilisateur@exemple.com",
  "password": "motdepasse123",
  "nom": "Dupont",
  "prenom": "Jean",
  "role": "agriculteur"
}
```

**Réponse (201 Created) :**
```json
{
  "id": "550e8400-e29b-41d4-a716-446655440000",
  "email": "utilisateur@exemple.com",
  "nom": "Dupont",
  "prenom": "Jean",
  "role": "agriculteur",
  "date_creation": "2025-05-20T12:00:00Z"
}
```

#### 3.1.2 Connexion

```
POST /api/auth/login
```

**Corps de la requête :**
```json
{
  "email": "utilisateur@exemple.com",
  "password": "motdepasse123"
}
```

**Réponse (200 OK) :**
```json
{
  "token": "eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9...",
  "user": {
    "id": "550e8400-e29b-41d4-a716-446655440000",
    "email": "utilisateur@exemple.com",
    "nom": "Dupont",
    "prenom": "Jean",
    "role": "agriculteur"
  }
}
```

#### 3.1.3 Déconnexion

```
POST /api/auth/logout
```

**Réponse (200 OK) :**
```json
{
  "message": "Déconnexion réussie"
}
```

### 3.2 Gestion des Fermes

#### 3.2.1 Créer une ferme

```
POST /api/fermes
```

**Corps de la requête :**
```json
{
  "nom": "Ma Ferme Virtuelle",
  "description": "Une ferme expérimentale pour tester différentes cultures",
  "localisation": {
    "type": "Point",
    "coordinates": [2.3522, 48.8566]
  },
  "superficie": 25.5
}
```

**Réponse (201 Created) :**
```json
{
  "id": "550e8400-e29b-41d4-a716-446655440001",
  "nom": "Ma Ferme Virtuelle",
  "description": "Une ferme expérimentale pour tester différentes cultures",
  "date_creation": "2025-05-20T12:30:00Z",
  "utilisateur_id": "550e8400-e29b-41d4-a716-446655440000",
  "localisation": {
    "type": "Point",
    "coordinates": [2.3522, 48.8566]
  },
  "superficie": 25.5
}
```

#### 3.2.2 Récupérer toutes les fermes de l'utilisateur

```
GET /api/fermes
```

**Réponse (200 OK) :**
```json
{
  "fermes": [
    {
      "id": "550e8400-e29b-41d4-a716-446655440001",
      "nom": "Ma Ferme Virtuelle",
      "description": "Une ferme expérimentale pour tester différentes cultures",
      "date_creation": "2025-05-20T12:30:00Z",
      "localisation": {
        "type": "Point",
        "coordinates": [2.3522, 48.8566]
      },
      "superficie": 25.5
    },
    {
      "id": "550e8400-e29b-41d4-a716-446655440002",
      "nom": "Ferme Bio",
      "description": "Ferme dédiée à l'agriculture biologique",
      "date_creation": "2025-05-19T10:15:00Z",
      "localisation": {
        "type": "Point",
        "coordinates": [2.3944, 48.8534]
      },
      "superficie": 15.2
    }
  ]
}
```

#### 3.2.3 Récupérer une ferme spécifique

```
GET /api/fermes/{ferme_id}
```

**Réponse (200 OK) :**
```json
{
  "id": "550e8400-e29b-41d4-a716-446655440001",
  "nom": "Ma Ferme Virtuelle",
  "description": "Une ferme expérimentale pour tester différentes cultures",
  "date_creation": "2025-05-20T12:30:00Z",
  "utilisateur_id": "550e8400-e29b-41d4-a716-446655440000",
  "localisation": {
    "type": "Point",
    "coordinates": [2.3522, 48.8566]
  },
  "superficie": 25.5,
  "parcelles": [
    {
      "id": "550e8400-e29b-41d4-a716-446655440003",
      "nom": "Parcelle Nord",
      "superficie": 10.2,
      "type_sol": {
        "id": "550e8400-e29b-41d4-a716-446655440010",
        "nom": "Limoneux",
        "texture": "limoneux"
      }
    }
  ]
}
```

#### 3.2.4 Mettre à jour une ferme

```
PUT /api/fermes/{ferme_id}
```

**Corps de la requête :**
```json
{
  "nom": "Ma Ferme Virtuelle Modifiée",
  "description": "Description mise à jour"
}
```

**Réponse (200 OK) :**
```json
{
  "id": "550e8400-e29b-41d4-a716-446655440001",
  "nom": "Ma Ferme Virtuelle Modifiée",
  "description": "Description mise à jour",
  "date_creation": "2025-05-20T12:30:00Z",
  "utilisateur_id": "550e8400-e29b-41d4-a716-446655440000",
  "localisation": {
    "type": "Point",
    "coordinates": [2.3522, 48.8566]
  },
  "superficie": 25.5
}
```

#### 3.2.5 Supprimer une ferme

```
DELETE /api/fermes/{ferme_id}
```

**Réponse (204 No Content)**

### 3.3 Gestion des Parcelles

#### 3.3.1 Créer une parcelle

```
POST /api/fermes/{ferme_id}/parcelles
```

**Corps de la requête :**
```json
{
  "nom": "Parcelle Est",
  "superficie": 5.3,
  "geometrie": {
    "type": "Polygon",
    "coordinates": [[[2.352, 48.856], [2.353, 48.856], [2.353, 48.857], [2.352, 48.857], [2.352, 48.856]]]
  },
  "type_sol_id": "550e8400-e29b-41d4-a716-446655440010"
}
```

**Réponse (201 Created) :**
```json
{
  "id": "550e8400-e29b-41d4-a716-446655440004",
  "nom": "Parcelle Est",
  "superficie": 5.3,
  "geometrie": {
    "type": "Polygon",
    "coordinates": [[[2.352, 48.856], [2.353, 48.856], [2.353, 48.857], [2.352, 48.857], [2.352, 48.856]]]
  },
  "ferme_id": "550e8400-e29b-41d4-a716-446655440001",
  "type_sol": {
    "id": "550e8400-e29b-41d4-a716-446655440010",
    "nom": "Limoneux",
    "texture": "limoneux"
  }
}
```

#### 3.3.2 Récupérer toutes les parcelles d'une ferme

```
GET /api/fermes/{ferme_id}/parcelles
```

**Réponse (200 OK) :**
```json
{
  "parcelles": [
    {
      "id": "550e8400-e29b-41d4-a716-446655440003",
      "nom": "Parcelle Nord",
      "superficie": 10.2,
      "type_sol": {
        "id": "550e8400-e29b-41d4-a716-446655440010",
        "nom": "Limoneux"
      },
      "cultures": [
        {
          "id": "550e8400-e29b-41d4-a716-446655440020",
          "culture": {
            "id": "550e8400-e29b-41d4-a716-446655440015",
            "nom": "Blé"
          },
          "date_semis": "2025-03-15",
          "etat": "en_croissance"
        }
      ]
    },
    {
      "id": "550e8400-e29b-41d4-a716-446655440004",
      "nom": "Parcelle Est",
      "superficie": 5.3,
      "type_sol": {
        "id": "550e8400-e29b-41d4-a716-446655440010",
        "nom": "Limoneux"
      },
      "cultures": []
    }
  ]
}
```

### 3.4 Gestion des Cultures

#### 3.4.1 Récupérer le catalogue de cultures

```
GET /api/cultures
```

**Réponse (200 OK) :**
```json
{
  "cultures": [
    {
      "id": "550e8400-e29b-41d4-a716-446655440015",
      "nom": "Blé",
      "description": "Blé tendre d'hiver",
      "duree_cycle": 270,
      "besoin_eau": 4.5,
      "temperature_min": 3,
      "temperature_max": 25,
      "rendement_potentiel": 8.5
    },
    {
      "id": "550e8400-e29b-41d4-a716-446655440016",
      "nom": "Maïs",
      "description": "Maïs grain",
      "duree_cycle": 150,
      "besoin_eau": 6.2,
      "temperature_min": 10,
      "temperature_max": 32,
      "rendement_potentiel": 12.0
    }
  ]
}
```

#### 3.4.2 Planter une culture sur une parcelle

```
POST /api/parcelles/{parcelle_id}/cultures
```

**Corps de la requête :**
```json
{
  "culture_id": "550e8400-e29b-41d4-a716-446655440016",
  "date_semis": "2025-04-15",
  "parametres": {
    "densite": 80000,
    "profondeur": 5
  }
}
```

**Réponse (201 Created) :**
```json
{
  "id": "550e8400-e29b-41d4-a716-446655440021",
  "parcelle_id": "550e8400-e29b-41d4-a716-446655440004",
  "culture": {
    "id": "550e8400-e29b-41d4-a716-446655440016",
    "nom": "Maïs"
  },
  "date_semis": "2025-04-15",
  "date_recolte_estimee": "2025-09-12",
  "etat": "planifie",
  "parametres": {
    "densite": 80000,
    "profondeur": 5
  }
}
```

### 3.5 Simulation

#### 3.5.1 Lancer une simulation

```
POST /api/fermes/{ferme_id}/simulations
```

**Corps de la requête :**
```json
{
  "date_debut": "2025-05-01",
  "date_fin": "2025-10-31",
  "parametres": {
    "pas_temps": "jour",
    "scenarios_meteo": "normal"
  }
}
```

**Réponse (202 Accepted) :**
```json
{
  "id": "550e8400-e29b-41d4-a716-446655440030",
  "ferme_id": "550e8400-e29b-41d4-a716-446655440001",
  "date_debut": "2025-05-01",
  "date_fin": "2025-10-31",
  "statut": "en_cours",
  "progression": 0
}
```

#### 3.5.2 Récupérer le statut d'une simulation

```
GET /api/simulations/{simulation_id}
```

**Réponse (200 OK) :**
```json
{
  "id": "550e8400-e29b-41d4-a716-446655440030",
  "ferme_id": "550e8400-e29b-41d4-a716-446655440001",
  "date_debut": "2025-05-01",
  "date_fin": "2025-10-31",
  "statut": "en_cours",
  "progression": 45,
  "resultats_partiels": {
    "rendements": {
      "550e8400-e29b-41d4-a716-446655440020": 7.2,
      "550e8400-e29b-41d4-a716-446655440021": 10.5
    }
  }
}
```

#### 3.5.3 Récupérer les résultats d'une simulation terminée

```
GET /api/simulations/{simulation_id}/resultats
```

**Réponse (200 OK) :**
```json
{
  "id": "550e8400-e29b-41d4-a716-446655440030",
  "ferme_id": "550e8400-e29b-41d4-a716-446655440001",
  "date_debut": "2025-05-01",
  "date_fin": "2025-10-31",
  "statut": "termine",
  "resultats": {
    "rendements": {
      "550e8400-e29b-41d4-a716-446655440020": 7.8,
      "550e8400-e29b-41d4-a716-446655440021": 11.2
    },
    "consommation_eau": {
      "550e8400-e29b-41d4-a716-446655440020": 450,
      "550e8400-e29b-41d4-a716-446655440021": 620
    },
    "bilan_nutriments": {
      "550e8400-e29b-41d4-a716-446655440020": {
        "N": -120,
        "P": -45,
        "K": -80
      },
      "550e8400-e29b-41d4-a716-446655440021": {
        "N": -180,
        "P": -60,
        "K": -120
      }
    },
    "indicateurs_environnementaux": {
      "emission_co2": 2.5,
      "biodiversite": 65
    }
  }
}
```

### 3.6 Données Météorologiques

#### 3.6.1 Récupérer les données météo historiques

```
GET /api/fermes/{ferme_id}/meteo?debut=2025-01-01&fin=2025-05-20
```

**Réponse (200 OK) :**
```json
{
  "donnees": [
    {
      "date": "2025-01-01",
      "temperature_min": -2.5,
      "temperature_max": 5.2,
      "precipitation": 0.0,
      "humidite": 85,
      "vent": 12.5,
      "ensoleillement": 2.3
    },
    {
      "date": "2025-01-02",
      "temperature_min": -1.8,
      "temperature_max": 6.5,
      "precipitation": 2.5,
      "humidite": 90,
      "vent": 8.2,
      "ensoleillement": 1.5
    }
  ]
}
```

#### 3.6.2 Récupérer les prévisions météo

```
GET /api/fermes/{ferme_id}/meteo/previsions
```

**Réponse (200 OK) :**
```json
{
  "previsions": [
    {
      "date": "2025-05-21",
      "temperature_min": 12.5,
      "temperature_max": 24.8,
      "precipitation": 0.0,
      "humidite": 65,
      "vent": 10.2,
      "ensoleillement": 8.5
    },
    {
      "date": "2025-05-22",
      "temperature_min": 14.2,
      "temperature_max": 26.5,
      "precipitation": 0.0,
      "humidite": 60,
      "vent": 8.5,
      "ensoleillement": 9.2
    }
  ]
}
```

### 3.7 Types de Sol

#### 3.7.1 Récupérer tous les types de sol

```
GET /api/types-sol
```

**Réponse (200 OK) :**
```json
{
  "types_sol": [
    {
      "id": "550e8400-e29b-41d4-a716-446655440010",
      "nom": "Limoneux",
      "description": "Sol à dominante limoneuse, fertile et bien drainé",
      "texture": "limoneux",
      "ph": 6.8,
      "matiere_organique": 3.5,
      "capacite_retention": 0.25
    },
    {
      "id": "550e8400-e29b-41d4-a716-446655440011",
      "nom": "Argileux",
      "description": "Sol à dominante argileuse, riche en nutriments mais drainage limité",
      "texture": "argileux",
      "ph": 7.2,
      "matiere_organique": 2.8,
      "capacite_retention": 0.35
    }
  ]
}
```

### 3.8 Événements

#### 3.8.1 Ajouter un événement

```
POST /api/parcelles/{parcelle_id}/evenements
```

**Corps de la requête :**
```json
{
  "type": "irrigation",
  "date": "2025-06-15T10:30:00Z",
  "description": "Irrigation par aspersion",
  "parametres": {
    "quantite": 25,
    "duree": 120
  },
  "culture_id": "550e8400-e29b-41d4-a716-446655440021"
}
```

**Réponse (201 Created) :**
```json
{
  "id": "550e8400-e29b-41d4-a716-446655440040",
  "type": "irrigation",
  "date": "2025-06-15T10:30:00Z",
  "description": "Irrigation par aspersion",
  "parametres": {
    "quantite": 25,
    "duree": 120
  },
  "parcelle_id": "550e8400-e29b-41d4-a716-446655440004",
  "culture_id": "550e8400-e29b-41d4-a716-446655440021"
}
```

#### 3.8.2 Récupérer les événements d'une parcelle

```
GET /api/parcelles/{parcelle_id}/evenements
```

**Réponse (200 OK) :**
```json
{
  "evenements": [
    {
      "id": "550e8400-e29b-41d4-a716-446655440040",
      "type": "irrigation",
      "date": "2025-06-15T10:30:00Z",
      "description": "Irrigation par aspersion",
      "parametres": {
        "quantite": 25,
        "duree": 120
      },
      "culture": {
        "id": "550e8400-e29b-41d4-a716-446655440021",
        "nom": "Maïs"
      }
    },
    {
      "id": "550e8400-e29b-41d4-a716-446655440041",
      "type": "fertilisation",
      "date": "2025-05-10T09:15:00Z",
      "description": "Épandage d'engrais NPK",
      "parametres": {
        "produit": "NPK 15-15-15",
        "quantite": 250
      },
      "culture": {
        "id": "550e8400-e29b-41d4-a716-446655440021",
        "nom": "Maïs"
      }
    }
  ]
}
```

## 4. Intégration des APIs Externes

### 4.1 API OpenWeatherMap

L'intégration avec OpenWeatherMap permettra de récupérer les données météorologiques actuelles et prévisionnelles.

#### 4.1.1 Configuration

```python
# Configuration dans le backend
OPENWEATHERMAP_API_KEY = os.getenv('OPENWEATHERMAP_API_KEY')
OPENWEATHERMAP_BASE_URL = 'https://api.openweathermap.org/data/2.5'
```

#### 4.1.2 Endpoints Utilisés

- **Météo actuelle** : `/weather?lat={lat}&lon={lon}&units=metric&appid={API_KEY}`
- **Prévisions** : `/forecast?lat={lat}&lon={lon}&units=metric&appid={API_KEY}`
- **Données historiques** : `/onecall/timemachine?lat={lat}&lon={lon}&dt={timestamp}&units=metric&appid={API_KEY}`

### 4.2 API Cartographique (OpenStreetMap/Mapbox)

L'intégration avec les APIs cartographiques permettra d'afficher et d'interagir avec les cartes.

#### 4.2.1 Configuration Mapbox

```javascript
// Configuration dans le frontend
const MAPBOX_ACCESS_TOKEN = process.env.REACT_APP_MAPBOX_ACCESS_TOKEN;
const mapboxgl = require('mapbox-gl');
mapboxgl.accessToken = MAPBOX_ACCESS_TOKEN;
```

#### 4.2.2 Services Utilisés

- **Affichage de cartes** : Mapbox GL JS ou Leaflet avec tuiles OSM
- **Géocodage** : Conversion d'adresses en coordonnées
- **Calcul de surfaces** : Calcul de la superficie des parcelles dessinées

## 5. Sécurité et Validation

### 5.1 Authentification et Autorisation

- **JWT (JSON Web Tokens)** pour l'authentification
- **Middleware d'autorisation** pour vérifier les droits d'accès
- **Expiration des tokens** après une période définie

### 5.2 Validation des Données

- **Validation côté client** avec des bibliothèques comme Formik et Yup
- **Validation côté serveur** avec des schémas de validation
- **Sanitisation des entrées** pour prévenir les injections

### 5.3 Protection CSRF

- **Tokens CSRF** pour les formulaires
- **SameSite cookies** pour limiter les requêtes cross-origin

## 6. Pagination et Filtrage

Pour optimiser les performances et l'expérience utilisateur, les endpoints retournant de grandes quantités de données supporteront la pagination et le filtrage.

### 6.1 Pagination

```
GET /api/fermes?page=2&limit=10
```

**Réponse (200 OK) :**
```json
{
  "fermes": [...],
  "pagination": {
    "total": 45,
    "page": 2,
    "limit": 10,
    "pages": 5
  }
}
```

### 6.2 Filtrage

```
GET /api/cultures?type=cereale&besoin_eau_max=5
```

## 7. Gestion des Erreurs

### 7.1 Format des Réponses d'Erreur

```json
{
  "error": {
    "code": "RESOURCE_NOT_FOUND",
    "message": "La ressource demandée n'existe pas",
    "details": {
      "resource": "ferme",
      "id": "550e8400-e29b-41d4-a716-446655440099"
    }
  }
}
```

### 7.2 Codes d'Erreur HTTP

- **400 Bad Request** : Requête mal formée ou données invalides
- **401 Unauthorized** : Authentification requise
- **403 Forbidden** : Accès non autorisé
- **404 Not Found** : Ressource non trouvée
- **409 Conflict** : Conflit avec l'état actuel de la ressource
- **422 Unprocessable Entity** : Données valides mais traitement impossible
- **500 Internal Server Error** : Erreur serveur

## 8. Conclusion

Ce document définit les schémas de base de données et les APIs nécessaires pour la plateforme de simulation de ferme virtuelle. Ces spécifications serviront de guide pour le développement du backend et du frontend, assurant une communication efficace entre les différentes parties du système.

La conception modulaire et l'utilisation de standards établis (REST, JSON, JWT) garantissent l'évolutivité et la maintenabilité de la plateforme, tout en facilitant les futures extensions vers la réalité virtuelle et augmentée.
