# Déploiement d'une instance WordPress avec MySQL en local via Docker Compose

## I. Introduction

Ce document explique comment déployer une instance WordPress avec une base de données MySQL en utilisant Docker Compose. L'objectif est d'assurer la persistance des données et une communication optimale entre les conteneurs au sein d'un réseau isolé.

---

## II. Mise en place de l’environnement

### **Prérequis**

- **Système d'exploitation** : Windows
- **Logiciels requis** :
    - Docker
    - Visual Studio Code (VSCode)
    - Navigateur Brave
    - Terminal PowerShell

### **Création du dossier de travail**

Avant de commencer, créons un dossier pour notre projet et ouvrons-le avec VSCode :

```bash
mkdir tp_wordpress && cd tp_wordpress
code .
```

---

## III. Configuration du fichier `docker-compose.yml`

Le fichier `docker-compose.yml` est conçu pour déployer deux services :

### **1. MySQL**

- Redémarre automatiquement en cas de plantage.
- Variables d’environnement pour l’initialisation :
    - `MYSQL_DATABASE` : Nom de la base de données.
    - `MYSQL_USER` / `MYSQL_PASSWORD` : Identifiants utilisateur.
    - `MYSQL_ROOT_PASSWORD` : Mot de passe root.
- Ajoute un **test de santé** pour garantir que MySQL est prêt avant que WordPress ne s’y connecte.
- Monte un **volume persistant** (`mysql-data`) pour stocker les données dans `/var/lib/mysql`.
- Se connecte au réseau `wp-network`.

### **2. WordPress**

- Démarre uniquement une fois que MySQL est **opérationnel** (`service_healthy`).
- Expose WordPress sur le port **8080** (`http://localhost:8080`).
- Configure les variables de connexion à MySQL (`WORDPRESS_DB_HOST`, `WORDPRESS_DB_USER`, `WORDPRESS_DB_PASSWORD`).
- Monte un **volume** (`wp-data`) pour conserver les fichiers WordPress (`/var/www/html`).
- Se connecte également à `wp-network`.

> **Remarque :** Docker assigne un réseau interne par défaut, mais ici nous le nommons explicitement `wp-network` pour plus de clarté.

---

## IV. Déploiement

Une fois le fichier `docker-compose.yml` finalisé, déployons les conteneurs :

```bash
docker-compose up -d
```

Cette commande :

- Crée les conteneurs.
- Télécharge les images si elles ne sont pas encore présentes.

### **1. Vérification des conteneurs**

Vérifions que tout fonctionne correctement :

```bash
docker ps
```

Cela permet de :

- Lister les conteneurs en cours d’exécution.
- Confirmer la création du réseau `wp-network`.
- Valider les volumes `wp-data` et `mysql-data`.

> Les données des volumes sont stockées sous `/var/lib/docker`.

### **2. Configuration de WordPress**

1. Accéder à l’interface d’installation :
    ```
    http://localhost:8080/wp-admin
    ```
2. Configurer le compte administrateur.
3. Accéder au tableau de bord.
4. Modifier le thème et publier un article de blog.

### **3. Vérification de la persistance des données**

1. Arrêter les conteneurs :

```bash
docker-compose down
```

2. Redémarrer les conteneurs :

```bash
docker-compose up -d
```

> **Les données sont persistantes** et ne sont pas perdues après redémarrage.

---

## V. Mise à jour d’une image Docker sans perte de données

Les volumes étant indépendants des conteneurs, une mise à jour d'image ne supprime pas les données.

1. Arrêter les conteneurs :

```bash
docker-compose down
```

2. Mettre à jour la version de l'image dans `docker-compose.yml`.

3. Télécharger l'image mise à jour :

```bash
docker-compose pull
```

4. Relancer les conteneurs :

```bash
docker-compose up -d
```

5. Vérifier que les données sont intactes.

---

## VI. Sauvegarde des données

Il est recommandé de réaliser des sauvegardes régulières pour prévenir toute perte de données.

### **1. Sauvegarde de la base de données MySQL**

```bash
docker exec my-mysql-container sh -c 'MYSQL_PWD=Password mysqldump -u root wordpress' > backup.sql
```

Cela crée un fichier `backup.sql` contenant la base de données.

### **2. Sauvegarde des fichiers WordPress**

```bash
docker cp wordpress-container:/var/www/html ./wordpress-backup
```

> Une fois la mise à jour effectuée, on pourra restaurer les données sauvegardées.
