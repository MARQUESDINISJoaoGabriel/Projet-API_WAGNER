## 1. Version de la spécification

```yaml
openapi: 3.0.0
```

- **openapi: 3.0.0** : Indique la version du format OpenAPI utilisée.  
  - Ici, c’est **3.0.0**, l’une des plus récentes versions du standard OpenAPI, qui décrit comment structurer la documentation et la configuration d’une API.

---

## 2. Informations générales (info)

```yaml
info:
  title: Wager API
  description: |
    API for managing flags, postboxes (forum-like functionality), live chat,
    and user info sections for a WebSDR-like system...
  version: 1.0.0
  contact:
    name: API Support
    email: support@websdr.com
  license:
    name: Apache 2.0
    url: https://www.apache.org/licenses/LICENSE-2.0.html
```

- **info** : Contient les métadonnées de l’API.  
  - **title** : Le nom de l’API (ici, “Wager API”).  
  - **description** : Un texte libre décrivant ce que fait l’API, à quoi elle sert.  
  - **version** : La version de votre API (pas celle de la spéc OpenAPI), par exemple `1.0.0`.  
  - **contact** : Les informations pour contacter l’équipe en charge (nom, email, etc.).  
  - **license** : Spécifie le type de licence utilisée pour l’API, ici “Apache 2.0”.

Cette section aide les utilisateurs et développeurs à comprendre **qui** a fait l’API, **quelle version** ils utilisent et **où** obtenir de l’aide.

---

## 3. Les serveurs (servers)

```yaml
servers:
  - url: https://api.WebSDR.com/v1
    description: Production server
  - url: http://localhost:3000
    description: Local development server
```

- **servers** : Liste les environnements où tourne l’API.  
  - Ici, on a un serveur de production et un serveur local de développement.  
  - **url** : Indique l’adresse de base de l’API.  
  - **description** : Explique de quel environnement il s’agit.

Cela permet aux clients (par ex. aux développeurs front-end, ou aux outils comme Swagger UI) de savoir vers **quelle URL** faire leurs requêtes.

---

## 4. Les tags (tags)

```yaml
tags:
  - name: Flags
    description: Manage flags and their data...
  - name: Postbox
    description: Forum-like functionality for user posts...
  - ...
```

- **tags** : Servent à **organiser** les différentes routes/points de terminaison (endpoints) de l’API en catégories logiques.  
  - **name** : Nom de la catégorie (ex. “Flags”).  
  - **description** : Brève explication de ce que représente ce tag.

Cela permet d’avoir un regroupement des endpoints par thème, ce qui est très pratique dans l’interface Swagger.

---

## 5. Les chemins (paths)

```yaml
paths:
  /flags:
    get:
      tags:
        - List
      ...
    put:
      tags:
        - Flags
      ...
  /postbox:
    get:
      tags:
        - Postbox
      ...
    post:
      tags:
        - Postbox
      ...
  /info:
    get: ...
    delete: ...
  /chat:
    get: ...
    put: ...
```

- **paths** : **C’est le cœur de l’API**. Chaque **clé** dans `paths` (par ex. `/flags`, `/postbox`, `/info`, `/chat`) représente un **endpoint**.  
- À l’intérieur de chaque endpoint, on a des méthodes HTTP (`get`, `post`, `put`, `delete`, etc.), chacune décrivant **ce que fait** l’API à cette URL.

### Exemple : `/flags`
- **GET** `/flags` : Récupère toutes les “flags”.  
  - On retrouve des paramètres de requête (par ex. `freqMin`, `freqMax`) pour filtrer ou paginer.
  - **responses** (200, 400, etc.) décrivent les codes de statut possibles et les formats de réponse.  
- **PUT** `/flags` : Crée ou met à jour une “flag” (station radio marquée).  
  - **requestBody** : Montre ce qu’on doit envoyer en JSON.  
  - **responses** : Différents scénarios (200, 400, 401, 500…).

### Autres endpoints
- **`/postbox`** : Pour gérer un forum de posts (GET pour lister, POST pour créer).  
- **`/info`** : Pour gérer les infos d’un utilisateur (GET pour récupérer, DELETE pour supprimer).  
- **`/chat`** : Pour récupérer ou envoyer des messages de chat (GET pour lire, PUT pour poster).

