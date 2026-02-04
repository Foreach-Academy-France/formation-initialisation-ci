# TP Jour 5 : Déploiement complet

> **Durée** : 2h30 | **Objectif** : Pipeline CD avec Docker, ghcr.io et GitHub Pages

## Point de départ

Vous devez avoir complété le TP Jour 4 avec :
- ✅ Branch protection active
- ✅ Release v1.0.0 publiée
- ✅ Workflow release fonctionnel

---

## Étape 1 : Créer le Dockerfile (25 min)

### 1.1 Créer le Dockerfile

Créer `Dockerfile` à la racine du projet :

```dockerfile
# ===========================================
# Stage 1: Build
# ===========================================
FROM node:20-alpine AS build

# Définir le répertoire de travail
WORKDIR /app

# Copier les fichiers de dépendances
COPY package*.json ./

# Installer les dépendances
RUN npm ci

# Copier le code source
COPY . .

# Builder l'application
RUN npm run build

# ===========================================
# Stage 2: Production
# ===========================================
FROM nginx:alpine

# Copier la configuration nginx personnalisée
COPY nginx.conf /etc/nginx/conf.d/default.conf

# Copier les fichiers buildés depuis le stage précédent
COPY --from=build /app/dist /usr/share/nginx/html

# Exposer le port 80
EXPOSE 80

# Commande de démarrage
CMD ["nginx", "-g", "daemon off;"]
```

### 1.2 Créer la configuration nginx

Créer `nginx.conf` :

```nginx
server {
    listen 80;
    server_name localhost;
    root /usr/share/nginx/html;
    index index.html;

    # Gestion du routing SPA
    location / {
        try_files $uri $uri/ /index.html;
    }

    # Cache des assets statiques
    location ~* \.(js|css|png|jpg|jpeg|gif|ico|svg)$ {
        expires 1y;
        add_header Cache-Control "public, immutable";
    }

    # Désactiver le cache pour index.html
    location = /index.html {
        add_header Cache-Control "no-cache, no-store, must-revalidate";
    }

    # Gzip compression
    gzip on;
    gzip_types text/plain text/css application/json application/javascript text/xml application/xml;
}
```

### 1.3 Créer le .dockerignore

Créer `.dockerignore` :

```
node_modules
dist
coverage
.git
.github
*.md
.env*
.vscode
```

### 1.4 Tester le build Docker localement

```bash
# Builder l'image
docker build -t taskflow:local .

# Lancer le conteneur
docker run -d -p 8080:80 --name taskflow-test taskflow:local

# Vérifier
open http://localhost:8080   # ou xdg-open sur Linux

# Nettoyer
docker stop taskflow-test && docker rm taskflow-test
```

### 1.5 Commit

```bash
git checkout -b feature/docker
git add Dockerfile nginx.conf .dockerignore
git commit -m "feat: add Docker support with nginx"
git push -u origin feature/docker
gh pr create --title "feat: add Docker support" --body "Add Dockerfile with multi-stage build and nginx configuration"
```

---

## Étape 2 : Workflow Docker (30 min)

### 2.1 Créer le workflow docker

Créer `.github/workflows/docker.yml` :

```yaml
name: Docker

on:
  push:
    branches: [main]
    tags: ['v*']
  pull_request:
    branches: [main]

permissions:
  contents: read
  packages: write

env:
  REGISTRY: ghcr.io
  IMAGE_NAME: ${{ github.repository }}

jobs:
  build-and-push:
    name: Build and Push Docker Image
    runs-on: ubuntu-latest
    steps:
      - name: Checkout
        uses: actions/checkout@v4

      - name: Setup Docker Buildx
        uses: docker/setup-buildx-action@v3

      - name: Login to GitHub Container Registry
        if: github.event_name != 'pull_request'
        uses: docker/login-action@v3
        with:
          registry: ${{ env.REGISTRY }}
          username: ${{ github.actor }}
          password: ${{ secrets.GITHUB_TOKEN }}

      - name: Extract metadata
        id: meta
        uses: docker/metadata-action@v5
        with:
          images: ${{ env.REGISTRY }}/${{ env.IMAGE_NAME }}
          tags: |
            type=ref,event=branch
            type=ref,event=pr
            type=semver,pattern={{version}}
            type=semver,pattern={{major}}.{{minor}}
            type=sha,prefix=sha-

      - name: Build and push
        uses: docker/build-push-action@v5
        with:
          context: .
          push: ${{ github.event_name != 'pull_request' }}
          tags: ${{ steps.meta.outputs.tags }}
          labels: ${{ steps.meta.outputs.labels }}
          cache-from: type=gha
          cache-to: type=gha,mode=max
```

