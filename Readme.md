## 1. En-tête OpenAPI

```yaml
openapi: 3.0.0
```
- **openapi** : Spécifie la version du format OpenAPI utilisée.  
  - Ici, la valeur `3.0.0` indique que nous respectons les règles et la structure de la spécification OpenAPI 3.0.  
  - Cela permet aux outils comme **Swagger UI**, **Redoc** ou **Postman** de lire et d’interpréter correctement la documentation.

---

## 2. Section `info`

```yaml
info:
  title: API Wager
  description: >
    API pour gérer les flags ...
  version: 1.0.0
  contact:
    name: Support de l'API
    email: support@websdr.com
  license:
    name: Apache 2.0
    url: https://www.apache.org/licenses/LICENSE-2.0.html
```

1. **title** : Le nom officiel de l’API (ici, "API Wager").  
2. **description** : Un texte plus détaillé décrivant l’objectif de l’API.  
3. **version** : La version de l’API (ici "1.0.0").  
   - Différent de la version OpenAPI ; c’est vous qui gérez la version de **votre** API.
4. **contact** :  
   - **name** : L’entité ou la personne à contacter pour du support.  
   - **email** : L’adresse e-mail de support.  
5. **license** :  
   - **name** : Le type de licence (ici, "Apache 2.0").  
   - **url** : Un lien vers le texte officiel de la licence.  

Cette section permet à toute personne (développeur, partenaire, etc.) de savoir qui a créé l’API, comment la contacter et sous quelle licence elle est publiée.

---

## 3. Section `servers`

```yaml
servers:
  - url: https://api.WebSDR.com/v1
    description: Serveur de production
  - url: http://localhost:3000
    description: Serveur de développement local
```

- **servers** : Liste des URLs où l’API peut être appelée.  
  - **url** : L’adresse de base de l’API.  
  - **description** : Un court texte explicatif (ex. "production" vs "développement local").  

Lorsqu’un outil comme Swagger UI lit ce document, il propose ces URLs pour effectuer des tests.  
- Le premier est l’environnement **officiel** (production).  
- Le second est pour **tester localement** en développement.

---

## 4. Section `tags`

```yaml
tags:
  - name: FLAGS
    description: Gérer les flags ...
  - name: POSTBOX
    description: ...
  - name: INFO
    description: ...
  - name: CHAT
    description: ...
  - name: LIST
    description: ...
```

- **tags** : Permet de **catégoriser** les endpoints (les chemins) afin de regrouper la documentation par thématique.  
  - **name** : Nom de la catégorie.  
  - **description** : Brève explication de ce que couvre cette catégorie.  

Ainsi, dans la documentation générée, on verra des sections distinctes pour chaque tag (par exemple, **FLAGS** pour toutes les routes qui concernent les drapeaux/frequences).

---

## 5. Section `paths`

**`paths`** est le cœur de la documentation : ici sont décrits tous les chemins (URL) et leurs différentes opérations HTTP (GET, POST, PUT, DELETE, …).

### 5.1. `/flags`

```yaml
/flags:
  get:
    tags:
      - LIST
    summary: Récupérer tous les flags
    description: >
      Cet endpoint permet de récupérer une liste de tous les flags disponibles...
    operationId: getAllFlags
    parameters:
      - in: query
        name: freqMin
        ...
      - in: query
        name: freqMax
        ...
      ...
    responses:
      "200":
        ...
      "400":
        ...
```

#### 5.1.1. Méthode `GET /flags`
- **tags** : `LIST`  
  - Associe cette opération à la catégorie "LIST" pour la documentation.  
- **summary** : Phrase courte expliquant l’objectif principal.  
- **description** : Plus de détails sur ce que fait concrètement l’endpoint (récupérer tous les flags).  
- **operationId** : Identifiant unique pour cette opération (utile pour la génération de code ou la référence interne).

##### 5.1.1.1. `parameters`
Cette liste décrit tous les **paramètres de requête** (`?param=value` dans l’URL).

1. **freqMin**  
   - **in: query** : Signifie qu’il s’agit d’un paramètre dans l’URL (ex. `GET /flags?freqMin=190`).  
   - **name: freqMin** : Nom du paramètre.  
   - **schema** : Définit le type (number), le format (float) et donne un exemple (190.00).  
   - **description** : Explique qu’on filtre par fréquence minimale en MHz.  

2. **freqMax**  
   - Même logique que `freqMin`, mais pour la fréquence maximale.  

3. **page**  
   - **type: integer** : Indique qu’on attend un entier.  
   - **default: 1** : Si rien n’est spécifié, on prend la page 1 par défaut.  
   - **minimum: 1** : On ne peut pas demander la page 0 ou une page négative.  
   - **description** : C’est pour la pagination, c’est-à-dire afficher les résultats page par page.  

