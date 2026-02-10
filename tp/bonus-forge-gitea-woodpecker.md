# TP Bonus : Créer sa forge logicielle avec Gitea + Woodpecker CI

> Durée : 3h30 (un après-midi)

## Objectifs

- Comprendre ce qu'est une forge logicielle et pourquoi l'auto-héberger
- Déployer Gitea (forge Git auto-hébergée) avec Docker Compose
- Configurer Woodpecker CI (moteur CI/CD léger) avec Gitea
- Créer un pipeline CI complet (.woodpecker.yml) avec lint, test et build
- Comparer cette approche auto-hébergée avec GitHub + GitHub Actions

---

## Partie 1 : Qu'est-ce qu'une forge logicielle ? (15 min)

### 1.1 Définition et composants

Une **forge logicielle** (software forge) est une plateforme qui combine plusieurs outils essentiels au développement :

- **Hébergement Git** : stockage et gestion du code source
- **CI/CD** : automatisation des tests, builds et déploiements
- **Issue tracking** : gestion des bugs et fonctionnalités
- **Code review** : revue de code et pull/merge requests
- **Documentation** : wiki, pages, releases

**Exemples de forges :**

| Forge | Type | Caractéristiques |
|-------|------|------------------|
| **GitHub** | SaaS | Leader, GitHub Actions intégré, gratuit pour public |
| **GitLab** | SaaS/Self-hosted | CI/CD intégré, très complet, lourd en ressources |
| **Gitea + Woodpecker** | Self-hosted | Léger, simple, open source, souveraineté totale |
| **Forgejo** | Self-hosted | Fork communautaire de Gitea, gouvernance libre |

### 1.2 Pourquoi auto-héberger une forge ?

**Avantages du self-hosting :**

- **Souveraineté des données** : vos données restent chez vous (RGPD, secrets)
- **Coût** : gratuit au-delà du serveur (pas de limite de minutes CI)
- **Personnalisation** : configuration complète selon vos besoins
- **Apprentissage** : comprendre le fonctionnement interne d'une forge
- **Indépendance** : pas de dépendance aux plateformes externes

**Inconvénients :**

- **Maintenance** : mises à jour, backups, sécurité à gérer
- **Infrastructure** : besoin d'un serveur ou VPS
- **Fonctionnalités** : moins de features que GitHub/GitLab
- **Écosystème** : moins de plugins et intégrations tierces

### 1.3 Gitea : la forge légère

**Gitea** est une forge Git open source écrite en Go :

- **Binaire unique** : facile à déployer, pas de dépendances complexes
- **Ressources minimales** : fonctionne avec 512 MB RAM (vs plusieurs GB pour GitLab)
- **Interface** : similaire à GitHub, facile à prendre en main
- **Base de données** : SQLite (simple) ou PostgreSQL/MySQL (production)
- **Licence** : MIT, complètement libre

### 1.4 Woodpecker CI : le moteur CI léger

**Woodpecker CI** est un fork communautaire de Drone CI :

- **Configuration YAML** : fichier `.woodpecker.yml` simple et lisible
- **Docker-native** : chaque step s'exécute dans un conteneur
- **Architecture** : serveur + agents (scalable)
- **Intégrations** : Gitea, GitHub, GitLab, Bitbucket
- **Plugins** : écosystème de plugins Docker pour déploiement, notifications, etc.

---

## Partie 2 : Installation de Gitea (45 min)

### 2.1 Préparation de l'environnement

**Créez un dossier de travail :**

```bash
mkdir ~/forge-lab
cd ~/forge-lab
```

### 2.2 Configuration Docker Compose pour Gitea

**Créez un fichier `docker-compose.yml` :**

```yaml
networks:
  ci-network:

services:
  gitea:
    image: gitea/gitea:1.24
    container_name: forge-gitea
    environment:
      - USER_UID=1000
      - USER_GID=1000
    restart: unless-stopped
    networks:
      - ci-network
    volumes:
      - ./data/gitea:/data
    ports:
      - "3000:3000"
      - "2222:22"
```

**Démarrez Gitea :**

```bash
docker compose up -d gitea
```

**Vérifiez les logs :**

```bash
docker compose logs -f gitea
```

Attendez le message indiquant que Gitea est prêt (~10-15 secondes).

### 2.3 Configuration initiale de Gitea

**Accédez à Gitea :** http://localhost:3000

Lors du premier accès, vous verrez l'écran de configuration initiale :