### 2.2 Comprendre les tags générés

| Événement | Tags générés |
|-----------|--------------|
| Push sur `main` | `ghcr.io/user/repo:main`, `ghcr.io/user/repo:sha-abc123` |
| Tag `v1.2.3` | `ghcr.io/user/repo:1.2.3`, `ghcr.io/user/repo:1.2`, `ghcr.io/user/repo:sha-def456` |
| Pull Request | Build sans push (test only) |

### 2.3 Commit

```bash
git add .github/workflows/docker.yml
git commit -m "ci: add Docker build and push workflow"
git push origin feature/docker
```

### 2.4 Merger la PR

Attendre que le CI passe, puis :
```bash
gh pr merge --squash --delete-branch
```

### 2.5 Vérifier l'image

1. Aller dans **Packages** de votre repository
2. L'image `taskflow-starter` doit apparaître
3. Cliquer pour voir les tags disponibles

---

## Étape 3 : GitHub Pages (30 min)

### 3.1 Configurer Vite pour GitHub Pages

Modifier `vite.config.js` :

```javascript
import { defineConfig } from 'vite';

export default defineConfig({
  base: '/taskflow-starter/',  // Nom de votre repo !
  build: {
    outDir: 'dist',
    sourcemap: true,
  },
});
```

### 3.2 Créer le workflow Pages

Créer `.github/workflows/pages.yml` :

```yaml
name: Deploy to GitHub Pages

on:
  push:
    branches: [main]
  workflow_dispatch:

permissions:
  contents: read
  pages: write
  id-token: write

concurrency:
  group: "pages"
  cancel-in-progress: false

jobs:
  build:
    name: Build
    runs-on: ubuntu-latest
    steps:
      - name: Checkout
        uses: actions/checkout@v4

      - name: Setup Node.js
        uses: actions/setup-node@v4
        with:
          node-version: '20'
          cache: 'npm'

      - name: Install dependencies
        run: npm ci

      - name: Run tests
        run: npm test

      - name: Build
        run: npm run build

      - name: Setup Pages
        uses: actions/configure-pages@v4

      - name: Upload artifact
        uses: actions/upload-pages-artifact@v3
        with:
          path: ./dist

  deploy:
    name: Deploy
    environment:
      name: github-pages
      url: ${{ steps.deployment.outputs.page_url }}
    runs-on: ubuntu-latest
    needs: build
    steps:
      - name: Deploy to GitHub Pages
        id: deployment
        uses: actions/deploy-pages@v4
```

### 3.3 Activer GitHub Pages

1. Aller dans **Settings > Pages**
2. Source : **GitHub Actions**

### 3.4 Commit et déployer

```bash
git checkout -b feature/pages
git add vite.config.js .github/workflows/pages.yml
git commit -m "ci: add GitHub Pages deployment"
git push -u origin feature/pages
gh pr create --title "ci: add GitHub Pages deployment" --body "Deploy application to GitHub Pages"
gh pr merge --squash --delete-branch
```

### 3.5 Vérifier le déploiement

1. Aller dans **Actions** : le workflow "Deploy to GitHub Pages" doit s'exécuter
2. Une fois terminé, aller sur : `https://VOTRE-USER.github.io/taskflow-starter/`

---

## Étape 4 : Pipeline unifié (20 min)

### 4.1 Consolider les workflows

Vous avez maintenant plusieurs workflows :
- `ci.yml` : Lint, Test, Build
- `release.yml` : Release sur tag
- `docker.yml` : Build/push Docker image
- `pages.yml` : Deploy sur GitHub Pages

### 4.2 Option : Workflow unifié

Pour simplifier, vous pouvez créer un workflow unique. Créer `.github/workflows/ci-cd.yml` :

