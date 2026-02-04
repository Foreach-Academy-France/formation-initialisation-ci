# TP Jour 2 : Configuration avancée

> **Durée** : 2h | **Objectif** : Jobs multiples, matrix builds, secrets et triggers avancés

## Point de départ

Vous devez avoir complété le TP Jour 1 avec :
- ✅ Workflow CI avec job `lint`
- ✅ Badge dans le README

---

## Étape 1 : Ajouter un job Build (20 min)

### 1.1 Modifier le workflow

Ouvrir `.github/workflows/ci.yml` et ajouter un job `build` :

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
      - uses: actions/checkout@v4
      - uses: actions/setup-node@v4
        with:
          node-version: '20'
          cache: 'npm'
      - run: npm ci
      - run: npm run lint

  build:
    name: Build
    runs-on: ubuntu-latest
    needs: lint                    # Attend que lint soit terminé
    steps:
      - uses: actions/checkout@v4
      - uses: actions/setup-node@v4
        with:
          node-version: '20'
          cache: 'npm'
      - run: npm ci
      - run: npm run build

      - name: Upload build artifact
        uses: actions/upload-artifact@v4
        with:
          name: dist
          path: dist/
          retention-days: 7
```

### 1.2 Comprendre les nouveautés

| Élément | Description |
|---------|-------------|
| `needs: lint` | Le job `build` attend que `lint` termine avec succès |
| `upload-artifact` | Sauvegarde le dossier `dist/` pour téléchargement |
| `retention-days: 7` | L'artifact est conservé 7 jours |

### 1.3 Commit et vérifier

```bash
git add .github/workflows/ci.yml
git commit -m "feat(ci): add build job with artifact upload"
git push origin main
```

**Vérifier** dans Actions :
```
✅ CI
   ├── ✅ lint
   └── ✅ build (après lint)
```

---

## Étape 2 : Matrix Build (25 min)

### 2.1 Ajouter une matrix pour tester plusieurs versions Node.js

Modifier le job `lint` pour utiliser une matrix :

```yaml
  lint:
    name: Lint (Node ${{ matrix.node-version }})
    runs-on: ubuntu-latest
    strategy:
      matrix:
        node-version: [18, 20, 22]
    steps:
      - uses: actions/checkout@v4
      - uses: actions/setup-node@v4
        with:
          node-version: ${{ matrix.node-version }}
          cache: 'npm'
      - run: npm ci
      - run: npm run lint
```

### 2.2 Mettre à jour le job build

Le job `build` doit maintenant attendre toutes les versions :

```yaml
  build:
    name: Build
    runs-on: ubuntu-latest
    needs: lint                    # Attend TOUS les jobs lint
    steps:
      - uses: actions/checkout@v4
      - uses: actions/setup-node@v4
        with:
          node-version: '20'       # Build avec Node 20 seulement
          cache: 'npm'
      - run: npm ci
      - run: npm run build
      - uses: actions/upload-artifact@v4
        with:
          name: dist
          path: dist/
```

### 2.3 Commit et observer

```bash
git add .github/workflows/ci.yml
git commit -m "feat(ci): add matrix build for Node 18, 20, 22"
git push origin main
```

**Résultat attendu** :
```
✅ CI
   ├── ✅ Lint (Node 18)
   ├── ✅ Lint (Node 20)
   ├── ✅ Lint (Node 22)
   └── ✅ Build
```

---

## Étape 3 : Variables et Secrets (20 min)

### 3.1 Créer une variable repository

1. Aller dans **Settings > Secrets and variables > Actions**
2. Onglet **Variables**
3. Cliquer **New repository variable**
4. Nom : `APP_VERSION`
5. Valeur : `1.0.0`

### 3.2 Créer un secret (simulation)

1. Onglet **Secrets**
2. Cliquer **New repository secret**
3. Nom : `DEPLOY_TOKEN`
4. Valeur : `demo-token-12345` (valeur fictive pour le TP)

### 3.3 Utiliser dans le workflow

Ajouter un step dans le job `build` :

```yaml
  build:
    name: Build
    runs-on: ubuntu-latest
    needs: lint
    steps:
      - uses: actions/checkout@v4
      - uses: actions/setup-node@v4
        with:
          node-version: '20'
          cache: 'npm'
      - run: npm ci

      - name: Show version info
        run: |
          echo "Building version: ${{ vars.APP_VERSION }}"
          echo "Commit: ${{ github.sha }}"
          echo "Branch: ${{ github.ref_name }}"

      - run: npm run build

      - uses: actions/upload-artifact@v4
        with:
          name: dist
          path: dist/
```

### 3.4 Commit et vérifier

```bash
git add .github/workflows/ci.yml
git commit -m "feat(ci): use repository variables"
git push origin main
```

Dans les logs, vous verrez :
```
Building version: 1.0.0
Commit: abc123...
Branch: main
```

---

## Étape 4 : Triggers avancés (20 min)

### 4.1 Ajouter workflow_dispatch (déclenchement manuel)

Modifier la section `on:` :

```yaml
on:
  push:
    branches: [main]
  pull_request:
    branches: [main]
  workflow_dispatch:
    inputs:
      environment:
        description: 'Target environment'
        required: true
        default: 'staging'
        type: choice
        options:
          - staging
          - production
      debug:
        description: 'Enable debug mode'
        type: boolean
        default: false
```

### 4.2 Utiliser les inputs

Ajouter un step conditionnel dans `build` :

```yaml
      - name: Debug info
        if: inputs.debug == true
        run: |
          echo "🐛 Debug mode enabled"
          echo "Environment: ${{ inputs.environment }}"
          node --version
          npm --version
