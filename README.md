# TPs DevOps 2026 – Architecture multi-niveaux et CI/CD

Ce projet met en œuvre une infrastructure complète et automatisée utilisant Docker, Ansible et GitHub Actions. L'application est composée d'un frontend Vue.js/Nginx, d'une API REST Java Spring Boot et d'une base de données PostgreSQL.

---

## 🏗 Architecture du projet

L'architecture suit un **modèle à 4 niveaux**, où le serveur HTTP fait office de point d'entrée unique (**proxy inverse**).

- **Frontend / Proxy (Nginx)** : Sert les fichiers statiques du frontend et redirige les requêtes `/api` vers le backend.

- **Backend (Spring Boot)** : API REST gérant la logique métier.

- **Base de données (PostgreSQL)** : Stockage persistant des données à l'aide d'un volume Docker.

- **Réseau (Docker Network)** : Un réseau interne (`app-network`) assure une communication isolée entre les conteneurs.

---

## 🚀 Fonctionnalités clés

### 1. Persistance des données

Un volume nommé `pgdata` est utilisé pour garantir la conservation des données de la base de données lors du redémarrage du conteneur.

**Correction appliquée :**

Ajout d'une tâche Ansible `docker_volume` pour garantir la création du volume avant le démarrage du conteneur.

---

### 2. Disponibilité (Contrôles d'intégrité)

Le conteneur de base de données intègre un contrôle d'intégrité (`pg_isready`).

Le serveur attend que la base de données soit marquée comme **saine** avant de tenter de s'y connecter, évitant ainsi les erreurs de connexion au démarrage.

---

## 3. Sécurisation avec Ansible Vault

Pour répondre aux exigences de sécurité « prêt pour la production », les données sensibles sont protégées grâce à **Ansible Vault**.

### 1. Coffre-fort de variables

Les secrets (identifiants DockerHub, clés privées, etc.) ne sont plus stockés en clair dans le dépôt Git.
Ils sont désormais **chiffrés avec l’algorithme AES256** dans le fichier :

```
ansible/credentials.yml
```

### 2. Utilisation en CI/CD

Le pipeline GitHub Actions déchiffre dynamiquement le coffre-fort lors du déploiement :

* Le mot de passe du Vault est récupéré depuis un **GitHub Secret** : 
`ANSIBLE_VAULT_PASSWORD`
* Un fichier temporaire `.vault_pass` est créé sur le coureur
* Ansible utilise ce fichier pour **déchiffrer et injecter les variables sécurisées** dans le playbook au moment du déploiement

Cela garantit que **les informations sensibles ne sont jamais exposées**, ni dans le code source, ni dans les logs.


---

### 4. Proxy inverse (Frontend)

Le serveur Nginx est configuré via `default.conf` pour gérer le routage :

- `location /` → Serve l'interface web

- `location /api/` → Transfère les requêtes vers `http://student-api:8080/`

---

## 🤖 Pipeline CI/CD (GitHub Actions)

Le workflow `.github/workflows/main.yml` automatise l'intégralité du cycle de vie :

- **Tests** : Exécution des tests unitaires Maven

- **Analyse** : Analyse de la qualité du code avec SonarCloud

- **Build & Push** : Création des images Docker pour les 4 services et leur envoi sur Docker Hub

- **Deployment** : Déploiement automatique via Ansible sur une instance distante

**Remarque concernant le déploiement manuel :**

Le déploiement peut être déclenché manuellement à l'aide du bouton **Exécuter le workflow**. Événement `workflow_dispatch`.

---

## 🛠 Installation et déploiement

### Prérequis

- Une instance Linux (Takima) accessible via SSH
- Docker et Ansible installés sur la machine de déploiement
- Clés secrètes GitHub configurées :

- `DOCKER_HUB_TOKEN` : jeton Docker Hub

- `DOCKER_HUB_USERNAME` : nom d'utilisateur Docker Hub

- `SSH_PRIVATE_KEY` : clé privée pour se connecter à l'instance Takima

- `SERVER_HOST` : nom du serveur Takima

- `SONAR_TOKEN` : jeton Sonar Cloud

- `ANSIBLE_INVENTORY`

---
### Déploiement via Ansible (local)

Pour un déploiement manuel depuis WSL :

```bash
ansible-playbook -i ansible/inventories/setup.yml ansible/playbook.yml --private-key ./id_rsa
```

### Auteur 👩‍💻

- Laura-DAMAS
- Groupe : ING2-APP-BDML1
- Projet de laboratoire DevOps – Mars-Avril 2026
- Cours : DevOps