```yaml
name: CI/CD

on:
  push:
    branches: [main]
    tags: ['v*']
  pull_request:
    branches: [main]

permissions:
  contents: write
  packages: write
  pages: write
  id-token: write

concurrency:
  group: ${{ github.workflow }}-${{ github.ref }}
  cancel-in-progress: true

env:
  REGISTRY: ghcr.io
  IMAGE_NAME: ${{ github.repository }}

jobs:
  # ========== CI ==========
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

  test:
    name: Test
    runs-on: ubuntu-latest
    needs: lint
    steps:
      - uses: actions/checkout@v4
      - uses: actions/setup-node@v4
        with:
          node-version: '20'
          cache: 'npm'
      - run: npm ci
      - run: npm run test:coverage
      - uses: actions/upload-artifact@v4
        with:
          name: coverage
          path: coverage/

  build:
    name: Build
    runs-on: ubuntu-latest
    needs: test
    steps:
      - uses: actions/checkout@v4
      - uses: actions/setup-node@v4
        with:
          node-version: '20'
          cache: 'npm'
      - run: npm ci
      - run: npm run build
      - uses: actions/upload-artifact@v4
        with:
          name: dist
          path: dist/

  # ========== CD: Docker ==========
  docker:
    name: Docker
    runs-on: ubuntu-latest
    needs: build
    if: github.event_name != 'pull_request'
    steps:
      - uses: actions/checkout@v4
      - uses: docker/setup-buildx-action@v3
      - uses: docker/login-action@v3
        with:
          registry: ${{ env.REGISTRY }}
          username: ${{ github.actor }}
          password: ${{ secrets.GITHUB_TOKEN }}
      - id: meta
        uses: docker/metadata-action@v5
        with:
          images: ${{ env.REGISTRY }}/${{ env.IMAGE_NAME }}
          tags: |
            type=ref,event=branch
            type=semver,pattern={{version}}
            type=sha,prefix=sha-
      - uses: docker/build-push-action@v5
        with:
          context: .
          push: true
          tags: ${{ steps.meta.outputs.tags }}
          labels: ${{ steps.meta.outputs.labels }}

  # ========== CD: GitHub Pages ==========
  pages:
    name: Deploy Pages
    runs-on: ubuntu-latest
    needs: build
    if: github.ref == 'refs/heads/main'
    environment:
      name: github-pages
      url: ${{ steps.deployment.outputs.page_url }}
    steps:
      - uses: actions/download-artifact@v4
        with:
          name: dist
          path: dist/
      - uses: actions/configure-pages@v4
      - uses: actions/upload-pages-artifact@v3
        with:
          path: dist/
      - id: deployment
        uses: actions/deploy-pages@v4

  # ========== CD: Release ==========
  release:
    name: Create Release
    runs-on: ubuntu-latest
    needs: [docker, pages]
    if: startsWith(github.ref, 'refs/tags/v')
    steps:
      - uses: actions/checkout@v4
        with:
          fetch-depth: 0
      - uses: actions/download-artifact@v4
        with:
          name: dist
          path: dist/
      - uses: softprops/action-gh-release@v2
        with:
          generate_release_notes: true
          files: dist/*
```

### 4.3 Supprimer les anciens workflows (optionnel)

Si vous utilisez le workflow unifié :
```bash
rm .github/workflows/ci.yml
rm .github/workflows/docker.yml
rm .github/workflows/pages.yml
rm .github/workflows/release.yml
```

---

## Étape 5 : Release v1.1.0 (15 min)

### 5.1 Mettre à jour la version

```bash
git checkout main
git pull origin main
git checkout -b release/v1.1.0
npm version 1.1.0 --no-git-tag-version
git add package.json
git commit -m "chore: bump version to 1.1.0"
git push -u origin release/v1.1.0
```

### 5.2 Créer et merger la PR

```bash
gh pr create \
  --title "chore: release v1.1.0" \
  --body "## Release v1.1.0

### New Features
- Docker support with nginx
- GitHub Pages deployment
- Unified CI/CD pipeline

### Improvements
- Filter tasks by status
- Better test coverage"
```

```bash
gh pr merge --squash --delete-branch
```

### 5.3 Créer le tag

```bash
git checkout main
git pull origin main
git tag -a v1.1.0 -m "Release v1.1.0 - Docker and GitHub Pages"
git push origin v1.1.0
```

### 5.4 Vérifier