Dans chaque méthode, tu vas trouver :  
- **tags** : Indique la catégorie de l’opération.  
- **summary** et **description** : Expliquent en quelques mots la fonctionnalité.  
- **parameters** (facultatif) : Les paramètres passés dans l’URL (par ex. `?page=2`).  
- **requestBody** (facultatif) : Les données envoyées dans le corps (souvent pour `POST` ou `PUT`).  
- **responses** : Les différentes réponses possibles avec code HTTP (`200`, `400`, `404`, etc.) et leur structure.  

---

## 6. Composants (components)

```yaml
components:
  securitySchemes:
    BearerAuth:
      type: http
      scheme: bearer
      bearerFormat: JWT
  
  schemas:
    Flag: ...
    Post: ...
    Chat: ...
    Info: ...
    Error: ...
```

- **components** : Stocke toutes les **ressources réutilisables**.  
  - **securitySchemes** : Définit la manière dont l’API gère la sécurité (ici un token JWT Bearer).  
  - **schemas** : Chaque schéma décrit la **forme** d’un objet (ex. `Flag`, `Post`, `Chat`, `Info`, `Error`).  

### 6.1. `securitySchemes`
- **BearerAuth** : Indique que si l’on protège un endpoint, on utilisera un token Bearer (type JWT).  

### 6.2. `schemas`
- **Flag** : Décrit l’objet JSON d’un “Flag” :  
  - `nom` (string),  
  - `freq` (number, float),  
  - `region` (string), etc.  
- **Post** : Décrit un message de forum (nom d’utilisateur, date de post, titre, contenu).  
- **Chat** : Décrit la structure d’un message de chat (nom_user, date, contenu).  
- **Info** : Décrit les informations de profil d’un utilisateur (nom_user, date_creation, historique).  
- **Error** : Décrit la structure d’une erreur de l’API, par exemple :  
  ```json
  {
    "code": 400,
    "message": "Invalid request parameters"
  }
  ```

Ces schemas sont réutilisés dans la section **`paths`** (lorsque l’on fait référence à `$ref: '#/components/schemas/Flag'`, par exemple).

---

## 7. Pourquoi tout ça est utile ?

- Un fichier OpenAPI (ou Swagger) est **une documentation vivante** de ton API.  
- Les outils (ex. **Swagger UI**, **Swagger Editor**, **Postman**, **Redoc**) peuvent **lire** ce fichier et :  
  - Générer **une page web** de documentation interactive.  
  - **Tester** les endpoints (on clique, ça envoie la requête).  
  - Générer du **code client** (ex. en Node, Python, Java) pour consommer l’API automatiquement.  
  - Générer du **code serveur** (squelettes de contrôleurs) dans certains cas.  

L’objectif est que quiconque souhaite utiliser ton API sache :  
1. **Quels endpoints** existent et à quelle URL ils sont accessibles.  
2. **Quels paramètres** on peut passer.  
3. **Quel type de données** on doit envoyer (JSON, XML, etc.).  
4. **À quoi ressemble la réponse** (codes HTTP, structure JSON…).  
5. **Comment s’authentifier** si nécessaire (ici, Bearer token JWT).  

Le grand avantage, c’est la **clarté** : tout est centralisé, documenté et normalisé.

---

### Récapitulatif Oral Rapide

1. **openapi: "3.0.0"** – La version du standard OpenAPI.  
2. **info** – Qui a fait l’API, comment la contacter, quelle licence, etc.  
3. **servers** – Les URLs où l’API est disponible (prod, dev...).  
4. **tags** – Classification par catégories (Flags, Postbox, Info, Chat, etc.).  
5. **paths** – Les chemins de l’API et ce qu’ils font (GET, POST, PUT, DELETE).  
6. **components** – Tout ce qui est réutilisable :  
   - **securitySchemes** : Type d’authentification (ici, Bearer JWT).  
   - **schemas** : Les structures de données (ex. Flag, Post, Info...).  

Les endpoints spécifient ensuite :  
- **requestBody** : Ce qu’on envoie, sous quel format.  
- **parameters** : Les filtres ou infos passées dans l’URL en `GET` ou `DELETE`.  
- **responses** : Les différentes réponses possibles (200 OK, 400 Bad Request, etc.) avec la structure de retour.