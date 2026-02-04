---
marp: true
theme: uncover
paginate: true
header: "**Module 1** - Introduction au CI"
footer: "M2 ESTD - Initialisation CI avec GitHub Actions | ForEach Academy"
style: |
  section {
    font-size: 20px;
    padding: 40px 50px;
  }
  h1 {
    font-size: 36px;
    color: #667eea;
    margin: 0 0 15px 0;
  }
  h2 {
    font-size: 28px;
    color: #764ba2;
    margin: 0 0 12px 0;
  }
  h3 {
    font-size: 24px;
    color: #667eea;
    margin: 0 0 10px 0;
  }
  code {
    font-size: 18px;
    background: #f1f5f9;
    padding: 2px 8px;
    border-radius: 4px;
  }
  pre {
    font-size: 15px;
    padding: 20px;
    margin: 15px 0;
    background: #1e293b !important;
    border-radius: 8px;
    box-shadow: 0 4px 6px rgba(0, 0, 0, 0.1);
  }
  pre code {
    background: transparent !important;
    color: #d4d4d4;
    font-size: 15px;
  }
  .columns {
    display: grid;
    grid-template-columns: 1fr 1fr;
    gap: 1rem;
  }
  blockquote {
    border-left: 4px solid #667eea;
    padding-left: 1rem;
    font-style: italic;
    color: #64748b;
  }
  table {
    font-size: 16px;
  }
  ul {
    margin: 10px 0;
    padding-left: 25px;
  }
  li {
    margin-bottom: 5px;
    line-height: 1.3;
  }
---

<!-- Mermaid support -->
<script type="module">
  import mermaid from 'https://cdn.jsdelivr.net/npm/mermaid@11/dist/mermaid.esm.min.mjs';
  mermaid.initialize({ startOnLoad: true, theme: 'default' });
</script>

# Module 1
## Introduction au Continuous Integration