4. **limit**  
   - Même logique que `page`, mais spécifie le nombre d’éléments par page.  
   - **default: 10** : On affiche par défaut 10 flags par page.  

5. **sort**  
   - **type: string**  
   - **enum: [freq, region, nom]** : On limite les valeurs possibles à ces 3 options.  
   - **example: freq** : Exemple pratique.  
   - **description** : Permet de préciser sur quel champ on veut trier la liste de flags.

##### 5.1.1.2. `responses`
- **"200"** : Code HTTP 200 signifie **succès**.  
  - **description** : "Liste des drapeaux".  
  - **content** : On retourne du **JSON**.  
  - **schema** : `type: array` (c’est un tableau). Chaque élément du tableau est défini par `items: $ref: '#/components/schemas/Flag'`, c’est-à-dire qu’il suit la structure décrite plus bas dans **components** → **schemas** → **Flag**.  
  - **example** : Montre à quoi ressemble la réponse (deux flags, "NRJ" et "RockFM").  

- **"400"** : Code HTTP 400 signifie **mauvaise requête** si, par exemple, `freqMin` est invalide.  
  - On référence le schéma d’erreur (`$ref: '#/components/schemas/Error'`).

---

### 5.2. `PUT /flags`

```yaml
    put:
      tags:
        - FLAGS
      summary: Créer ou mettre à jour un drapeau
      ...
      requestBody:
        required: true
        content:
          application/json:
            schema:
              $ref: '#/components/schemas/Flag'
            example:
              nom: "NRJ"
              freq: 190.00
              region: "Bordeaux, FRANCE"
      responses:
        ...
```

#### 5.2.1. `tags`
- **FLAGS** : Indique qu’on manipule ici la ressource “Flags”.

#### 5.2.2. `summary` et `description`
- Explique que cette opération crée ou met à jour un "flag" (une station radio marquée).

#### 5.2.3. `security`
```yaml
security:
  - BearerAuth: []
```
- Indique que pour appeler ce **PUT**, on doit fournir une **authentification Bearer** (ex. un token JWT) dans l’en-tête de la requête HTTP.

#### 5.2.4. `requestBody`
- **required: true** : Un corps de requête est obligatoire.  
- **content: application/json** : On envoie du JSON.  
- **schema** : On fait référence au schéma `Flag` pour la structure des données attendues.  
- **example** : Montre un exemple de JSON valide.

#### 5.2.5. `responses`
- **"200"** : Le drapeau est créé ou mis à jour.  
- **"400"** : Données invalides.  
- **"401"** : Problème d’authentification (manque de token ou token invalide).  
- **"500"** : Erreur interne.

---

### 5.3. `/postbox`

```yaml
/postbox:
  get:
    tags:
      - POSTBOX
    summary: Récupérer tous les messages
    ...
    parameters:
      - in: query
        name: user
        ...
      - in: query
        name: date
        ...
    responses:
      "200":
        ...
  post:
    tags:
      - POSTBOX
    summary: Créer un nouveau message
    ...
    security:
      - BearerAuth: []
    requestBody:
      required: true
      content:
        application/json:
          schema:
            $ref: '#/components/schemas/Post'
          example: ...
    responses:
      "201":
        ...
```

#### 5.3.1. `GET /postbox`
- **tags: [POSTBOX]** : Catégorie “POSTBOX”.  
- **summary** : Récupérer tous les messages (de type forum).  
- **parameters** :  
  - `user` (string) : Filtrer par nom d’utilisateur.  
  - `date` (date) : Filtrer par date précise.  
- **responses** :  
  - **"200"** : Renvoie un tableau de posts (voir schéma **Post**).  

#### 5.3.2. `POST /postbox`
- **summary** : Créer un **nouveau** message.  
- **security** : Requiert un token Bearer.  
- **requestBody** : Se réfère au schéma **Post** (titre, contenu, date, etc.).  
- **responses** :  
  - **"201"** : Créé avec succès (typique pour un POST).  
  - **"400"**, **"401"**, **"500"** : Différents cas d’erreurs.

---

### 5.4. `/info`

```yaml
/info:
  get:
    tags:
      - INFO
    summary: Récupérer les informations utilisateur
    ...
    parameters:
      - in: query
        name: user
        ...
  delete:
    tags:
      - INFO
    summary: Supprimer les informations utilisateur
    ...
    parameters:
      - in: query
        name: user
        ...
    security:
      - BearerAuth: []
    responses:
      "204":
        ...
```

#### 5.4.1. `GET /info`
- Permet de **lire** les infos d’un utilisateur (nom, date de création, historique d’écoute).  
- Paramètre **`user`** : préciser l’utilisateur concerné.  
- **"200"** : Retourne un objet de type **Info**.  
- **"404"** : L’utilisateur n’existe pas.  

