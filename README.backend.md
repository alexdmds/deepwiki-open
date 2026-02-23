# DeepWiki Backend - Guide d'utilisation

Ce document explique comment lancer uniquement le backend de DeepWiki avec Docker.

## 🚀 Démarrage rapide

### Option 1 : Docker Build & Run

```bash
# Construire l'image
docker build -f Dockerfile.backend -t deepwiki-backend .

# Lancer sur le port par défaut (8001)
docker run -p 8001:8001 --env-file .env deepwiki-backend

# Lancer sur un port personnalisé (exemple: 9000)
docker run -p 9000:9000 -e PORT=9000 --env-file .env deepwiki-backend
```

### Option 2 : Docker Compose

```bash
# Lancer avec le port par défaut (8001)
docker-compose -f docker-compose.backend.yml up

# Lancer sur un port personnalisé
PORT=9000 docker-compose -f docker-compose.backend.yml up

# Lancer en arrière-plan
docker-compose -f docker-compose.backend.yml up -d

# Arrêter
docker-compose -f docker-compose.backend.yml down
```

## ⚙️ Configuration

### Variables d'environnement requises

Créez un fichier `.env` à la racine du projet avec les clés API nécessaires :

```env
OPENAI_API_KEY=your_openai_api_key_here
GOOGLE_API_KEY=your_google_api_key_here
PORT=8001
```

### Changer le port

Vous pouvez changer le port de trois façons :

1. **Via la variable d'environnement PORT dans .env** :
   ```env
   PORT=9000
   ```

2. **Via la ligne de commande Docker** :
   ```bash
   docker run -p 9000:9000 -e PORT=9000 --env-file .env deepwiki-backend
   ```

3. **Via Docker Compose** :
   ```bash
   PORT=9000 docker-compose -f docker-compose.backend.yml up
   ```

## 🔍 Vérification

Une fois lancé, vous pouvez vérifier que le backend fonctionne :

```bash
# Vérifier la santé du serveur
curl http://localhost:8001/health

# Ou sur un port personnalisé
curl http://localhost:9000/health
```

## 📋 Commandes utiles

```bash
# Voir les logs
docker logs deepwiki-backend

# Suivre les logs en temps réel
docker logs -f deepwiki-backend

# Arrêter le conteneur
docker stop deepwiki-backend

# Supprimer le conteneur
docker rm deepwiki-backend

# Supprimer l'image
docker rmi deepwiki-backend
```

## 🔧 Mode développement

Pour monter le code source en temps réel (utile pour le développement) :

```bash
docker run -p 8001:8001 \
  -v $(pwd)/api:/app/api \
  --env-file .env \
  deepwiki-backend
```

## 📦 Taille de l'image

L'image backend est optimisée et significativement plus légère que l'image complète :
- Image complète (frontend + backend) : ~1.5 GB
- Image backend seule : ~600 MB

## 🌐 API Endpoints

Une fois démarré, le backend expose les endpoints suivants :
- WebSocket : `ws://localhost:8001/ws/wiki`
- Health check : `http://localhost:8001/health`
- API docs : `http://localhost:8001/docs`

## ⚠️ Notes importantes

1. **Clés API** : Les variables `OPENAI_API_KEY` et `GOOGLE_API_KEY` sont requises pour le fonctionnement du backend.
2. **Port** : Assurez-vous que le port choisi n'est pas déjà utilisé par un autre service.
3. **Firewall** : Si vous lancez le backend sur un serveur, assurez-vous que le port est ouvert dans votre firewall.