![bg right:40% 80%](https://github.githubassets.com/images/modules/logos_page/GitHub-Mark.png)

**Jour 1 - Formation CI/CD**
ForEach Academy

---

# Programme du jour

| Horaire | Contenu |
|---------|---------|
| 9h00-9h30 | Accueil et introduction |
| 9h30-11h00 | **Principes et avantages du CI** |
| 11h15-12h15 | **Workflow et concepts CI** |
| 13h15-15h00 | **Outils CI populaires** |
| 15h15-17h00 | **TP : Premier workflow GitHub Actions** |

---

# Objectifs du jour

À la fin de cette journée, vous serez capables de :

- ✅ Expliquer ce qu'est l'intégration continue
- ✅ Identifier les avantages du CI/CD
- ✅ Comprendre les concepts : pipeline, job, step, artifact
- ✅ Comparer les principaux outils CI
- ✅ Créer votre premier workflow GitHub Actions

---

<!-- _class: lead -->

# Partie 1
## Principes et avantages du CI

---

# C'est quoi le problème ?

## Le développement "à l'ancienne"

<div class="mermaid">
flowchart TB
    subgraph A[" 👨‍💻 Développeur A "]
        A1[Code pendant 2 semaines]
    end
    subgraph B[" 👩‍💻 Développeur B "]
        B1[Code pendant 2 semaines]
    end
    A1 --> M[🔀 MERGE]
    B1 --> M
    M --> C["💥 CONFLITS<br/>💥 BUGS<br/>💥 STRESS"]
</div>

---

# L'enfer de l'intégration

> "Integration Hell" - Le moment où tout le monde essaie de merger son code en même temps

## Symptômes classiques :

- 🔥 Conflits de merge massifs
- 🐛 Bugs découverts tardivement
- 😰 "Ça marchait sur ma machine !"
- 📅 Retards de livraison
- 💸 Coûts de correction élevés

---

# La solution : Intégration Continue

## Définition

> **Continuous Integration (CI)** : Pratique de développement où les développeurs intègrent leur code fréquemment (plusieurs fois par jour), chaque intégration étant vérifiée par un build automatisé.

**Martin Fowler, 2006**

---

# Le principe fondamental

```
        Code        Push        CI Server       Feedback
          |          |             |               |
Dev A --> commit --> push --> [BUILD + TEST] --> ✅ OK (2 min)
          |          |             |               |
Dev B --> commit --> push --> [BUILD + TEST] --> ❌ FAIL (2 min)
          |          |             |               |
          |          |             |          Fix immédiat !
```

## La règle d'or
> **Intégrer souvent, détecter vite, corriger immédiatement**

---

# Avant vs Après CI

<div class="columns">
<div>

## ❌ Sans CI

- Intégration rare (semaines/mois)
- Tests manuels
- "Ça marche chez moi"
- Bugs découverts tard
- Stress avant release
- Déploiement = événement

</div>
<div>

## ✅ Avec CI

- Intégration continue (heures)
- Tests automatiques
- Même environnement pour tous
- Bugs détectés immédiatement
- Confiance permanente
- Déploiement = routine

</div>
</div>

---

# Les bénéfices du CI

## Pour les développeurs
- 🔍 Feedback rapide (minutes vs jours)
- 🐛 Bugs faciles à identifier (petit diff)
- 🧘 Moins de stress, plus de confiance

## Pour le projet
- 📦 Toujours un build fonctionnel
- 📊 Qualité de code mesurable
- 🚀 Releases plus fréquentes

## Pour le business
- 💰 Réduction des coûts de correction
- ⏱️ Time-to-market réduit
- 😊 Clients satisfaits

---

# Le coût d'un bug

![bg right:50% 90%](https://www.researchgate.net/publication/255965523/figure/fig1/AS:297951104544776@1448046358036/Relative-cost-to-fix-a-defect-depending-on-when-it-is-found.png)

Plus un bug est détecté tard, plus il coûte cher à corriger.

**CI = Détecter au plus tôt**

---

# CI vs CD vs CD

```
┌─────────────────────────────────────────────────────────────────┐
│                                                                 │
│  ┌──────────────────┐                                          │
│  │ Continuous       │  Code → Build → Test                     │
│  │ Integration (CI) │  "Le code est-il correct ?"              │
│  └──────────────────┘                                          │
│           │                                                     │
│           ▼                                                     │
│  ┌──────────────────┐                                          │
│  │ Continuous       │  ... → Package → Deploy to Staging       │
│  │ Delivery         │  "Le code est-il livrable ?"             │
│  └──────────────────┘  (déploiement manuel en prod)            │
│           │                                                     │
│           ▼                                                     │
│  ┌──────────────────┐                                          │
│  │ Continuous       │  ... → Deploy to Production              │
│  │ Deployment       │  "Tout est automatique"                  │
│  └──────────────────┘                                          │
│                                                                 │
└─────────────────────────────────────────────────────────────────┘
```

---

# Les pratiques essentielles du CI

## Les 10 commandements

1. **Maintenir un repository unique** (Git)
2. **Automatiser le build**
3. **Rendre le build auto-testant**
4. **Chaque commit déclenche un build**
5. **Garder le build rapide** (< 10 min)
6. **Tester dans un clone de production**
7. **Rendre les artifacts accessibles**
8. **Tout le monde voit les résultats**
9. **Automatiser le déploiement**
10. **Corriger immédiatement si le build casse**

---

<!-- _class: lead -->

# Partie 2
## Workflow et concepts CI

---

# Anatomie d'un pipeline CI

```yaml
Pipeline
│
├── Job 1: Lint
│   ├── Step 1: Checkout code
│   ├── Step 2: Install dependencies
│   └── Step 3: Run ESLint
│
├── Job 2: Test
│   ├── Step 1: Checkout code
│   ├── Step 2: Install dependencies
│   ├── Step 3: Run tests
│   └── Step 4: Upload coverage
│
└── Job 3: Build
    ├── Step 1: Checkout code
    ├── Step 2: Install dependencies
    ├── Step 3: Build application
    └── Step 4: Upload artifact
```

---

# Les concepts clés

## Pipeline (Workflow)
> Ensemble des opérations automatisées déclenchées par un événement

## Job
> Unité de travail indépendante qui s'exécute sur un runner

## Step
> Action individuelle au sein d'un job (commande, script, action)

## Runner
> Machine (virtuelle ou physique) qui exécute les jobs

---

# Triggers (Déclencheurs)

```yaml
# Exemples de déclencheurs

on: push                    # À chaque push

on: pull_request            # À chaque PR

on:
  push:
    branches: [main]        # Push sur main uniquement

on:
  schedule:
    - cron: '0 0 * * *'     # Tous les jours à minuit

on: workflow_dispatch       # Déclenchement manuel
```

---

# Artifacts

> **Artifact** : Fichier ou dossier produit par un job et conservé après son exécution

## Exemples d'artifacts :

- 📦 Application buildée (dist/)
- 📊 Rapport de coverage
- 📝 Logs de tests
- 🐳 Image Docker

## Utilité :
- Partager entre jobs
- Télécharger pour debug
- Déployer

---

# Cache vs Artifacts

<div class="columns">
<div>

## Cache
- Accélère les builds
- `node_modules/`
- Clé basée sur `package-lock.json`
- Automatiquement invalidé

</div>
<div>

## Artifacts
- Résultats à conserver
- `dist/`, `coverage/`
- Téléchargeables
- Expiration configurable

</div>
</div>

```yaml
# Cache
- uses: actions/cache@v4
  with:
    path: node_modules
    key: ${{ hashFiles('package-lock.json') }}

# Artifact
- uses: actions/upload-artifact@v4
  with:
    name: coverage-report
    path: coverage/
```

---

# Environnements d'exécution

## Runners hébergés (GitHub-hosted)

| Runner | OS | Specs |
|--------|----|----|
| `ubuntu-latest` | Ubuntu 22.04 | 2 CPU, 7 GB RAM |
| `windows-latest` | Windows Server 2022 | 2 CPU, 7 GB RAM |
| `macos-latest` | macOS 12 | 3 CPU, 14 GB RAM |

## Runners self-hosted
- Votre propre machine
- Contrôle total
- Coût réduit pour gros volumes

---

# Secrets et Variables

## Secrets (données sensibles)
```yaml
env:
  API_KEY: ${{ secrets.API_KEY }}
  DATABASE_URL: ${{ secrets.DATABASE_URL }}
```
- ✅ Chiffrés
- ✅ Masqués dans les logs
- ⚠️ Ne jamais commiter !

## Variables (configuration)
```yaml
env:
  NODE_ENV: production
  APP_VERSION: ${{ vars.APP_VERSION }}
```
- Configuration non sensible
- Visibles dans les logs

---

<!-- _class: lead -->

# Partie 3
## Outils CI populaires

---

# Panorama des outils CI/CD

```
┌─────────────────────────────────────────────────────────────────┐
│                                                                 │
│  Self-hosted          │    Cloud / SaaS                        │
│  ──────────────────   │    ─────────────                        │
│                       │                                         │
│  • Jenkins            │    • GitHub Actions ⭐                  │
│  • GitLab CI (aussi)  │    • GitLab CI                          │
│  • TeamCity           │    • CircleCI                           │
│  • Bamboo             │    • Travis CI                          │
│  • Drone              │    • Azure DevOps                       │
│  • Woodpecker         │    • AWS CodePipeline                   │
│                       │                                         │
└─────────────────────────────────────────────────────────────────┘
```

---

# Jenkins - Le vétéran

![bg right:30% 80%](https://www.jenkins.io/images/logos/jenkins/jenkins.png)

## Caractéristiques
- 🎂 Créé en 2011 (fork de Hudson)
- 🔧 Self-hosted (vous gérez le serveur)
- 🧩 1800+ plugins
- 📝 Jenkinsfile (Groovy)

## Avantages
- ✅ Très flexible et personnalisable
- ✅ Grande communauté
- ✅ Gratuit et open source

## Inconvénients
- ❌ Maintenance du serveur
- ❌ Configuration complexe
- ❌ Interface datée

---

# Exemple Jenkinsfile

```groovy
pipeline {
    agent any

    stages {
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
        stage('Test') {
            steps {
                sh 'npm test'
            }
        }
        stage('Build') {
            steps {
                sh 'npm run build'
            }
        }
    }
}
```

---

# GitHub Actions - Le moderne

![bg right:30% 80%](https://github.githubassets.com/images/modules/logos_page/GitHub-Mark.png)

## Caractéristiques
- 🎂 Lancé en 2019
- ☁️ Cloud (intégré à GitHub)
- 🛒 Marketplace (15 000+ actions)
- 📝 YAML

## Avantages
- ✅ Intégration native GitHub
- ✅ Pas de serveur à gérer
- ✅ Gratuit pour repos publics
- ✅ Syntaxe simple

## Inconvénients
- ❌ Limité à GitHub
- ❌ Minutes limitées (repos privés)

---

# Exemple GitHub Actions

```yaml
name: CI

on: [push, pull_request]

jobs:
  build:
    runs-on: ubuntu-latest

    steps:
      - uses: actions/checkout@v4

      - uses: actions/setup-node@v4
        with:
          node-version: '20'

      - run: npm ci
      - run: npm run lint
      - run: npm test
      - run: npm run build
```

---

# Comparatif rapide

| Critère | Jenkins | GitHub Actions | GitLab CI |
|---------|---------|----------------|-----------|
| Hébergement | Self-hosted | Cloud | Les deux |
| Configuration | Groovy | YAML | YAML |
| Intégration Git | Plugins | Native | Native |
| Courbe apprentissage | Moyenne | Facile | Facile |
| Coût | Infra | Gratuit* | Gratuit* |
| Marketplace | 1800 plugins | 15000 actions | Templates |

*Avec limites sur repos privés

---

# Pourquoi GitHub Actions pour ce cours ?

## 1. Intégration native
Pas besoin de configurer de webhooks, tout est intégré

## 2. Zéro infrastructure
Pas de serveur Jenkins à maintenir

## 3. Syntaxe moderne
YAML simple et lisible

## 4. Marketplace riche
Actions prêtes à l'emploi pour tout

## 5. Concepts transférables
Ce que vous apprenez s'applique aux autres outils

---

# Migration Jenkins → GitHub Actions

| Jenkins | GitHub Actions |
|---------|----------------|
| `Jenkinsfile` | `.github/workflows/*.yml` |
| `pipeline { }` | `jobs:` |
| `stage('Build')` | `jobs: build:` |
| `steps { sh 'cmd' }` | `steps: - run: cmd` |
| `agent any` | `runs-on: ubuntu-latest` |
| `environment { }` | `env:` |
| Plugins | Actions (uses:) |

---

# Structure d'un workflow GitHub Actions

```yaml
name: CI                          # Nom du workflow

on: [push, pull_request]          # Déclencheurs

jobs:                             # Liste des jobs
  build:                          # Nom du job
    runs-on: ubuntu-latest        # Runner

    steps:                        # Étapes du job
      - uses: actions/checkout@v4 # Action
      - run: npm install          # Commande
```

---

# L'écosystème GitHub Actions

## Actions officielles (actions/*)
- `actions/checkout` - Clone le repo
- `actions/setup-node` - Installe Node.js
- `actions/cache` - Cache des dépendances
- `actions/upload-artifact` - Upload fichiers

## Actions communautaires
- `docker/build-push-action` - Build Docker
- `peaceiris/actions-gh-pages` - Deploy Pages
- `codecov/codecov-action` - Coverage

## Vos propres actions
- JavaScript ou Docker
- Réutilisables

---

<!-- _class: lead -->

# Partie 4
## TP : Premier workflow

---

# Le projet TaskFlow

![bg right:40% 90%](https://raw.githubusercontent.com/Foreach-Academy-France/taskflow-starter/main/screenshot.png)

## Application de gestion de tâches

- Vanilla JS + Vite
- Tests avec Vitest
- ESLint + Prettier

## Objectif du TP

Créer votre premier workflow CI qui :
1. Clone le code
2. Installe les dépendances
3. Lance le linter

---

# Étapes du TP

## 1. Fork du projet starter
```bash
gh repo fork Foreach-Academy-France/taskflow-starter --clone
cd taskflow-starter
npm install
```

## 2. Créer le workflow
```bash
mkdir -p .github/workflows
touch .github/workflows/ci.yml
```

## 3. Configurer et pousser
```bash
git add .
git commit -m "feat: add CI workflow"
git push
```

---

# Workflow de base

```yaml
name: CI

on:
  push:
    branches: [main]
  pull_request:
    branches: [main]

jobs:
  lint:
    runs-on: ubuntu-latest

    steps:
      - name: Checkout code
        uses: actions/checkout@v4

      - name: Setup Node.js
        uses: actions/setup-node@v4
        with:
          node-version: '20'

      - name: Install dependencies
        run: npm ci

      - name: Run linter
        run: npm run lint
```

---

# Vérifier le résultat

## Dans GitHub

1. Aller sur votre repo
2. Onglet **Actions**
3. Voir le workflow en cours/terminé
4. Cliquer pour voir les logs

## Badge de statut

Ajouter dans README.md :
```markdown
[![CI](https://github.com/VOTRE-USER/taskflow/actions/workflows/ci.yml/badge.svg)](https://github.com/VOTRE-USER/taskflow/actions)
```

---

# Récapitulatif Jour 1

## Ce qu'on a appris

- ✅ CI = Intégrer souvent, tester automatiquement
- ✅ Pipeline = Jobs = Steps
- ✅ Triggers, Artifacts, Runners, Secrets
- ✅ Jenkins vs GitHub Actions
- ✅ Premier workflow fonctionnel

## Demain (Jour 2)

- Jobs multiples et dépendances
- Matrix builds (Node 18/20/22)
- Secrets et variables
- Triggers avancés

---

<!-- _class: lead -->

# Questions ?

---

# Ressources

## Documentation
- [GitHub Actions Docs](https://docs.github.com/en/actions)
- [Workflow syntax](https://docs.github.com/en/actions/reference/workflow-syntax-for-github-actions)

## Cheatsheet
- [Cheatsheet GitHub Actions](../ressources/cheatsheet.md)

## Projet
- [TaskFlow Starter](https://github.com/Foreach-Academy-France/taskflow-starter)

---

<!-- _class: lead -->

# Merci !

**Jour 2 : Configuration avancée**
Matrix, secrets, triggers
