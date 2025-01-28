# **README – Documentation de l’API Wager**  

Ce document décrit l’API Wager, qui permet de gérer :  
- Des **flags** (fréquences marquées)  
- Des **postbox** (forums de discussion)  
- Des **informations** utilisateur spécifiques  
- Un **chat** en temps réel  

L’API est décrite selon la spécification **OpenAPI 3.0.0** ci-dessus.

---

## Sommaire

1. [Aperçu général](#aperçu-général)  
2. [Fonctionnement de l’API](#fonctionnement-de-lapi)  
3. [Authentification et Sécurité](#authentification-et-sécurité)  
4. [Endpoints](#endpoints)  
   - [Flags](#1-flags)  
   - [Postbox](#2-postbox)  
   - [Info](#3-info)  
   - [Chat](#4-chat)  
5. [Codes de retour (Erreurs)](#codes-de-retour-erreurs)  
6. [Exemples d’utilisation](#exemples-dutilisation)  
   - [Exemple de requête HTTP (cURL)](#exemple-de-requête-http-curl)  
   - [Exemple d’appel avec un token Bearer](#exemple-dappel-avec-un-token-bearer)  
7. [Schémas de données](#schémas-de-données)  
8. [Contact et Licence](#contact-et-licence)  

---

## Aperçu général

- **Nom de l’API** : **Wager API**  
- **Version** : 1.0.0  
- **Description** : API permettant de stocker et récupérer des informations sur des fréquences radio (flags), des discussions (postbox), des chats en direct et des informations utilisateurs.  
- **URL de production** : `https://api.WebSDR.com/v1`  
- **URL de développement** : `http://localhost:3000`  

Les clients (applications externes, front-end, scripts) peuvent utiliser cette API pour créer, lire, modifier et supprimer les différentes ressources listées plus bas.

---

## Fonctionnement de l’API

L’API suit un modèle **REST** :  
- Les **ressources** (flags, postbox, info, chat) sont accessibles via des chemins spécifiques (`/flags`, `/postbox`, etc.).  
- Les **méthodes HTTP** (GET, POST, PUT, PATCH, DELETE) indiquent l’action à effectuer (lire, créer, mettre à jour, supprimer).  
- Les **requêtes** et **réponses** sont généralement au format **JSON** (content-type: `application/json`).  
- Certains endpoints sont **protégés** et nécessitent une **authentification** via un token de type **Bearer** (JWT).

---

## Authentification et Sécurité

- **BearerAuth** (JWT) :  
  - Pour les opérations de création, de modification ou de suppression, vous devez inclure un header :  
    ```http
    Authorization: Bearer <votre_token_jwt>
    ```
  - Les endpoints publics (ex: GET /flags sans restrictions) peuvent ne pas nécessiter de token.  

- **HTTPS** :  
  - Sur le serveur de production (`https://api.WebSDR.com/v1`), l’API est sécurisée via TLS/SSL.  
  - Il est **fortement recommandé** d’utiliser HTTPS pour la confidentialité et l’intégrité des données.

---

## Endpoints

La structure ci-dessous résume les endpoints et leurs principales fonctionnalités. Chaque endpoint dispose de méthodes permettant différentes opérations (GET, POST, PUT, PATCH, DELETE).

### 1) FLAGS

Chemin de base : **`/flags`**  
- **GET** `/flags` : Liste ou recherche de tous les flags existants (avec options de filtrage et pagination).  
- **PUT** `/flags` : Créer ou mettre à jour un flag (basé sur un identifiant unique `nom`).  
- **POST** `/flags` : Créer un nouveau flag **si** aucun flag du même nom n’existe déjà.  
- **PATCH** `/flags` : Mise à jour partielle (ex. juste changer la fréquence ou la région).  
- **DELETE** `/flags` : Supprimer un flag spécifique par son `nom` (query param).

| Paramètres (GET /flags) | Type      | Description                               |
|-------------------------|----------|-------------------------------------------|
| freqMin                 | float    | Filtre par fréquence minimale (MHz)       |
| freqMax                 | float    | Filtre par fréquence maximale (MHz)       |
| page                    | integer  | Pagination – numéro de page (défaut: 1)   |
| limit                   | integer  | Pagination – éléments par page (défaut: 10)|
| sort                    | string   | Tri possible par `freq`, `region` ou `nom`|

---

### 2) POSTBOX

Chemin de base : **`/postbox`**  
- **GET** `/postbox` : Récupère la liste des messages de type forum.  
- **POST** `/postbox` : Crée un nouveau message (ajout au forum).  
- **PUT** `/postbox` : Met à jour complètement un post existant (identifié par query param).  
- **PATCH** `/postbox` : Met à jour partiellement un post (ex. changer seulement le contenu).  
- **DELETE** `/postbox` : Supprime un ou plusieurs posts (via titre ou nom d’utilisateur).

| Paramètres (GET /postbox) | Type   | Description                               |
|---------------------------|--------|-------------------------------------------|
| user                      | string | Filtrer par nom d’utilisateur.           |
| date                      | date   | Filtrer par date (format: YYYY-MM-DD).   |

---

### 3) INFO

Chemin de base : **`/info`**  
- **GET** `/info` : Obtenir les informations d’un utilisateur (nom, date de création, historique).  
- **POST** `/info` : Créer de nouvelles infos utilisateur (si l’utilisateur n’existe pas déjà).  
- **PUT** `/info` : Mettre à jour ou remplacer totalement les infos existantes d’un utilisateur.  
- **PATCH** `/info` : Modifier partiellement les infos (ex. ajout dans l’historique).  
- **DELETE** `/info` : Supprimer complètement les informations d’un utilisateur.

| Paramètres (GET /info)  | Type   | Description                                            |
|-------------------------|--------|--------------------------------------------------------|
| user (requis)           | string | Nom d’utilisateur dont on veut récupérer les infos.   |

---

### 4) CHAT

Chemin de base : **`/chat`**  
- **GET** `/chat` : Liste les messages de chat récents, filtrables par user ou fréquence.  
- **PUT** `/chat` : Envoie un nouveau message (variation standard).  
- **POST** `/chat` : Envoie un nouveau message (variation “toujours créé”).  
- **PATCH** `/chat` : Mise à jour partielle (ex. modifier le contenu).  
- **DELETE** `/chat` : Supprimer un ou plusieurs messages (via nom d’utilisateur et/ou date).

| Paramètres (GET /chat) | Type   | Description                                      |
|------------------------|--------|--------------------------------------------------|
| frequency              | float  | Filtrer les messages par fréquence (MHz).       |
| user                   | string | Filtrer par nom d’utilisateur.                  |

---

## Codes de retour (Erreurs)

En cas d’erreur, l’API renvoie un code HTTP standard (ex. `400`, `401`, `404`, `500`) et un objet JSON suivant le schéma `Error` :

```json
{
  "code": 400,
  "message": "Paramètres de requête invalides"
}
```

Exemples de codes fréquemment utilisés :  

- **200** : OK – La requête a abouti avec succès.  
- **201** : Created – Une ressource a été créée.  
- **204** : No Content – Requête réussie, pas de contenu à renvoyer (ex: après un DELETE).  
- **400** : Bad Request – Paramètres invalides, corps de requête mal formé, etc.  
- **401** : Unauthorized – Token d’authentification absent ou invalide.  
- **404** : Not Found – La ressource demandée n’existe pas.  
- **409** : Conflict – Conflit, par exemple un enregistrement existe déjà.  
- **500** : Internal Server Error – Erreur côté serveur.

---

## Exemples d’utilisation

### Exemple de requête HTTP (cURL)

- Récupérer la liste des flags, triés par région, sur la première page de résultats :

```bash
curl -X GET "http://localhost:3000/flags?sort=region&page=1&limit=5"
```

### Exemple d’appel avec un token Bearer

- Créer un nouveau flag :  

```bash
curl -X POST "http://localhost:3000/flags" \
     -H "Content-Type: application/json" \
     -H "Authorization: Bearer <MON_TOKEN_JWT>" \
     -d '{
       "nom": "RockFM",
       "freq": 250.00,
       "region": "Lyon, FRANCE"
     }'
```

---

## Schémas de données

Les **schemas** figurant dans la section `components.schemas` de la spécification **OpenAPI** décrivent la structure des objets échangés.  

- **Flag** :  
  ```json
  {
    "nom": "NRJ",
    "freq": 190.0,
    "region": "Bordeaux, FRANCE"
  }
  ```  

- **Post** :  
  ```json
  {
    "nom_user": "S7VenStars",
    "date_post": "2025-01-28",
    "titre": "Etranges Frequences sur ...",
    "contenu": "Découverte d'une station inconnue."
  }
  ```  

- **Info** :  
  ```json
  {
    "nom_user": "Bobo",
    "date_creation": "2025-01-30",
    "historique": [
      { "flag": "RadioLibre", "date": "2025-01-30" }
    ]
  }
  ```  

- **Chat** :  
  ```json
  {
    "nom_user": "S7VenStars",
    "date": "2025-01-28T15:56:00Z",
    "contenu": "Wow!"
  }
  ```