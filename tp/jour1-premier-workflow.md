# TP Jour 1 : Introduction au CI - Premier workflow

> Durée : 3h30 (après-midi)

## Objectifs

- Installer et configurer Jenkins (CI self-hosted)
- Créer un premier job Jenkins
- Créer un premier workflow GitHub Actions
- Comparer les deux approches

---

## Partie 1 : Découverte de Jenkins (1h30)

### 1.1 Lancement de Jenkins avec Docker

Jenkins est l'outil CI historique. Nous allons l'installer via Docker pour comprendre son fonctionnement.

**Créez un dossier de travail :**

```bash
mkdir ~/jenkins-lab
cd ~/jenkins-lab
```

**Créez un fichier `docker-compose.yml` :**

```yaml
services:
  jenkins:
    image: jenkins/jenkins:lts
    container_name: jenkins
    ports:
      - "8080:8080"
      - "50000:50000"
    volumes:
      - jenkins_data:/var/jenkins_home
    environment:
      - JAVA_OPTS=-Djenkins.install.runSetupWizard=true

volumes:
  jenkins_data:
```

**Lancez Jenkins :**

```bash
docker compose up -d
```

**Récupérez le mot de passe initial :**

```bash
docker compose logs jenkins | grep -A 2 "initial"
# ou
docker exec jenkins cat /var/jenkins_home/secrets/initialAdminPassword
```

**Accédez à Jenkins :** http://localhost:8080

### 1.2 Configuration initiale de Jenkins

1. Entrez le mot de passe initial
2. Choisissez **"Install suggested plugins"**
3. Attendez l'installation (~2-3 minutes)
4. Créez un utilisateur admin :
   - Username: `admin`
   - Password: `admin` (pour le lab uniquement !)
   - Full name: `Votre Nom`
5. Gardez l'URL par défaut
6. Cliquez sur **"Start using Jenkins"**

### 1.3 Premier job Jenkins (Freestyle)

**Créer un nouveau job :**

1. Cliquez sur **"New Item"**
2. Nom : `hello-ci`
3. Type : **"Freestyle project"**
4. Cliquez **OK**

**Configuration du job :**

1. Section **"Build Steps"** → **"Add build step"** → **"Execute shell"**
2. Entrez le script :
```bash
echo "=== Mon premier job Jenkins ==="
echo "Date: $(date)"
echo "Hostname: $(hostname)"
echo "Utilisateur: $(whoami)"
echo "=== Job terminé avec succès ==="
```
3. Cliquez **"Save"**

**Exécuter le job :**

1. Cliquez **"Build Now"** (menu gauche)
2. Cliquez sur le build **#1** dans l'historique
3. Cliquez **"Console Output"** pour voir les logs

### 1.4 Job Jenkins avec Git

**Installer le plugin NodeJS :**

1. **Manage Jenkins** → **Plugins** → **Available plugins**
2. Recherchez **"NodeJS"**
3. Cochez et cliquez **"Install"**
4. Redémarrez Jenkins si demandé

**Configurer NodeJS :**

1. **Manage Jenkins** → **Tools**
2. Section **NodeJS** → **Add NodeJS**
   - Name: `Node 20`
   - Version: `NodeJS 20.x`
3. Cliquez **Save**

**Créer un job avec Git :**

1. **New Item** → Nom : `taskflow-jenkins` → **Freestyle project**
2. Section **Source Code Management** :
   - Sélectionnez **Git**
   - Repository URL : `https://github.com/Foreach-Academy-France/taskflow-starter.git`
   - Branch : `*/main`
3. Section **Build Environment** :
   - Cochez **"Provide Node & npm bin/ folder to PATH"**
   - Sélectionnez `Node 20`
4. Section **Build Steps** → **Execute shell** :
```bash
echo "=== Installation des dépendances ==="
npm ci

echo "=== Exécution du linter ==="
npm run lint

echo "=== Build terminé ==="
```
5. **Save** et **Build Now**

### 1.5 Pipeline Jenkins (Jenkinsfile)

Les pipelines sont la méthode moderne de Jenkins. Créons un pipeline déclaratif.

**Créer un pipeline :**

1. **New Item** → Nom : `taskflow-pipeline` → **Pipeline**
2. Section **Pipeline** → Definition : **Pipeline script**
3. Entrez le script :

```groovy
pipeline {
    agent any

    tools {
        nodejs 'Node 20'
    }

    stages {
        stage('Checkout') {
            steps {
                git url: 'https://github.com/Foreach-Academy-France/taskflow-starter.git',
                    branch: 'main'
            }
        }

        stage('Install') {
            steps {
                sh 'npm ci'
            }
        }

        stage('Lint') {
            steps {
                sh 'npm run lint'
            }
        }

        stage('Info') {
            steps {
                sh '''
                    echo "Node version: $(node --version)"
                    echo "NPM version: $(npm --version)"
                '''
            }
        }
    }

    post {
        success {
            echo '✅ Pipeline terminé avec succès !'
        }
        failure {
            echo '❌ Le pipeline a échoué'
        }
    }
}
```

4. **Save** et **Build Now**
5. Observez les stages dans l'interface graphique

### 1.6 Observations Jenkins

**Notez les points suivants :**

| Aspect | Observation |
|--------|-------------|
| Installation | Docker + config initiale (~10 min) |
| Interface | Web, graphique, beaucoup d'options |
| Configuration | Via interface ou Jenkinsfile (Groovy) |
| Plugins | Nécessaires pour Git, Node, etc. |
| Exécution | Sur le serveur Jenkins (agent) |
| Maintenance | Vous gérez le serveur |