#### 5.4.2. `DELETE /info`
- Permet de **supprimer** les infos de cet utilisateur.  
- Paramètre **`user`** obligatoire.  
- Protégé par **BearerAuth**.  
- **"204"** : Succès (No Content).  
- **"404"** : L’utilisateur n’existe pas.  
- **"401"** / **"500"** : Autres cas d’erreurs.

---

### 5.5. `/chat`

```yaml
/chat:
  get:
    tags:
      - CHAT
    summary: Récupérer les messages de chat
    ...
    parameters:
      - in: query
        name: frequency
        ...
      - in: query
        name: user
        ...
  put:
    tags:
      - CHAT
    summary: Envoyer un nouveau message de chat
    ...
    security:
      - BearerAuth: []
    requestBody:
      required: true
      content:
        application/json:
          schema:
            $ref: '#/components/schemas/Chat'
          example: ...
```

#### 5.5.1. `GET /chat`
- Récupère tous (ou une partie) des messages de chat.  
- Paramètres de filtrage :  
  - **frequency** (float) : Pour ne récupérer que les messages relatifs à une fréquence donnée.  
  - **user** (string) : Pour filtrer par auteur du message.  

#### 5.5.2. `PUT /chat`
- Permet d’**envoyer** (ou de mettre à jour) un message de chat.  
- Protégé par **BearerAuth**.  
- Les données attendues (dans **requestBody**) suivent le schéma **Chat**.  
- **"201"** : Créé avec succès.

---

## 6. Section `components`

```yaml
components:
  securitySchemes:
    BearerAuth:
      type: http
      scheme: bearer
      bearerFormat: JWT
```
- **securitySchemes** : Définit les différentes manières de s’authentifier à l’API.  
  - **BearerAuth** : Authentification HTTP de type “bearer token” (souvent un **JWT**).  
  - Quand un endpoint a `security: - BearerAuth: []`, cela veut dire qu’on doit envoyer un en-tête `Authorization: Bearer <token>` pour y accéder.

### 6.1. `schemas`
Cette sous-section décrit les **objets** (structures de données) réutilisés dans tout le document via `$ref: '#/components/schemas/...'`.

#### 6.1.1. `Flag`
```yaml
Flag:
  type: object
  required:
    - nom
    - freq
    - region
  properties:
    nom:
      type: string
      description: Nom unique ...
    freq:
      type: number
      format: float
      example: 190.00
    region:
      type: string
      ...
```
- **type: object** : Le drapeau est un objet JSON.  
- **required** : Les champs obligatoires pour qu’un Flag soit valide. Ici, `nom`, `freq` et `region`.  
- **properties** : Les **champs** de l’objet.  
  - `nom` (string) : Identifiant unique du flag.  
  - `freq` (number, float) : La fréquence en MHz.  
  - `region` (string) : Région géographique (ex. "Bordeaux, FRANCE").

#### 6.1.2. `Post`
```yaml
Post:
  type: object
  required:
    - nom_user
    - date_post
    - titre
    - contenu
  ...
```
- Utilisé pour décrire un **message** dans le forum (Postbox).  
- **nom_user** : Le nom de l’utilisateur qui a posté.  
- **date_post** : La date du post.  
- **titre** : Le titre du post.  
- **contenu** : Le texte du post.

#### 6.1.3. `Chat`
```yaml
Chat:
  type: object
  required:
    - nom_user
    - date
    - contenu
  ...
```
- Objet décrivant un **message de chat**.  
  - `nom_user` (string) : L’auteur du message.  
  - `date` (date-time) : Timestamp du message, format ISO 8601 (`YYYY-MM-DDTHH:mm:ssZ`).  
  - `contenu` (string) : Le texte du message.

#### 6.1.4. `Info`
```yaml
Info:
  type: object
  required:
    - nom_user
    - date_creation
  properties:
    nom_user:
      ...
    date_creation:
      ...
    historique:
      type: array
      items:
        type: object
        properties:
          flag:
            ...
          date:
            ...
```
- Décrit les informations liées à un utilisateur :  
  - **nom_user** : Le nom de l’utilisateur.  
  - **date_creation** : Date de création de son compte.  
  - **historique** : Liste d’éléments qui représentent chaque station (flag) écoutée et la date d’écoute.

#### 6.1.5. `Error`
```yaml
Error:
  type: object
  properties:
    code:
      type: integer
      example: 400
    message:
      type: string
      example: "Paramètres de requête invalides"
```
- Décrit la structure d’un objet **erreur** que l’API retourne (ex. code 400, message explicatif).
