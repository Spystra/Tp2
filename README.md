
### 1. Cloner le projet

Clonez le dépôt GitLab suivant :

```bash
git clone https://gitlab.com/devops_tps/mern-app
```

Naviguez dans le répertoire du projet et familiarisez-vous avec sa structure. Vous y trouverez deux dossiers principaux :
- **client** : application React
- **server** : API Express

### 2. Créer les Dockerfiles

#### a. Pour le serveur (Express)

Créez un fichier `Dockerfile` dans le dossier `server`.

Nom de l’image : `mern-server`.

Les instructions à inclure dans le `Dockerfile` :
- Utiliser l’image de base `node:lts-alpine`.
- Définir le répertoire de travail dans le conteneur.
- Copier les fichiers `package*.json`.
- Installer les dépendances avec `npm install`.
- Copier le reste des fichiers du serveur.
- Exposer le port `9000`.
- Démarrer l’application avec `npm start`.
- Variables :
  - `MONGO_URI` : URL de connexion à MongoDB (définie dans Docker Compose).

Commande pour construire l’image du serveur :
```bash
cd server
docker build -t mern-server .
```

#### b. Pour le client (React)

Créez un fichier `Dockerfile` dans le dossier `client`.

Nom de l’image : `mern-client`.

Les instructions à inclure dans le `Dockerfile` :
- Utiliser l’image de base `node:lts-alpine`.
- Définir le répertoire de travail dans le conteneur.
- Copier les fichiers `package*.json`.
- Installer les dépendances avec `npm install`.
- Copier le reste des fichiers du client.
- Construire l’application React avec `npm run build`.
- Installer un serveur HTTP léger (par exemple, `serve`).
- Exposer le port `3000`.
- Démarrer le serveur pour servir les fichiers de build.

Commande pour construire l’image du client :
```bash
cd client
docker build -t mern-client .
```

### 3. Créer un réseau Docker

Créez un réseau pour permettre la communication entre les conteneurs :
```bash
docker network create mern-network
```

### 4. Exécuter MongoDB dans un conteneur

Lancez un conteneur MongoDB connecté au réseau créé :
```bash
docker run -d --name mongodb --network mern-network mongo
```

### 5. Exécuter les conteneurs du serveur et du client

Pour le serveur :
```bash
docker run -d --name server --network mern-network -p 9000:9000 mern-server
```

Pour le client :
```bash
docker run -d --name client --network mern-network -p 3000:3000 mern-client
```

### 6. Créer un fichier Docker Compose

Dans le répertoire racine du projet, créez un fichier `docker-compose.yml`.

#### Services à définir :
- **mongodb** : Utiliser l’image `mongo`, nommer le conteneur `mongodb`, le connecter au réseau `mern-network`.
- **server** : Construire à partir du dossier `./server`, nommer le conteneur `server`, exposer le port `9000`, le connecter au réseau `mern-network`, dépendre de `mongodb`, définir la variable d’environnement `MONGO_URI`.
- **client** : Construire à partir du dossier `./client`, nommer le conteneur `client`, exposer le port `3000`, le connecter au réseau `mern-network`, dépendre de `server`.

Définir le réseau `mern-network` avec le driver `bridge`.

#### Exemple de `docker-compose.yml` :

```yaml
version: '3.7'

services:
  mongodb:
    image: mongo
    container_name: mongodb
    networks:
      - mern-network

  server:
    build: ./server
    container_name: server
    ports:
      - "9000:9000"
    networks:
      - mern-network
    environment:
      - MONGO_URI=mongodb://mongodb:27017/mern
    depends_on:
      - mongodb

  client:
    build: ./client
    container_name: client
    ports:
      - "3000:3000"
    networks:
      - mern-network
    depends_on:
      - server

networks:
  mern-network:
    driver: bridge
```

#### Commande pour lancer l’application avec Docker Compose :

```bash
docker-compose up --build
```

---


```
