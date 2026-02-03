# GitHub Actions Cheatsheet

## Structure de base

```yaml
name: Mon Workflow

on:
  push:
    branches: [main]
  pull_request:
    branches: [main]

jobs:
  mon-job:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      - name: Ma step
        run: echo "Hello World"
```

## Triggers courants

```yaml
# Push sur main
on:
  push:
    branches: [main]

# Pull request
on:
  pull_request:
    branches: [main]

# Manuel
on:
  workflow_dispatch:
    inputs:
      environment:
        description: 'Environment to deploy'
        required: true
        default: 'staging'

# Planifié (cron)
on:
  schedule:
    - cron: '0 0 * * *'  # Minuit UTC chaque jour

# Plusieurs événements
on: [push, pull_request]
```

## Jobs

```yaml
jobs:
  # Job simple
  build:
    runs-on: ubuntu-latest
    steps:
      - run: echo "Building..."

  # Job avec dépendance
  deploy:
    needs: build
    runs-on: ubuntu-latest
    steps:
      - run: echo "Deploying..."

  # Matrix
  test:
    runs-on: ubuntu-latest
    strategy:
      matrix:
        node: [18, 20, 22]
    steps:
      - uses: actions/setup-node@v4
        with:
          node-version: ${{ matrix.node }}
```

## Actions courantes

```yaml
# Checkout
- uses: actions/checkout@v4

# Setup Node.js
- uses: actions/setup-node@v4
  with:
    node-version: '20'
    cache: 'npm'

# Setup Bun
- uses: oven-sh/setup-bun@v2
  with:
    bun-version: latest

# Cache
- uses: actions/cache@v4
  with:
    path: ~/.npm
    key: ${{ runner.os }}-node-${{ hashFiles('**/package-lock.json') }}

# Upload artifact
- uses: actions/upload-artifact@v4
  with:
    name: my-artifact
    path: dist/

# Download artifact
- uses: actions/download-artifact@v4
  with:
    name: my-artifact
```

## Secrets et Variables

```yaml
# Utiliser un secret
- name: Deploy
  env:
    API_KEY: ${{ secrets.API_KEY }}
  run: ./deploy.sh

# Utiliser une variable
- name: Build
  env:
    APP_ENV: ${{ vars.ENVIRONMENT }}
  run: npm run build
```

## Conditions

```yaml
# Condition sur branch
- name: Deploy to prod
  if: github.ref == 'refs/heads/main'
  run: ./deploy-prod.sh

# Condition sur événement
- name: Comment on PR
  if: github.event_name == 'pull_request'
  run: echo "This is a PR"

# Condition sur succès/échec
- name: Notify on failure
  if: failure()
  run: echo "Something failed"

# Condition sur expression
- name: Skip if draft
  if: ${{ !github.event.pull_request.draft }}
  run: npm test
```

## Contextes utiles

```yaml
# Informations sur le commit
${{ github.sha }}           # SHA complet
${{ github.ref }}           # refs/heads/main
${{ github.ref_name }}      # main
${{ github.actor }}         # Username

# Informations sur le runner
${{ runner.os }}            # Linux, Windows, macOS
${{ runner.arch }}          # X64, ARM64

# Informations sur le job
${{ job.status }}           # success, failure, cancelled

# Secrets
${{ secrets.GITHUB_TOKEN }} # Token auto-généré
${{ secrets.MY_SECRET }}    # Secret personnalisé
```

## Outputs entre jobs

```yaml
jobs:
  job1:
    runs-on: ubuntu-latest
    outputs:
      version: ${{ steps.get-version.outputs.version }}
    steps:
      - id: get-version
        run: echo "version=1.0.0" >> $GITHUB_OUTPUT

  job2:
    needs: job1
    runs-on: ubuntu-latest
    steps:
      - run: echo "Version is ${{ needs.job1.outputs.version }}"
```

## Concurrency

```yaml
# Annuler les runs précédents
concurrency:
  group: ${{ github.workflow }}-${{ github.ref }}
  cancel-in-progress: true
```

## Permissions

```yaml
permissions:
  contents: read
  pull-requests: write
  issues: write
  pages: write
  id-token: write
```

## Docker

```yaml
# Build et push
- name: Build and push
  uses: docker/build-push-action@v5
  with:
    context: .
    push: true
    tags: user/app:latest

# Login to registry
- name: Login to Docker Hub
  uses: docker/login-action@v3
  with:
    username: ${{ secrets.DOCKER_USERNAME }}
    password: ${{ secrets.DOCKER_PASSWORD }}
```

## Debugging

```yaml
# Activer le debug
env:
  ACTIONS_STEP_DEBUG: true

# Afficher les contextes
- name: Debug
  run: |
    echo "Event: ${{ github.event_name }}"
    echo "Ref: ${{ github.ref }}"
    echo "SHA: ${{ github.sha }}"
```

## Bonnes pratiques

1. **Utiliser des versions fixes** pour les actions : `@v4` plutôt que `@main`
2. **Cache les dépendances** pour accélérer les builds
3. **Utiliser des secrets** pour les données sensibles
4. **Limiter les permissions** au minimum nécessaire
5. **Utiliser concurrency** pour éviter les runs parallèles inutiles
6. **Séparer les jobs** pour paralléliser
7. **Utiliser matrix** pour tester plusieurs versions
8. **Documenter** les workflows avec des commentaires

---

*Formation CI/CD - ForEach Academy*