**Arrêtez Jenkins** (on y reviendra peut-être) :
```bash
cd ~/jenkins-lab
docker compose stop
```

---

## Partie 2 : GitHub Actions (1h30)

Passons maintenant à GitHub Actions, l'approche cloud/SaaS.

### 2.1 Fork du projet TaskFlow

1. Allez sur https://github.com/Foreach-Academy-France/taskflow-starter
2. Cliquez **"Fork"** (en haut à droite)
3. Gardez les options par défaut
4. Cliquez **"Create fork"**

**Clonez votre fork :**

```bash
cd ~
git clone https://github.com/VOTRE-USERNAME/taskflow-starter.git
cd taskflow-starter
```

### 2.2 Exploration du projet

```bash
# Structure du projet
ls -la

# Installer les dépendances
npm install

# Lancer le linter
npm run lint

# Lancer en développement
npm run dev
```

Ouvrez http://localhost:5173 pour voir l'application.

### 2.3 Créer le workflow GitHub Actions

**Créez la structure :**

```bash
mkdir -p .github/workflows
```

**Créez le fichier `.github/workflows/ci.yml` :**

```yaml
name: CI

on:
  push:
    branches: [main]
  pull_request:
    branches: [main]

jobs:
  lint:
    name: Lint
    runs-on: ubuntu-latest

    steps:
      - name: Checkout code
        uses: actions/checkout@v4

      - name: Setup Node.js
        uses: actions/setup-node@v4
        with:
          node-version: '20'
          cache: 'npm'

      - name: Install dependencies
        run: npm ci

      - name: Run linter
        run: npm run lint
```

### 2.4 Commit et push

```bash
git add .github/workflows/ci.yml
git commit -m "feat: add CI workflow with linting"
git push origin main
```

### 2.5 Vérifier l'exécution

1. Allez sur votre repository GitHub
2. Cliquez sur l'onglet **"Actions"**
3. Vous devriez voir votre workflow en cours d'exécution
4. Cliquez dessus pour voir les détails
5. Explorez les logs de chaque step

### 2.6 Ajouter le badge de statut

Ajoutez dans votre `README.md` :

```markdown
# TaskFlow

[![CI](https://github.com/VOTRE-USERNAME/taskflow-starter/actions/workflows/ci.yml/badge.svg)](https://github.com/VOTRE-USERNAME/taskflow-starter/actions/workflows/ci.yml)

Application de gestion de tâches.
```

```bash
git add README.md
git commit -m "docs: add CI badge"
git push
```

### 2.7 Provoquer un échec

Modifiez `src/app.js` pour ajouter une erreur de lint :

```javascript
// Ajoutez cette ligne avec une variable non utilisée
const unusedVariable = "test";
```

```bash
git add src/app.js
git commit -m "test: add lint error"
git push
```

Observez le workflow échouer dans GitHub Actions, puis corrigez :

```bash
git revert HEAD
git push
```

---

## Partie 3 : Comparaison Jenkins vs GitHub Actions (30 min)

### 3.1 Tableau comparatif

Complétez ce tableau avec vos observations :

| Critère | Jenkins | GitHub Actions |
|---------|---------|----------------|
| Installation | | |
| Temps de mise en place | | |
| Configuration | | |
| Interface | | |
| Plugins/Actions | | |
| Coût | | |
| Maintenance | | |
| Intégration Git | | |

### 3.2 Questions de réflexion

1. **Quel outil choisiriez-vous pour un projet personnel ?** Pourquoi ?

2. **Quel outil choisiriez-vous pour une grande entreprise avec des contraintes de sécurité ?** Pourquoi ?

3. **Quels sont les avantages du Jenkinsfile par rapport à la configuration via interface ?**

4. **Pourquoi GitHub Actions est-il souvent préféré pour les projets open source ?**

### 3.3 Correspondances syntaxiques

| Concept | Jenkins (Groovy) | GitHub Actions (YAML) |
|---------|------------------|----------------------|
| Fichier config | `Jenkinsfile` | `.github/workflows/*.yml` |
| Ensemble du pipeline | `pipeline { }` | `jobs:` |
| Étape logique | `stage('Name')` | `jobs: name:` |
| Commande | `sh 'command'` | `run: command` |
| Environnement | `agent any` | `runs-on: ubuntu-latest` |
| Variables | `environment { }` | `env:` |
| Extensions | Plugins | Actions (`uses:`) |

---

## Livrables Jour 1

À la fin de ce TP, vous devez avoir :

- [ ] Jenkins installé et fonctionnel (docker compose)
- [ ] Un job Jenkins `taskflow-pipeline` qui passe
- [ ] Un fork de `taskflow-starter` sur votre GitHub
- [ ] Un workflow `.github/workflows/ci.yml` fonctionnel
- [ ] Le badge CI dans votre README
- [ ] Le tableau comparatif complété

---

## Ressources

- [Documentation Jenkins](https://www.jenkins.io/doc/)
- [Documentation GitHub Actions](https://docs.github.com/en/actions)
- [Migrating from Jenkins to GitHub Actions](https://docs.github.com/en/actions/migrating-to-github-actions/migrating-from-jenkins-to-github-actions)

---

## Pour aller plus loin

- Configurez un webhook GitHub dans Jenkins pour déclencher automatiquement les builds
- Explorez Blue Ocean, l'interface moderne de Jenkins
- Comparez les temps d'exécution entre Jenkins local et GitHub Actions cloud
