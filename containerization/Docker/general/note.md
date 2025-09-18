# Autour de Docker

Docker est un logiciel qui peut aider à créer , à tester et à exécuter des applis facilement (en gros il regroupe tout ce dont l'appli a besoin : le code, les outils et les paramètres; le tout dans une unité standardisé qu'on va appelé "container" qu'on voit littérallement sur le logo de Docker avec la baleine elle mème qui porte beaucoup de conteneur)
Ses conteneurs sont légers et portables et ne dépendent pas du système (ou de l'ordi où ils sont exécuté ou d'autres conteneurs, ça veut dire qu'on exécuter un conteneur Docker sur n'importe quel machine qui possède Docker)

---

🐳 Métaphore du logo Docker

- La baleine = Docker
- Les conteneurs sur son dos = les applications emballées et prêtes à être transportées partout.

---

Docker est une plateforme qui permet d'empaqueter une application et toutes ses dépendances (code, bibliothèques, outils, etc.) dans une unité standardisée appelée **conteneur**.

Imaginez un conteneur comme une boîte standard pour votre application. Peu importe ce que vous mettez dedans, la boîte peut être expédiée et utilisée n'importe où, que ce soit sur un ordinateur portable,
un serveur ou dans le cloud, tant que cet endroit dispose de Docker. Cela résout le fameux problème du "ça marche sur ma machine !".

Les conteneurs sont **légers** et **isolés** les uns des autres, ce qui signifie que vous pouvez exécuter plusieurs applications sur la même machine sans qu'elles n'interfèrent entre elles.

## Comment ça marche ?

### 1. **Docker Engine** (le cœur de Docker)

C’est le moteur qui **crée et gère les conteneurs**.
Il comporte 3 parties principales :

- **Serveur / Daemon (`dockerd`)**

  - Programme qui s’exécute en arrière-plan sur l’ordinateur.
  - Il gère toutes les tâches liées à Docker : **créer, exécuter, distribuer les conteneurs**.
  - Exemple : quand tu tapes `docker run nginx`, c’est `dockerd` qui télécharge l’image **nginx** (si besoin) et lance le conteneur.

- **API Docker**

  - Un ensemble de règles qui permet à d’autres programmes de communiquer avec `dockerd`.
  - Exemple : une application externe peut demander à Docker de lancer un conteneur via l’API sans passer par la ligne de commande.

- **CLI (Command Line Interface)**

  - L’outil en ligne de commande (`docker`) qui sert à envoyer des instructions au daemon.
  - Exemple : `docker ps` → liste les conteneurs en cours d’exécution.

---

### 2. **Docker Image** (le modèle)

- Une **image** est comme un **template en lecture seule** qui contient :

  - le code de l’application,
  - les dépendances,
  - les configurations nécessaires.

- Les conteneurs sont créés **à partir d’images**.
- Exemple : l’image `python:3.11` contient déjà Python 3.11 prêt à être utilisé.

---

### 3. **Docker Hub** (la bibliothèque publique)

- C’est un **répertoire cloud** où tu peux **trouver et partager** des images Docker.
- Exemple : tu peux chercher `mysql` sur Docker Hub et exécuter directement :

  ```bash
  docker run mysql
  ```

---

### 4. **Dockerfile** (la recette)

- C’est un **script texte** qui décrit **comment construire une image**.
- Exemple simple :

  ```dockerfile
  FROM node:18
  COPY . /app
  WORKDIR /app
  RUN npm install
  CMD ["node", "server.js"]
  ```

  Ce fichier dit : prends Node.js, copie le projet, installe les dépendances, et démarre le serveur.

---

### 5. **Docker Registry** (le garde-manger)

- Système de **stockage et distribution des images**.
- Peut être **public** (ex. Docker Hub) ou **privé** (ton entreprise peut avoir son propre registre).
- Exemple : Google, AWS ou GitHub offrent leurs propres **registries** pour gérer les images.

---

# Schéma résumé (simplifié)

```
              [ Dockerfile ]
                   ↓
             docker build
                   ↓
            [ Docker Image ]
                   ↓
 docker run  ------------------>  [ Docker Container ]
                   ↑
             [ Docker Hub / Registry ]
                   ↓
              stocke & partage
```

---

              [ Dockerfile ]   --->  recette
                   ↓
             docker build
                   ↓
            [ Docker Image ]   --->  modèle
                   ↓

docker run ------------------> [ Docker Container ]
↑
[ Docker Hub / Registry ] ---> bibliothèque

              |--------------------------------------|
              |      Linux Kernel Features           |
              |  - Namespaces (isolement)            |
              |  - cgroups (limitation ressources)   |
              |--------------------------------------|

---

```
      +--------------+
      |  Dockerfile  |  (La recette)
      +--------------+
             |
             v  (Commande: docker build)
      +--------------+
      | Image Docker |  (Le moule)
      +--------------+
             |                                    +----------------------+
             | (Commande: docker push) ---------> |  Docker Hub/Registry | (La bibliothèque)
             |                                    +----------------------+
             | <--------- (Commande: docker pull)           ^
             |                                              |
             v  (Commande: docker run)
      +--------------+
      |   Conteneur  |  (L'application en cours d'exécution)
      +--------------+

```

---

+-------------------------------------------------------------+
| MACHINE HÔTE |
| |
| +-------------------------------------------------------+ |
| | SYSTÈME D'EXPLOITATION (OS) | |
| | | |
| | +-------------------+ | |
| | | Docker Engine | <-- (Gère tout) | |
| | +-------------------+ | |
| | | |
| | +---------------------------------------------------+ |
| | | NOYAU LINUX | |
| | | | |
| | | [ CGROUPS : Limite les ressources ] | |
| | | [ NAMESPACES : Isole la "vue" ] | |
| | +---------------------^-----------------------------+ |
| | | |
| +------------------------|--------------------------------+
| |  
| +-----------+-----------+  
| | |  
| +---------+---------+ +---------+---------+  
| | Conteneur A | | Conteneur B |  
| | (Isolé) | | (Isolé) |  
| +-------------------+ +-------------------+  
| |
+-------------------------------------------------------------+

---

Résumé ultra-concis :

- **Dockerfile** = recette
- **Image** = modèle prêt à l’emploi
- **Conteneur** = appli en marche
- **Hub/Registry** = bibliothèque / entrepôt
- **Engine** = le moteur qui fait tourner tout ça

---

**Fonctionnement bas niveau** de Docker : en fait, Docker n’est pas de la magie, il **s’appuie sur des fonctionnalités natives du noyau Linux** pour isoler et limiter les conteneurs.

---

# Isolation des conteneurs dans Docker

### 1. Namespaces

- **Définition** : mécanisme du noyau Linux qui **crée une vision isolée** des ressources pour chaque conteneur.
- Chaque conteneur croit qu’il a son **propre système**, mais en réalité il partage le même noyau.
- Exemples :

  - `pid namespace` → chaque conteneur a son propre espace de processus.
  - `net namespace` → chaque conteneur peut avoir sa propre interface réseau.
  - `mnt namespace` → chaque conteneur a son propre système de fichiers monté.

Grâce à ça, deux conteneurs peuvent avoir des process **PID 1** différents sans se marcher dessus.

---

### 2. Control Groups (cgroups)

- **Définition** : mécanisme qui **limite et contrôle les ressources** utilisées par un conteneur.
- Exemple :

  - limiter la mémoire (`--memory 512m`)
  - limiter le CPU (`--cpus 1`)

- Ça garantit qu’un conteneur ne peut pas monopoliser toutes les ressources de la machine.

---

### 3. Sécurité et multi-conteneurs

- **Isolation** → un conteneur n’accède pas aux process/fichiers d’un autre.
- **Stabilité** → si un conteneur crash ou sature ses ressources, il n’affecte pas les autres.
- **Scalabilité** → plusieurs conteneurs peuvent tourner **côte à côte** efficacement.

---

**Résumé enrichi :**
Docker ne réinvente pas l’isolation, il s’appuie sur le **noyau Linux** (namespaces + cgroups) pour créer des conteneurs **isolés, sécurisés et efficaces**, tout en les rendant **légers et portables**.

---

## Pourquoi utiliser Docker ?

Docker rend le développement et le déploiement **plus simples, rapides et fiables**, principalement en assurant une **cohérence** entre environnements.

### 1. **Cohérence entre environnements**

- Sans Docker : ton appli marche sur ton PC, mais pas sur celui du collègue ni sur le serveur → "ça marche chez moi mais pas chez toi".
- Avec Docker : ton appli **tourne partout de la même manière**, car elle est emballée dans un conteneur avec tout ce dont elle a besoin.
  Exemple : une API Node.js avec une version précise de Node + dépendances → dans Docker, pas de conflit de version.

### 2. **Portabilité**

- Les conteneurs tournent **sur n’importe quelle machine** (Windows, Linux, Mac, serveur cloud) tant qu’il y a Docker.
  Exemple : tu développes sur ton PC Windows, tu envoies l’image sur un serveur Linux → elle s’exécute sans changement.

### 3. **Rapidité & légèreté**

- Les conteneurs partagent le noyau de l’OS → ils se lancent en **secondes** (contrairement aux VM qui doivent démarrer un OS entier).
  Exemple : `docker run nginx` → ton serveur web est prêt en quelques secondes.

### 4. **Isolation**

- Chaque conteneur est indépendant → pas de conflits entre applis (ex. deux bases de données différentes peuvent tourner côte à côte).
  Exemple : tu peux avoir un conteneur avec **MySQL 5.7** et un autre avec **MySQL 8.0** sur la même machine.

### 5. **Scalabilité & DevOps**

- Docker s’intègre très bien dans les pipelines CI/CD et l’orchestration (Kubernetes, Docker Swarm).
  Exemple : ton appli web peut être répliquée en **10 conteneurs identiques** derrière un load balancer en quelques secondes.

# Résumé :

Docker = **"packager une application une fois et la faire tourner partout, de la même manière, rapidement et sans prise de tête"**.

---