1. **Base de données** :
   - Type : `SQLite3` (par défaut, parfait pour le lab)
   - Chemin : `/data/gitea.db` (déjà configuré)

2. **Paramètres généraux** :
   - Titre du site : `Forge Lab`
   - URL de base : `http://localhost:3000`

3. **Paramètres administrateur** :
   - Nom d'utilisateur : `admin`
   - Mot de passe : `admin123` (lab uniquement !)
   - Adresse email : `admin@localhost`

4. Cliquez **"Installer Gitea"**

### 2.4 Exploration de l'interface Gitea

Une fois connecté, explorez l'interface :

1. Créez un nouveau repository :
   - Nom : `test-repo`
   - Description : `Repository de test`
   - Visibilité : Public
   - Cliquez **"Créer le dépôt"**

2. Clonez-le localement :

```bash
cd ~/forge-lab
git clone http://localhost:3000/admin/test-repo.git
cd test-repo
echo "# Test Repo" > README.md
git add README.md
git commit -m "Initial commit"
git push origin main
```

### 2.5 Import du projet TaskFlow

Au lieu de migrer votre projet principal, nous allons **mirror** le projet TaskFlow dans Gitea :

1. Dans Gitea, cliquez **"+"** → **"Nouvelle migration"**
2. Choisissez **"Git"** comme source
3. Configuration :
   - URL de clonage : `https://github.com/Foreach-Academy-France/taskflow-starter.git`
   - Nom du dépôt : `taskflow`
   - Visibilité : Public
   - **Ne cochez PAS** "Ce dépôt sera un miroir" (sinon vous ne pourrez pas push)
4. Cliquez **"Migrer le dépôt"**

Attendez quelques secondes, le projet est maintenant dans votre Gitea !

### 2.6 Configuration des webhooks (CRITIQUE)

Pour que Woodpecker CI puisse recevoir les événements de Gitea, il faut autoriser les webhooks vers localhost.

**Modifiez la configuration de Gitea :**

```bash
docker compose exec gitea sh
```

Dans le conteneur :

```sh
cat >> /data/gitea/conf/app.ini << 'EOF'

[webhook]
ALLOWED_HOST_LIST = external,loopback,private
EOF

exit
```

**Redémarrez Gitea :**

```bash
docker compose restart gitea
```

Attendez quelques secondes que Gitea redémarre.

---

## Partie 3 : Installation de Woodpecker CI (45 min)

### 3.1 Génération du secret agent

Woodpecker utilise un secret partagé entre le serveur et les agents pour sécuriser la communication.

**Générez un secret aléatoire :**

```bash
openssl rand -hex 32
```

Copiez la valeur générée (vous en aurez besoin dans le `.env`).

### 3.2 Création du fichier .env

**Créez un fichier `.env` à la racine de `~/forge-lab` :**

```bash
# Secret partagé entre server et agent
AGENT_SECRET=VOTRE_SECRET_GENERE_ICI

# OAuth Gitea (à remplir après création de l'application OAuth)
GITEA_CLIENT_ID=
GITEA_CLIENT_SECRET=
```

Remplacez `VOTRE_SECRET_GENERE_ICI` par la valeur générée à l'étape précédente.

### 3.3 Création de l'application OAuth dans Gitea

Woodpecker a besoin d'une application OAuth pour se connecter à Gitea et gérer les repositories.