```

### 4.3 Commit

```bash
git add .github/workflows/ci.yml
git commit -m "feat(ci): add manual workflow dispatch with inputs"
git push origin main
```

### 4.4 Tester le déclenchement manuel

1. Aller dans **Actions > CI**
2. Cliquer **Run workflow**
3. Sélectionner `staging` et cocher `debug`
4. Cliquer **Run workflow**

---

## Étape 5 : Filtrage par paths (15 min)

### 5.1 Ne pas rebuilder pour la documentation

Modifier la section `on:` :

```yaml
on:
  push:
    branches: [main]
    paths-ignore:
      - '**.md'
      - 'docs/**'
      - '.gitignore'
  pull_request:
    branches: [main]
  workflow_dispatch:
    inputs:
      environment:
        description: 'Target environment'
        required: true
        default: 'staging'
        type: choice
        options:
          - staging
          - production
```

### 5.2 Tester

```bash
# Modifier seulement le README
echo "\n\nUpdated!" >> README.md
git add README.md
git commit -m "docs: update readme"
git push origin main
```

**Résultat** : Le workflow NE DOIT PAS se déclencher (car seul un `.md` a changé).

---

## Étape 6 : Déploiement conditionnel (20 min)

### 6.1 Ajouter un job deploy (simulé)

Ajouter à la fin du workflow :

```yaml
  deploy:
    name: Deploy
    runs-on: ubuntu-latest
    needs: build
    if: github.ref == 'refs/heads/main' && github.event_name == 'push'
    steps:
      - name: Download artifact
        uses: actions/download-artifact@v4
        with:
          name: dist
          path: dist/

      - name: Deploy info
        run: |
          echo "🚀 Would deploy to ${{ inputs.environment || 'production' }}"
          echo "Files to deploy:"
          ls -la dist/
```

### 6.2 Workflow complet

Votre fichier `.github/workflows/ci.yml` devrait ressembler à :

```yaml
name: CI

on:
  push:
    branches: [main]
    paths-ignore:
      - '**.md'
      - 'docs/**'
  pull_request:
    branches: [main]
  workflow_dispatch:
    inputs:
      environment:
        description: 'Target environment'
        required: true
        default: 'staging'
        type: choice
        options:
          - staging
          - production
      debug:
        description: 'Enable debug mode'
        type: boolean
        default: false

jobs:
  lint:
    name: Lint (Node ${{ matrix.node-version }})
    runs-on: ubuntu-latest
    strategy:
      matrix:
        node-version: [18, 20, 22]
    steps:
      - uses: actions/checkout@v4
      - uses: actions/setup-node@v4
        with:
          node-version: ${{ matrix.node-version }}
          cache: 'npm'
      - run: npm ci
      - run: npm run lint

  build:
    name: Build
    runs-on: ubuntu-latest
    needs: lint
    steps:
      - uses: actions/checkout@v4
      - uses: actions/setup-node@v4
        with:
          node-version: '20'
          cache: 'npm'
      - run: npm ci

      - name: Show version info
        run: |
          echo "Building version: ${{ vars.APP_VERSION }}"
          echo "Commit: ${{ github.sha }}"

      - name: Debug info
        if: inputs.debug == true
        run: |
          echo "🐛 Debug mode enabled"
          node --version
          npm --version

      - run: npm run build

      - uses: actions/upload-artifact@v4
        with:
          name: dist
          path: dist/

  deploy:
    name: Deploy
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
          echo "🚀 Would deploy to production"
          ls -la dist/
```

### 6.3 Commit final

```bash
git add .github/workflows/ci.yml
git commit -m "feat(ci): add conditional deploy job"
git push origin main
```

---

## Checklist de validation

Avant de passer au Jour 3, vérifiez :

- [ ] Matrix build avec Node 18, 20, 22
- [ ] Job `build` dépend de `lint`
- [ ] Artifact `dist/` uploadé
- [ ] Variable `APP_VERSION` configurée et utilisée
- [ ] Secret `DEPLOY_TOKEN` créé (même si non utilisé)
- [ ] Workflow dispatch fonctionnel
- [ ] Filtrage par paths actif
- [ ] Job `deploy` conditionnel (seulement sur main)

---

## Erreurs courantes

### Matrix : Un job échoue, les autres continuent

**Cause** : `fail-fast: true` par défaut

**Solution** (si vous voulez voir tous les résultats) :
```yaml
strategy:
  fail-fast: false
  matrix:
    node-version: [18, 20, 22]
```

### L'artifact n'est pas téléchargeable

**Vérifier** :
1. Le step `upload-artifact` a bien réussi
2. Le `path: dist/` existe (le build a généré les fichiers)

### Le deploy job ne s'exécute pas

**Cause** : La condition `if:` n'est pas remplie

**Vérifier** :
- Vous êtes sur la branche `main`
- C'est un `push` (pas une PR)

---

## Ressources

- [Matrix builds](https://docs.github.com/en/actions/using-jobs/using-a-matrix-for-your-jobs)
- [Variables](https://docs.github.com/en/actions/learn-github-actions/variables)
- [Secrets](https://docs.github.com/en/actions/security-guides/encrypted-secrets)
- [Workflow triggers](https://docs.github.com/en/actions/using-workflows/events-that-trigger-workflows)

---

**Prochain TP** : [Jour 3 - Tests et qualité](./jour3-tests-qualite.md)
