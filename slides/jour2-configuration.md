---
marp: true
theme: uncover
paginate: true
footer: M2 ESTD - Initialisation CI avec GitHub Actions | ForEach Academy
style: |
  section {
    font-size: 20px;
    padding: 40px 50px;
  }
  h1 {
    font-size: 36px;
    color: #2563eb;
    margin: 0 0 15px 0;
  }
  h2 {
    font-size: 28px;
    color: #1e40af;
    margin: 0 0 12px 0;
  }
  h3 {
    font-size: 24px;
    color: #3b82f6;
    margin: 0 0 10px 0;
  }
  code {
    font-size: 18px;
    background: #f3f4f6;
    padding: 1px 4px;
    border-radius: 4px;
  }
  .highlight {
    background: linear-gradient(120deg, #3b82f6 0%, #2563eb 100%);
    padding: 2px 6px;
    border-radius: 4px;
    color: white;
    font-weight: bold;
  }
  table {
    font-size: 16px;
  }
  blockquote {
    border-left: 4px solid #3b82f6;
    padding-left: 15px;
    font-style: italic;
    color: #4b5563;
    margin: 10px 0;
    font-size: 18px;
  }
  ul {
    margin: 10px 0;
    padding-left: 25px;
  }
  li {
    margin-bottom: 5px;
    line-height: 1.3;
  }
  pre {
    font-size: 15px;
    padding: 20px;
    margin: 15px 0;
    background: #1e1e1e !important;
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
---

<!-- Mermaid support -->
<script type="module">
  import mermaid from 'https://cdn.jsdelivr.net/npm/mermaid@11/dist/mermaid.esm.min.mjs';
  mermaid.initialize({ startOnLoad: true, theme: 'default' });
</script>

<!-- _class: lead -->

# ⚙️ Jour 2
## Configuration avancée

**Formation CI/CD - GitHub Actions**
M2 ESTD - Architecte Web
ForEach Academy

---

## 👋 Rappel Jour 1

**Ce qu'on a appris** :
- ✅ CI = Intégrer souvent, tester automatiquement
- ✅ Pipeline, Jobs, Steps, Runners
- ✅ Premier workflow avec lint

**Aujourd'hui (Jour 2)** :
1. Variables et secrets
2. Jobs multiples et dépendances
3. Matrix builds
4. Triggers avancés

---

# Programme du jour

| Horaire | Contenu |
|---------|---------|
| 9h00-10h30 | **Variables, contextes et secrets** |
| 10h45-12h15 | **Jobs multiples et parallélisation** |
| 13h15-15h00 | **Déclencheurs et conditions** |
| 15h15-17h00 | **TP : Configuration avancée TaskFlow** |

---

# Objectifs du jour

À la fin de cette journée, vous serez capables de :

- ✅ Utiliser variables, contextes et secrets
- ✅ Créer des workflows multi-jobs
- ✅ Configurer des matrix builds
- ✅ Maîtriser les triggers avancés
- ✅ Appliquer des conditions d'exécution

---

<!-- _class: lead -->

# Partie 1
## Variables et Secrets

---

# Les contextes GitHub Actions

> **Contexte** : Objet contenant des informations sur le workflow, le job, le runner, etc.

```yaml
${{ github.repository }}     # owner/repo
${{ github.ref }}            # refs/heads/main
${{ github.sha }}            # commit SHA
${{ github.actor }}          # utilisateur qui a déclenché
${{ github.event_name }}     # push, pull_request, etc.
${{ runner.os }}             # Linux, Windows, macOS
${{ job.status }}            # success, failure, cancelled
```

---

# Contexte `github`

```yaml
name: Context Demo

on: push

jobs:
  info:
    runs-on: ubuntu-latest
    steps:
      - name: Show context
        run: |
          echo "Repository: ${{ github.repository }}"
          echo "Branch: ${{ github.ref_name }}"
          echo "Commit: ${{ github.sha }}"
          echo "Actor: ${{ github.actor }}"
          echo "Event: ${{ github.event_name }}"
          echo "Run ID: ${{ github.run_id }}"
          echo "Run Number: ${{ github.run_number }}"
```

---

# Variables d'environnement

## Trois niveaux

```yaml
env:                              # Niveau workflow
  NODE_ENV: production

jobs:
  build:
    env:                          # Niveau job
      CI: true
    runs-on: ubuntu-latest
    steps:
      - name: Build
        env:                      # Niveau step
          DEBUG: true
        run: npm run build
```

---

# Variables GitHub (Repository)

## Configuration dans Settings > Secrets and variables > Actions

```yaml
jobs:
  deploy:
    runs-on: ubuntu-latest
    steps:
      - name: Deploy
        run: |
          echo "Deploying version ${{ vars.APP_VERSION }}"
          echo "Environment: ${{ vars.ENVIRONMENT }}"
```

**Avantages** :
- ✅ Modifiables sans changer le code
- ✅ Différentes par environnement
- ✅ Visibles dans les logs

---

# Secrets

> **Secret** : Variable chiffrée, masquée dans les logs

## Configuration
Settings > Secrets and variables > Actions > New repository secret

```yaml
jobs:
  deploy:
    runs-on: ubuntu-latest
    steps:
      - name: Deploy to server
        env:
          API_KEY: ${{ secrets.API_KEY }}
          DATABASE_URL: ${{ secrets.DATABASE_URL }}
        run: |
          curl -H "Authorization: $API_KEY" https://api.example.com
```

---

# Secrets vs Variables

<div class="columns">
<div>

## 🔐 Secrets
- Données sensibles
- Chiffrés au repos
- Masqués dans les logs (`***`)
- API keys, tokens, passwords

</div>
<div>

## 📝 Variables
- Configuration non sensible
- Visibles dans les logs
- Modifiables facilement
- Versions, URLs publiques

</div>
</div>

```yaml
env:
  # Secret - masqué
  TOKEN: ${{ secrets.GITHUB_TOKEN }}
  # Variable - visible
  VERSION: ${{ vars.APP_VERSION }}
```

---

# Le secret GITHUB_TOKEN

> Token automatique fourni par GitHub pour chaque workflow

```yaml
jobs:
  release:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4

      - name: Create Release
        env:
          GITHUB_TOKEN: ${{ secrets.GITHUB_TOKEN }}
        run: gh release create v1.0.0
```

**Permissions par défaut** : lecture repo, écriture packages

---

# Configurer les permissions du token

```yaml
permissions:
  contents: write      # Push, create releases
  pull-requests: write # Comment on PRs
  issues: write        # Create/update issues
  packages: write      # Push to ghcr.io

jobs:
  release:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      - name: Push tag
        run: |
          git tag v1.0.0
          git push origin v1.0.0
```

---

# Environnements GitHub

> **Environment** : Contexte de déploiement avec secrets et règles spécifiques

```yaml
jobs:
  deploy-staging:
    environment: staging
    runs-on: ubuntu-latest
    steps:
      - run: echo "Deploy to ${{ vars.SERVER_URL }}"
        # Utilise les secrets/variables de l'environnement "staging"

  deploy-prod:
    environment: production
    runs-on: ubuntu-latest
    needs: deploy-staging
    steps:
      - run: echo "Deploy to ${{ vars.SERVER_URL }}"
        # Utilise les secrets/variables de l'environnement "production"
```

---

# Protection des environnements

## Settings > Environments > production

**Options de protection** :
- ✅ Required reviewers (approbation manuelle)
- ✅ Wait timer (délai avant déploiement)
- ✅ Deployment branches (branches autorisées)

```yaml
jobs:
  deploy:
    environment:
      name: production
      url: https://myapp.com
    runs-on: ubuntu-latest
```

---

<!-- _class: lead -->

# Partie 2
## Jobs multiples

---

# Workflow multi-jobs

```yaml
name: CI

on: push

jobs:
  lint:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      - run: npm ci
      - run: npm run lint

  test:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      - run: npm ci
      - run: npm test

  build:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      - run: npm ci
      - run: npm run build
```

---

# Exécution parallèle vs séquentielle

<div class="columns">
<div>

## ⚡ Sans `needs` (parallèle)

<div class="mermaid">
flowchart TB
    L[lint] --> F1[Fin]
    T[test] --> F2[Fin]
    B[build] --> F3[Fin]
</div>

**Durée : ~2 min**

</div>
<div>

## 🔗 Avec `needs` (séquentiel)

<div class="mermaid">
flowchart TB
    L[lint] --> T[test] --> B[build] --> F[Fin]
</div>

**Durée : ~6 min**

</div>
</div>

---

# Dépendances entre jobs

```yaml
jobs:
  lint:
    runs-on: ubuntu-latest
    steps:
      - run: npm run lint

  test:
    runs-on: ubuntu-latest
    steps:
      - run: npm test

  build:
    runs-on: ubuntu-latest
    needs: [lint, test]        # Attend lint ET test
    steps:
      - run: npm run build

  deploy:
    runs-on: ubuntu-latest
    needs: build               # Attend build
    steps:
      - run: ./deploy.sh
```

---

# Visualisation des dépendances

<div class="mermaid">
flowchart TB
    L[lint] --> B[build]
    T[test] --> B
    B --> D[deploy]
</div>

**lint** et **test** en parallèle → **build** → **deploy**

---

# Partager des données entre jobs

## Avec les artifacts

```yaml
jobs:
  build:
    runs-on: ubuntu-latest
    steps:
      - run: npm run build
      - uses: actions/upload-artifact@v4
        with:
          name: dist
          path: dist/

  deploy:
    needs: build
    runs-on: ubuntu-latest
    steps:
      - uses: actions/download-artifact@v4
        with:
          name: dist
          path: dist/
      - run: ./deploy.sh dist/
```

---

# Outputs entre jobs

```yaml
jobs:
  prepare:
    runs-on: ubuntu-latest
    outputs:
      version: ${{ steps.version.outputs.value }}
    steps:
      - id: version
        run: echo "value=1.2.3" >> $GITHUB_OUTPUT

  build:
    needs: prepare
    runs-on: ubuntu-latest
    steps:
      - run: |
          echo "Building version ${{ needs.prepare.outputs.version }}"
```

---

# Job conditionnel

```yaml
jobs:
  test:
    runs-on: ubuntu-latest
    steps:
      - run: npm test

  deploy:
    needs: test
    if: github.ref == 'refs/heads/main'
    runs-on: ubuntu-latest
    steps:
      - run: ./deploy.sh
```

**Le job deploy ne s'exécute que sur la branche main**

---

<!-- _class: lead -->

# Partie 3
## Matrix builds

---

# Qu'est-ce qu'un matrix build ?

> Exécuter le même job avec différentes combinaisons de paramètres

```yaml
jobs:
  test:
    runs-on: ubuntu-latest
    strategy:
      matrix:
        node-version: [18, 20, 22]
    steps:
      - uses: actions/setup-node@v4
        with:
          node-version: ${{ matrix.node-version }}
      - run: npm test
```

**Résultat** : 3 jobs en parallèle (Node 18, 20, 22)

---

# Matrix multi-dimensionnelle

```yaml
strategy:
  matrix:
    node-version: [18, 20, 22]
    os: [ubuntu-latest, windows-latest]

# Résultat : 6 combinaisons
# - Node 18 + Ubuntu
# - Node 18 + Windows
# - Node 20 + Ubuntu
# - Node 20 + Windows
# - Node 22 + Ubuntu
# - Node 22 + Windows
```

```yaml
runs-on: ${{ matrix.os }}
```

---

# Exclure des combinaisons

```yaml
strategy:
  matrix:
    node-version: [18, 20, 22]
    os: [ubuntu-latest, windows-latest, macos-latest]
    exclude:
      - node-version: 18
        os: macos-latest      # Node 18 pas testé sur macOS
```

**Résultat** : 8 combinaisons (9 - 1 exclue)

---

# Inclure des combinaisons spécifiques

```yaml
strategy:
  matrix:
    node-version: [18, 20]
    include:
      - node-version: 22
        os: ubuntu-latest
        experimental: true    # Variable custom
```

```yaml
steps:
  - run: npm test
    continue-on-error: ${{ matrix.experimental == true }}
```

---

# fail-fast

```yaml
strategy:
  fail-fast: true    # Par défaut
  matrix:
    node-version: [18, 20, 22]
```

<div class="columns">
<div>

## fail-fast: true
- ❌ Node 18 échoue
- ⏹️ Node 20 et 22 annulés
- Économise du temps

</div>
<div>

## fail-fast: false
- ❌ Node 18 échoue
- ✅ Node 20 continue
- ✅ Node 22 continue
- Résultat complet

</div>
</div>

---

# max-parallel

```yaml
strategy:
  max-parallel: 2    # Limite à 2 jobs simultanés
  matrix:
    node-version: [18, 20, 22]
```

**Utile pour** :
- Limiter l'utilisation des ressources
- Éviter les rate limits (API externes)
- Respecter les limites de minutes GitHub

---

# Exemple complet

```yaml
jobs:
  test:
    runs-on: ${{ matrix.os }}
    strategy:
      fail-fast: false
      matrix:
        os: [ubuntu-latest, windows-latest]
        node-version: [18, 20, 22]
        exclude:
          - os: windows-latest
            node-version: 18
    steps:
      - uses: actions/checkout@v4
      - uses: actions/setup-node@v4
        with:
          node-version: ${{ matrix.node-version }}
      - run: npm ci
      - run: npm test
```

---

<!-- _class: lead -->

# Partie 4
## Triggers avancés

---

# Rappel : Triggers de base

```yaml
on: push                        # Tout push

on: [push, pull_request]        # Push OU PR

on:
  push:
    branches: [main]            # Push sur main uniquement
```

---

# Filtrer par branches

```yaml
on:
  push:
    branches:
      - main
      - 'release/**'           # release/v1, release/v2
      - '!release/**-beta'     # Exclure les beta
    branches-ignore:
      - 'feature/**'           # Ignorer feature/*
```

---

# Filtrer par chemins

```yaml
on:
  push:
    paths:
      - 'src/**'               # Changements dans src/
      - 'package.json'
      - '!src/**/*.test.js'    # Sauf les tests
    paths-ignore:
      - 'docs/**'
      - '*.md'
```

**Utile pour** : Ne pas rebuilder si seule la doc change

---

# Filtrer par tags

```yaml
on:
  push:
    tags:
      - 'v*'                   # v1.0.0, v2.0.0
      - '!v*-beta'             # Exclure beta

# Exemple: Release automatique sur tag
jobs:
  release:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      - run: echo "Releasing ${{ github.ref_name }}"
```

---

# Pull Request events

```yaml
on:
  pull_request:
    types:
      - opened                  # PR créée
      - synchronize            # Nouveaux commits
      - reopened               # PR réouverte
      - closed                 # PR fermée
    branches:
      - main                   # PR vers main
```

---

# Schedule (CRON)

```yaml
on:
  schedule:
    - cron: '0 0 * * *'        # Tous les jours à minuit UTC
    - cron: '0 9 * * 1-5'      # Lun-Ven à 9h UTC

# Format: minute hour day month weekday
#         0-59   0-23 1-31 1-12  0-6 (0=dimanche)
```

**Cas d'usage** :
- Tests de sécurité nocturnes
- Nettoyage automatique
- Rapports hebdomadaires

---

# Déclenchement manuel

```yaml
on:
  workflow_dispatch:
    inputs:
      environment:
        description: 'Environnement cible'
        required: true
        default: 'staging'
        type: choice
        options:
          - staging
          - production
      debug:
        description: 'Mode debug'
        type: boolean
        default: false
```

---

# Utiliser les inputs manuels

```yaml
jobs:
  deploy:
    runs-on: ubuntu-latest
    steps:
      - name: Deploy
        run: |
          echo "Deploying to ${{ inputs.environment }}"
          if [ "${{ inputs.debug }}" == "true" ]; then
            echo "Debug mode enabled"
          fi
```

**Déclenchement** : Actions > Workflow > Run workflow

---

# Workflow call (réutilisable)

```yaml
# .github/workflows/reusable-build.yml
on:
  workflow_call:
    inputs:
      node-version:
        required: true
        type: string
    secrets:
      npm-token:
        required: true

jobs:
  build:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/setup-node@v4
        with:
          node-version: ${{ inputs.node-version }}
```

---

# Appeler un workflow réutilisable

```yaml
# .github/workflows/ci.yml
name: CI

on: push

jobs:
  call-build:
    uses: ./.github/workflows/reusable-build.yml
    with:
      node-version: '20'
    secrets:
      npm-token: ${{ secrets.NPM_TOKEN }}
```

---

# Conditions d'exécution

```yaml
jobs:
  test:
    runs-on: ubuntu-latest
    steps:
      - run: npm test

  deploy-staging:
    needs: test
    if: github.ref == 'refs/heads/develop'
    runs-on: ubuntu-latest
    steps:
      - run: ./deploy.sh staging

  deploy-prod:
    needs: test
    if: github.ref == 'refs/heads/main'
    runs-on: ubuntu-latest
    steps:
      - run: ./deploy.sh production
```

---

# Expressions conditionnelles

```yaml
# Condition sur le résultat d'un job
if: needs.test.result == 'success'

# Condition sur l'événement
if: github.event_name == 'pull_request'

# Condition sur l'acteur
if: github.actor != 'dependabot[bot]'

# Conditions combinées
if: |
  github.ref == 'refs/heads/main' &&
  github.event_name == 'push'

# Toujours exécuter (même si échec précédent)
if: always()

# Exécuter seulement en cas d'échec
if: failure()
```

---

# Fonctions utiles

```yaml
# Vérifier si contient
if: contains(github.event.head_commit.message, '[skip ci]')

# Vérifier le début
if: startsWith(github.ref, 'refs/tags/')

# Vérifier la fin
if: endsWith(github.repository, '-template')

# Format
run: echo "Date: ${{ format('{0}-{1}', 2024, 01) }}"

# JSON
run: echo '${{ toJSON(github.event) }}'
```

---

<!-- _class: lead -->

# Partie 5
## TP : Configuration avancée

---

# TP2 : Enrichir TaskFlow

## Objectifs

1. Ajouter un job de build séparé
2. Configurer une matrix Node 18/20/22
3. Utiliser des secrets pour les notifications
4. Ajouter un déploiement conditionnel

---

# Étape 1 : Multi-jobs

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
      - uses: actions/checkout@v4
      - uses: actions/setup-node@v4
        with:
          node-version: '20'
      - run: npm ci
      - run: npm run lint
```

---

# Étape 2 : Job test avec matrix

```yaml
  test:
    runs-on: ubuntu-latest
    needs: lint
    strategy:
      matrix:
        node-version: [18, 20, 22]
    steps:
      - uses: actions/checkout@v4
      - uses: actions/setup-node@v4
        with:
          node-version: ${{ matrix.node-version }}
      - run: npm ci
      - run: npm test
```

---

# Étape 3 : Job build

```yaml
  build:
    runs-on: ubuntu-latest
    needs: test
    steps:
      - uses: actions/checkout@v4
      - uses: actions/setup-node@v4
        with:
          node-version: '20'
      - run: npm ci
      - run: npm run build
      - uses: actions/upload-artifact@v4
        with:
          name: dist
          path: dist/
```

---

# Étape 4 : Déploiement conditionnel

```yaml
  deploy:
    runs-on: ubuntu-latest
    needs: build
    if: github.ref == 'refs/heads/main' && github.event_name == 'push'
    steps:
      - uses: actions/download-artifact@v4
        with:
          name: dist
          path: dist/
      - name: Deploy info
        run: |
          echo "Would deploy from commit ${{ github.sha }}"
          echo "By: ${{ github.actor }}"
          ls -la dist/
```

---

# Workflow complet

<div class="mermaid">
flowchart TB
    L[lint]
    L --> T[test matrix]
    T --> T18[Node 18]
    T --> T20[Node 20]
    T --> T22[Node 22]
    T18 --> B[build]
    T20 --> B
    T22 --> B
    B -->|if main| D[deploy]
</div>

---

# Récapitulatif Jour 2

## Ce qu'on a appris

- ✅ Variables, contextes et secrets
- ✅ Jobs multiples avec dépendances
- ✅ Matrix builds (multi-versions)
- ✅ Triggers avancés (paths, tags, schedule)
- ✅ Conditions d'exécution

## Demain (Jour 3)

- Tests automatisés avec Vitest
- Coverage et rapports
- Artifacts et cache optimisé

---

<!-- _class: lead -->

# Questions ?

---

# Ressources

## Documentation
- [Contexts](https://docs.github.com/en/actions/learn-github-actions/contexts)
- [Variables](https://docs.github.com/en/actions/learn-github-actions/variables)
- [Secrets](https://docs.github.com/en/actions/security-guides/encrypted-secrets)
- [Matrix](https://docs.github.com/en/actions/using-jobs/using-a-matrix-for-your-jobs)

## Projet
- [TaskFlow - Votre fork](https://github.com/YOUR-USER/taskflow)

---

<!-- _class: lead -->

# Merci !

**Jour 3 : Tests automatisés**
Vitest, coverage, artifacts