1. Connectez-vous à Gitea (http://localhost:3000)
2. Cliquez sur votre avatar → **"Paramètres"**
3. Dans le menu de gauche → **"Applications"**
4. Section **"Gérer les applications OAuth2"** → **"Créer une nouvelle application OAuth2"**

**Configuration de l'application :**

- Nom de l'application : `Woodpecker CI`
- URI de redirection : `http://localhost:8000/authorize`
- Cochez **Confidential Client** : Oui

5. Cliquez **"Créer une application"**

**Copiez les credentials :**

Vous verrez maintenant :
- **Client ID** : une longue chaîne (ex: `abc123...`)
- **Client Secret** : une autre longue chaîne (visible une seule fois !)

**Mettez à jour le fichier `.env` :**

```bash
AGENT_SECRET=votre_secret_agent
GITEA_CLIENT_ID=le_client_id_copié
GITEA_CLIENT_SECRET=le_client_secret_copié
```

### 3.4 Ajout de Woodpecker au docker-compose.yml

**Éditez le fichier `docker-compose.yml` et ajoutez les services Woodpecker :**

```yaml
networks:
  ci-network:

services:
  gitea:
    image: gitea/gitea:1.24
    container_name: forge-gitea
    environment:
      - USER_UID=1000
      - USER_GID=1000
    restart: unless-stopped
    networks:
      - ci-network
    volumes:
      - ./data/gitea:/data
    ports:
      - "3000:3000"
      - "2222:22"

  woodpecker-server:
    image: woodpeckerci/woodpecker-server:v3
    container_name: forge-woodpecker-server
    restart: unless-stopped
    ports:
      - "8000:8000"
    networks:
      - ci-network
    volumes:
      - ./data/woodpecker-server:/var/lib/woodpecker/
    environment:
      - WOODPECKER_OPEN=true
      - WOODPECKER_HOST=http://localhost:8000
      - WOODPECKER_GITEA=true
      - WOODPECKER_GITEA_URL=http://gitea:3000
      - WOODPECKER_GITEA_CLIENT=${GITEA_CLIENT_ID}
      - WOODPECKER_GITEA_SECRET=${GITEA_CLIENT_SECRET}
      - WOODPECKER_AGENT_SECRET=${AGENT_SECRET}
    depends_on:
      - gitea

  woodpecker-agent:
    image: woodpeckerci/woodpecker-agent:v3
    container_name: forge-woodpecker-agent
    command: agent
    restart: unless-stopped
    depends_on:
      - woodpecker-server
    networks:
      - ci-network
    volumes:
      - /var/run/docker.sock:/var/run/docker.sock
    environment:
      - WOODPECKER_SERVER=woodpecker-server:9000
      - WOODPECKER_AGENT_SECRET=${AGENT_SECRET}
      - WOODPECKER_BACKEND=docker
      - WOODPECKER_BACKEND_DOCKER_NETWORK=forge-lab_ci-network
      - WOODPECKER_MAX_WORKFLOWS=2
```

**Points importants :**

- `WOODPECKER_GITEA_URL=http://gitea:3000` : utilise le nom du service Docker (pas localhost !)
- `WOODPECKER_HOST=http://localhost:8000` : URL accessible depuis le navigateur
- `WOODPECKER_BACKEND_DOCKER_NETWORK=forge-lab_ci-network` : le réseau Docker (préfixé par le nom du dossier)
- `WOODPECKER_OPEN=true` : permet l'accès sans authentification (lab uniquement)
- Agent monte `/var/run/docker.sock` : permet de lancer des conteneurs Docker

### 3.5 Démarrage de Woodpecker

```bash
docker compose up -d
```

**Vérifiez les logs :**

```bash
docker compose logs -f woodpecker-server woodpecker-agent
```

Vous devriez voir :
- Server : `starting Woodpecker server`
- Agent : `successfully connected to server`

### 3.6 Connexion à Woodpecker

**Accédez à Woodpecker :** http://localhost:8000

1. Cliquez **"Login"**
2. Vous serez redirigé vers Gitea
3. Autorisez l'application Woodpecker
4. Vous revenez sur Woodpecker, connecté !

### 3.7 Activation du repository TaskFlow

Dans Woodpecker :

1. Vous voyez la liste de vos repositories Gitea
2. Trouvez **"taskflow"**
3. Cliquez sur le bouton **"Enable"** (activer)
4. Le repository est maintenant surveillé par Woodpecker

---

## Partie 4 : Premier pipeline Woodpecker (45 min)

### 4.1 Comprendre la syntaxe Woodpecker

Un fichier `.woodpecker.yml` contient :

- **steps** : liste de steps (équivalent des jobs GitHub Actions)
- **image** : l'image Docker à utiliser pour le step
- **commands** : les commandes shell à exécuter
- **depends_on** : dépendances entre steps (parallélisme ou séquentiel)
- **when** : conditions d'exécution (branche, événement, etc.)

### 4.2 Cloner le repository TaskFlow en local

```bash
cd ~/forge-lab
git clone http://localhost:3000/admin/taskflow.git
cd taskflow
```

### 4.3 Créer le fichier .woodpecker.yml

**Créez `.woodpecker.yml` à la racine du projet :**

```yaml
steps:
  - name: install
    image: node:20-alpine
    commands:
      - npm ci

  - name: lint
    image: node:20-alpine
    commands:
      - npm run lint
    depends_on:
      - install

  - name: test
    image: node:20-alpine
    commands:
      - npm test
    depends_on:
      - install

  - name: build
    image: node:20-alpine
    commands:
      - npm run build
    depends_on:
      - lint
      - test
    when:
      - branch: main
        event: push
```

**Explication du pipeline :**

- **install** : s'exécute en premier, installe les dépendances
- **lint** et **test** : s'exécutent en parallèle après `install`
- **build** : s'exécute uniquement si `lint` et `test` réussissent
- **when** : `build` ne s'exécute que sur la branche `main` lors d'un `push`

### 4.4 Commit et push du pipeline

```bash
git add .woodpecker.yml
git commit -m "ci: add Woodpecker CI pipeline"
git push origin main
```

### 4.5 Observer l'exécution du pipeline

1. Retournez sur Woodpecker (http://localhost:8000)
2. Cliquez sur **"taskflow"** dans la liste des repos
3. Vous verrez le build **#1** en cours d'exécution
4. Cliquez dessus pour voir les steps en temps réel
5. Observez l'exécution de chaque step avec les logs

**Analyse de l'exécution :**

- Les steps `lint` et `test` s'exécutent **en parallèle** (gain de temps)
- Chaque step utilise une image Docker fraîche (`node:20-alpine`)
- Le workspace (`/woodpecker/src`) est partagé entre les steps
- Le build devrait passer en **vert** (succès)

### 4.6 Provoquer un échec volontaire

Testons comment Woodpecker gère les échecs.

**Modifiez `src/app.js` pour introduire une erreur de lint :**

```javascript
// Ajoutez cette ligne en haut du fichier après les imports
const unusedVariable = "test"; // Volontairement non utilisée
```

**Commit et push :**

```bash
git add src/app.js
git commit -m "test: add lint error"
git push origin main
```

**Observez dans Woodpecker :**

1. Le build **#2** démarre
2. Le step `install` passe
3. Le step `lint` **échoue** (erreur ESLint : variable non utilisée)
4. Le step `build` ne s'exécute **pas** (dépendance bloquée)
5. Le build est marqué en **rouge** (échec)

### 4.7 Corriger et valider

**Supprimez la ligne problématique dans `src/app.js` :**

```bash
git revert HEAD
git push origin main
```

Le build **#3** devrait passer en vert.

### 4.8 Aller plus loin : améliorer le pipeline

**Ajoutez un step d'information :**

Pour afficher des informations de debug, ajoutez un step au début :

```yaml
steps:
  - name: info
    image: node:20-alpine
    commands:
      - echo "Node version $(node --version)"
      - echo "NPM version $(npm --version)"
      - echo "Branch $CI_COMMIT_BRANCH"
      - echo "Commit $CI_COMMIT_SHA"

  - name: install
    image: node:20-alpine
    commands:
      - npm ci
    depends_on:
      - info
  # ... reste du pipeline
```

**Variables CI disponibles :** Woodpecker injecte automatiquement des variables d'environnement `CI_*` dans chaque step (commit, branche, auteur, etc.).

**Note sur le cache :** En production, utilisez le plugin `woodpeckerci/plugin-s3-cache` pour cacher `node_modules` entre les builds et accélérer les pipelines.

---

## Partie 5 : Comparaison et réflexion (30 min)

### 5.1 Tableau comparatif des forges

Complétez ce tableau avec vos observations :

| Critère | GitHub + GitHub Actions | Gitea + Woodpecker |
|---------|-------------------------|---------------------|
| Installation | Aucune (SaaS) | Docker Compose (~5 min) |
| Configuration CI | Fichier YAML dans `.github/workflows/` | Fichier YAML `.woodpecker.yml` |
| Syntaxe pipeline | `jobs:`, `steps:`, `uses:`, `runs-on:` | `steps:`, `image:`, `commands:`, `depends_on:` |
| Interface | Web moderne, très complète | Web simple, fonctionnelle |
| Coût | Gratuit (public) ou $4/mois (privé) + minutes CI | Gratuit (coût serveur uniquement) |
| Maintenance | Aucune (géré par GitHub) | Vous gérez (updates, backups) |
| Plugins/Extensions | Marketplace énorme (10000+) | Plugins Docker (écosystème plus petit) |
| Souveraineté des données | Données chez Microsoft | Données chez vous |
| Scalabilité | Illimitée (cloud) | Limitée par vos ressources |
| Temps d'exécution | Variable (runners partagés) | Dépend de votre machine |

### 5.2 Correspondances syntaxiques

| Concept | GitHub Actions | Woodpecker |
|---------|----------------|------------|
| Fichier config | `.github/workflows/*.yml` | `.woodpecker.yml` |
| Jobs | `jobs:` (en parallèle par défaut) | `steps:` (avec `depends_on`) |
| Étapes | `steps:` (dans un job) | `commands:` (dans un step) |
| Image runner | `runs-on: ubuntu-latest` | `image: node:20` |
| Actions tierces | `uses: actions/checkout@v4` | `image: woodpeckerci/plugin-*` |
| Secrets | `${{ secrets.NAME }}` | `from_secret: name` |
| Conditions | `if:` | `when:` |
| Checkout code | `uses: actions/checkout@v4` | Automatique (clone git intégré) |
| Cache | `actions/cache@v4` | Volumes ou plugins cache |
| Parallélisme | Par défaut entre jobs | Explicite avec `depends_on` |

### 5.3 Questions de réflexion

**1. Dans quel contexte choisiriez-vous Gitea + Woodpecker plutôt que GitHub ?**

Réfléchissez aux cas d'usage :
- Projets avec contraintes de souveraineté (données sensibles, RGPD strict)
- Environnements déconnectés (air-gapped, intranet)
- Budgets limités avec beaucoup de CI/CD (pas de limite de minutes)
- Apprentissage et compréhension profonde du CI/CD
- Personnalisation extrême de la forge

**2. Quels sont les avantages d'avoir sa forge sur son propre serveur ?**

Points à considérer :
- Contrôle total des données (secrets, propriété intellectuelle)
- Pas de dépendance à un fournisseur externe
- Coût fixe (serveur) vs coût variable (minutes CI)
- Possibilité de modifier le code source (open source)
- Intégration avec infrastructure interne

**3. Comment Woodpecker gère-t-il les runners différemment de GitHub Actions ?**

Comparez :
- **GitHub** : runners cloud partagés ou self-hosted (VMs ou conteneurs)
- **Woodpecker** : agents Docker-native (chaque step = conteneur éphémère)
- **Isolation** : Woodpecker = isolation forte par conteneur
- **Performance** : Woodpecker peut être plus rapide (pas de temps de provisioning VM)
- **Scalabilité** : GitHub illimité, Woodpecker limité par vos agents

**4. Quels inconvénients voyez-vous au self-hosting ?**

Soyez honnêtes :
- Maintenance (mises à jour, sécurité, backups)
- Disponibilité (vous êtes responsable de l'uptime)
- Expertise requise (DevOps, Docker, réseau)
- Fonctionnalités manquantes (vs GitHub/GitLab)
- Pas de support officiel (communauté uniquement)

---

## Livrables

À la fin de ce TP bonus, vous devez avoir :

- [ ] Forge Gitea accessible sur http://localhost:3000
- [ ] Woodpecker CI accessible sur http://localhost:8000
- [ ] Repository TaskFlow mirrored dans Gitea
- [ ] Pipeline `.woodpecker.yml` fonctionnel (install, lint, test, build)
- [ ] Au moins un build vert dans Woodpecker
- [ ] Tableau comparatif complété
- [ ] Réponses aux questions de réflexion

---

## Ressources

- [Documentation Gitea](https://docs.gitea.com/)
- [Documentation Woodpecker CI](https://woodpecker-ci.org/docs/intro)
- [Woodpecker Plugins](https://woodpecker-ci.org/plugins)
- [Comparaison Gitea vs GitLab](https://docs.gitea.com/installation/comparison)
- [Architecture Woodpecker](https://woodpecker-ci.org/docs/administration/architecture)

---

## Pour aller plus loin

- **Build Docker** : ajoutez un step pour construire une image Docker avec `woodpeckerci/plugin-docker-buildx`
- **Secrets** : configurez des secrets dans Woodpecker pour des tokens de déploiement
- **Branch protection** : protégez la branche `main` dans Gitea (require passing CI)
- **Multi-workflows** : explorez le répertoire `.woodpecker/` pour plusieurs pipelines
- **Registry** : connectez Woodpecker au registry intégré de Gitea pour stocker vos images
- **Notifications** : ajoutez un plugin Slack/Discord pour recevoir les résultats de build
- **Matrix builds** : testez avec plusieurs versions de Node.js en parallèle
- **Production** : déployez sur un VPS avec HTTPS (Traefik + Let's Encrypt)
