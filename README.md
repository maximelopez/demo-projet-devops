# Projet DevOps Task Manager

Ce projet contient l'application Task Manager (Frontend, Tasks Service et Stats Service) mise en conteneurs avec Docker et déployée sur Kubernetes.

## Pipeline Complet
Toutes les commandes sont regroupées dans le `Taskfile.yml`.
Pour lancer tout le projet d'un coup, il suffit de taper : `task pipeline`

Cette commande fait tout dans l'ordre :
- Lancer la base Postgres et le registry local.
- Lint : Vérifier le code avec ESLint.
- Test : Lancer les tests avec Vitest.
- Build : Construire les images Docker.
- Tag et push : Envoyer les images compilées sur le registry local.
- Deploy : Lancer le déploiement sur Kubernetes.

### Dockerfiles multi-stage
J'ai créé un Dockerfile pour chaque partie (frontend, tasks-service, stats-service).
J'utilise le multi-stage build : ça compile le code d'abord, puis ça garde juste le résultat final pour que l'image soit très légère.
J'ai aussi mis un fichier `.dockerignore` dans chaque projet pour ne pas copier les `node_modules` et gagner du temps au build.

### Tests et qualité
Pour le code, j'ai installé ESLint sur les trois projets. On vérifie tout avec `task lint`.
Pour les tests unitaires, j'utilise Vitest qu'on lance avec `task test`.

### Docker Compose Dev
Le fichier `docker-compose.yml` permet de lancer le projet en local pour développer (`docker compose up --build`).
J'ai ajouté les arguments de build (`args`) pour utiliser les ports locaux 3001 et 3002. Comme ça, le frontend communique directement avec l'API locale. Le fichier lance aussi la base de données et le registry.

### Build des images
Pour les images destinées à Kubernetes, j'utilise de base la commande `docker build` (via `task build`).
Ça permet de donner les vrais ports Kubernetes (30080 et 30081) au frontend avec des `--build-arg`. L'image envoyée sur le registre est donc prête pour K8s sans devoir être modifiée.

### Registry local
J'ai ajouté le registry local dans le docker-compose.
La commande `task push` envoie directement les images dessus à la fin du build.

### Déploiement K8s
Pour déployer sur Kubernetes, j'utilise kubectl avec des fichiers YAML simples.
J'ai rangé les configurations (Deployments, Services, ConfigMaps, Secrets) par service dans le dossier `/k8s/`.

### Pipeline Taskfile
Le fichier `Taskfile.yml` sert de chef d'orchestre. Avec une seule commande `task pipeline`, on n'a plus rien à faire à la main. Le script allume la base, fait les vérifications (lint, test), compile les fichiers, envoie sur le registre et déploie sur K8s tout seul.