1. **Actions** : Le workflow complet s'exécute
2. **Packages** : Nouvelle image avec tag `1.1.0`
3. **GitHub Pages** : Site mis à jour
4. **Releases** : v1.1.0 publiée

---

## Étape 6 : Finalisation (10 min)

### 6.1 Mettre à jour le README final

```markdown
# TaskFlow

[![CI/CD](https://github.com/VOTRE-USER/taskflow-starter/actions/workflows/ci-cd.yml/badge.svg)](https://github.com/VOTRE-USER/taskflow-starter/actions)
[![Release](https://img.shields.io/github/v/release/VOTRE-USER/taskflow-starter)](https://github.com/VOTRE-USER/taskflow-starter/releases)
[![Coverage](https://img.shields.io/badge/coverage-%3E70%25-brightgreen)](./coverage/)

> Application de gestion de tâches avec CI/CD complet

## 🚀 Demo

**Live** : https://VOTRE-USER.github.io/taskflow-starter/

## 🐳 Docker

```bash
docker pull ghcr.io/VOTRE-USER/taskflow-starter:latest
docker run -d -p 8080:80 ghcr.io/VOTRE-USER/taskflow-starter:latest
```

## 🛠️ Development

```bash
npm install
npm run dev
npm test
npm run build
```

## 📦 CI/CD Pipeline

- ✅ Lint (ESLint)
- ✅ Test (Vitest + Coverage ≥ 70%)
- ✅ Build (Vite)
- ✅ Docker (ghcr.io)
- ✅ Deploy (GitHub Pages)
- ✅ Release (auto sur tag)
```

### 6.2 Commit final

```bash
git checkout -b docs/final-readme
git add README.md
git commit -m "docs: update README with complete documentation"
git push -u origin docs/final-readme
gh pr create --title "docs: final README update" --body "Complete documentation with all features"
gh pr merge --squash --delete-branch
```

---

## Checklist de validation finale

Votre projet TaskFlow doit avoir :

### CI/CD Pipeline
- [ ] Lint avec ESLint (matrix Node 18/20/22)
- [ ] Tests avec Vitest (coverage ≥ 70%)
- [ ] Build avec Vite
- [ ] Artifact coverage uploadé
- [ ] Artifact dist uploadé

### Docker
- [ ] Dockerfile multi-stage
- [ ] nginx.conf configuré
- [ ] .dockerignore présent
- [ ] Image sur ghcr.io (tags: main, vX.X.X, sha-xxx)

### GitHub Pages
- [ ] Site déployé et accessible
- [ ] Configuration Vite avec `base` correct

### Releases
- [ ] Branch protection sur main
- [ ] Au moins 2 releases (v1.0.0, v1.1.0)
- [ ] Tags poussés
- [ ] Release notes générées

### Documentation
- [ ] README avec badges
- [ ] Instructions Docker
- [ ] Lien vers le site

---

## Grille d'évaluation

| Critère | Points | Vérifié par |
|---------|--------|-------------|
| Workflow CI passe | 20 | Badge vert ✅ |
| Tests + coverage ≥ 70% | 20 | Artifact coverage |
| Branch protection | 15 | Settings > Branches |
| Au moins 1 release | 15 | Releases GitHub |
| Image Docker sur ghcr.io | 15 | Packages |
| Site sur GitHub Pages | 15 | URL accessible |
| **TOTAL** | **100** | |

**Validation** : ≥ 50 points

---

## 🎉 Félicitations !

Vous avez complété le projet fil rouge TaskFlow avec un pipeline CI/CD complet !

### Ce que vous avez appris

- ✅ GitHub Actions workflows
- ✅ Jobs, matrix, triggers
- ✅ Tests automatisés avec Vitest
- ✅ Coverage et qualité de code
- ✅ Branch protection et PRs
- ✅ SemVer et releases
- ✅ Docker multi-stage builds
- ✅ GitHub Container Registry
- ✅ GitHub Pages deployment

---

## Ressources finales

- [GitHub Actions Documentation](https://docs.github.com/en/actions)
- [Docker Documentation](https://docs.docker.com/)
- [Vitest Documentation](https://vitest.dev/)
- [GitHub Packages](https://docs.github.com/en/packages)

---

**Formation CI/CD - ForEach Academy**
