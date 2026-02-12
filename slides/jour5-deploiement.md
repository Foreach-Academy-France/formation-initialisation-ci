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


<!-- _class: lead -->

# 🚀 Jour 5
## Déploiement continu

**Formation CI/CD - GitHub Actions**
M2 ESTD - Architecte Web
ForEach Academy

---

## 👋 Rappel Jour 4

**Ce qu'on a appris** :
- ✅ Stratégies de branches
- ✅ Branch protection et PR
- ✅ SemVer et releases automatiques

**Aujourd'hui (Jour 5)** :
1. CD : Delivery vs Deployment
2. Docker et ghcr.io
3. GitHub Pages
4. Pipeline complet + QCM

---

# Programme du jour

| Horaire | Contenu |
|---------|---------|
| 9h00-10h30 | **Stratégies de déploiement** |
| 10h45-12h15 | **Docker et Container Registry** |
| 13h15-15h30 | **TP : Pipeline CD complet** |
| 15h45-16h30 | **Bonnes pratiques et amélioration** |
| 16h30-17h00 | **QCM Final** |

---

# Objectifs du jour

À la fin de cette journée, vous serez capables de :

- ✅ Distinguer Delivery et Deployment
- ✅ Builder et pusher des images Docker
- ✅ Déployer sur GitHub Pages
- ✅ Créer un pipeline CD complet
- ✅ Appliquer les bonnes pratiques

---

<!-- _class: lead -->

# Partie 1
## Stratégies CD

---

# Rappel : CI vs CD

```
┌─────────────────────────────────────────────────────────────────┐
│                                                                 │
│  CI (Continuous Integration)                                    │
│  ────────────────────────────                                   │
│  Code → Build → Test → "Le code est-il correct ?"               │
│                                                                 │
│  CD (Continuous Delivery)                                       │
│  ────────────────────────                                       │
│  ... → Package → Stage → "Le code est-il livrable ?"            │
│                          (déploiement manuel)                   │
│                                                                 │
│  CD (Continuous Deployment)                                     │
│  ─────────────────────────                                      │
│  ... → Deploy → Production (tout automatique)                   │
│                                                                 │
└─────────────────────────────────────────────────────────────────┘
```

---

# Delivery vs Deployment

<div class="columns">
<div>

## Continuous Delivery

- Artefact prêt à déployer
- **Bouton manuel** pour prod
- Contrôle humain
- Adapté aux réglementations

</div>
<div>

## Continuous Deployment

- **Tout automatique**
- Chaque commit va en prod
- Nécessite CI robuste
- Feature flags recommandés

</div>
</div>

---

# Environnements

```
┌─────────────────────────────────────────────────────────────────┐
│                                                                 │
│  Development → Staging → Production                             │
│       │            │           │                                │
│   localhost    preview.app   app.com                            │
│                                                                 │
│  ┌─────────┐  ┌─────────┐  ┌─────────┐                         │
│  │  Dev    │→ │ Staging │→ │  Prod   │                         │
│  │  Auto   │  │  Auto   │  │ Manual? │                         │
│  └─────────┘  └─────────┘  └─────────┘                         │
│                                                                 │
└─────────────────────────────────────────────────────────────────┘
```

---

# GitHub Environments

```yaml
jobs:
  deploy-staging:
    environment: staging
    runs-on: ubuntu-latest
    steps:
      - run: ./deploy.sh staging

  deploy-production:
    environment:
      name: production
      url: https://myapp.com
    needs: deploy-staging
    runs-on: ubuntu-latest
    steps:
      - run: ./deploy.sh production
```

---

# Protection des environnements

## Settings > Environments > production

- ✅ **Required reviewers** : Approbation manuelle
- ✅ **Wait timer** : Délai (ex: 30 min)
- ✅ **Deployment branches** : Seulement `main`

```yaml
environment:
  name: production
# → Attend l'approbation d'un reviewer
```

---

# Stratégies de déploiement

| Stratégie | Description | Risque |
|-----------|-------------|--------|
| **Recreate** | Stop ancien → Start nouveau | Downtime |
| **Rolling** | Remplacement progressif | Faible |
| **Blue/Green** | 2 environnements, switch | Très faible |
| **Canary** | % users sur nouvelle version | Minimal |

---

# Blue/Green Deployment

```
       Avant                        Après
┌─────────────────┐          ┌─────────────────┐
│   Load Balancer │          │   Load Balancer │
└────────┬────────┘          └────────┬────────┘
         │                            │
    ┌────┴────┐                  ┌────┴────┐
    ▼         ▼                  ▼         ▼
┌──────┐  ┌──────┐          ┌──────┐  ┌──────┐
│ Blue │  │Green │          │ Blue │  │Green │
│ v1.0 │  │(idle)│    →     │(idle)│  │ v2.0 │
│ ✅   │  │      │          │      │  │ ✅   │
└──────┘  └──────┘          └──────┘  └──────┘
```

**Rollback** : Re-switch vers Blue

---

# Canary Deployment

```
        Étape 1              Étape 2              Étape 3
     ┌──────────┐         ┌──────────┐         ┌──────────┐
     │   95%    │         │   70%    │         │   0%     │
     │   v1.0   │         │   v1.0   │         │   v1.0   │
     └──────────┘         └──────────┘         └──────────┘
     ┌──────────┐         ┌──────────┐         ┌──────────┐
     │    5%    │         │   30%    │         │  100%    │
     │   v2.0   │         │   v2.0   │         │   v2.0   │
     └──────────┘         └──────────┘         └──────────┘

     Test sur 5%      Si OK, augmenter      Déploiement complet
```

---

# Feature Flags

```javascript
// Déployer du code "inactif"
if (featureFlags.newCheckout) {
  return <NewCheckout />;
}
return <OldCheckout />;
```

**Avantages** :
- Déployer sans activer
- A/B testing
- Rollback instantané (toggle off)
- Activation progressive

---

<!-- _class: lead -->

# Partie 2
## Docker et Registry

---

# Pourquoi Docker en CD ?

## Avantages

- 📦 **Packaging** : App + dépendances
- 🔄 **Reproductibilité** : Même image partout
- 🚀 **Déploiement** : Pull and run
- ↩️ **Rollback** : Version précédente

```bash
# Déploiement = 1 commande
docker pull ghcr.io/user/app:v1.2.0
docker run -d ghcr.io/user/app:v1.2.0
```

---

# Dockerfile pour Vite

```dockerfile
# Build stage
FROM node:20-alpine AS build
WORKDIR /app
COPY package*.json ./
RUN npm ci
COPY . .
RUN npm run build

# Production stage
FROM nginx:alpine
COPY --from=build /app/dist /usr/share/nginx/html
EXPOSE 80
CMD ["nginx", "-g", "daemon off;"]
```

---

# Multi-stage build

```
┌─────────────────────────────────────────────────────────────────┐
│  Stage 1: Build                                                 │
│  ┌─────────────────────────────────────────────────────────┐   │
│  │  Node.js 20                                              │   │
│  │  npm install                                             │   │
│  │  npm run build → dist/                                   │   │
│  └─────────────────────────────────────────────────────────┘   │
│                            │                                    │
│                            ▼                                    │
│  Stage 2: Production                                            │
│  ┌─────────────────────────────────────────────────────────┐   │
│  │  Nginx Alpine (~20MB)                                    │   │
│  │  COPY dist/ seulement                                    │   │
│  └─────────────────────────────────────────────────────────┘   │
└─────────────────────────────────────────────────────────────────┘

Résultat: Image légère sans Node.js ni node_modules
```

---

# Container Registries

| Registry | URL | Usage |
|----------|-----|-------|
| Docker Hub | docker.io | Public, images officielles |
| **ghcr.io** | ghcr.io | GitHub, intégré Actions |
| AWS ECR | *.ecr.*.amazonaws.com | AWS |
| GCP GCR | gcr.io | Google Cloud |

**Pour ce cours** : GitHub Container Registry (ghcr.io)

---

# GitHub Container Registry

```
ghcr.io/OWNER/IMAGE:TAG

Exemples:
ghcr.io/foreach-academy/taskflow:latest
ghcr.io/foreach-academy/taskflow:v1.0.0
ghcr.io/foreach-academy/taskflow:sha-abc123
```

**Avantages** :
- Intégré à GitHub
- Permissions via GITHUB_TOKEN
- Lié au repository
- Gratuit pour repos publics

---

# Build et push avec Actions

```yaml
- name: Login to GitHub Container Registry
  uses: docker/login-action@v3
  with:
    registry: ghcr.io
    username: ${{ github.actor }}
    password: ${{ secrets.GITHUB_TOKEN }}

- name: Build and push
  uses: docker/build-push-action@v5
  with:
    push: true
    tags: ghcr.io/${{ github.repository }}:latest
```

---

# Tags d'image intelligents

```yaml
- name: Docker meta
  id: meta
  uses: docker/metadata-action@v5
  with:
    images: ghcr.io/${{ github.repository }}
    tags: |
      type=ref,event=branch
      type=semver,pattern={{version}}
      type=sha,prefix=sha-

- name: Build and push
  uses: docker/build-push-action@v5
  with:
    push: true
    tags: ${{ steps.meta.outputs.tags }}
```

---

# Résultat des tags

```bash
# Sur push branche main
ghcr.io/user/app:main
ghcr.io/user/app:sha-abc1234

# Sur tag v1.2.3
ghcr.io/user/app:1.2.3
ghcr.io/user/app:sha-def5678

# Toujours
ghcr.io/user/app:latest  # (optionnel)
```

---

# Workflow Docker complet

```yaml
name: Docker

on:
  push:
    branches: [main]
    tags: ['v*']

permissions:
  contents: read
  packages: write

jobs:
  build:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4

      - uses: docker/login-action@v3
        with:
          registry: ghcr.io
          username: ${{ github.actor }}
          password: ${{ secrets.GITHUB_TOKEN }}
```

---

# Workflow Docker (suite)

```yaml
      - name: Docker meta
        id: meta
        uses: docker/metadata-action@v5
        with:
          images: ghcr.io/${{ github.repository }}
          tags: |
            type=ref,event=branch
            type=semver,pattern={{version}}

      - name: Build and push
        uses: docker/build-push-action@v5
        with:
          context: .
          push: true
          tags: ${{ steps.meta.outputs.tags }}
          labels: ${{ steps.meta.outputs.labels }}
```

---

<!-- _class: lead -->

# Partie 3
## GitHub Pages

---

# GitHub Pages

> Hébergement gratuit de sites statiques

**Types de contenu** :
- Documentation
- Sites statiques (Vite, React, Vue)
- Slides (Marp)
- Landing pages

**URL** : `https://<user>.github.io/<repo>/`

---

# Déploiement Pages - Méthode 1

## Deploy from branch

```yaml
# Build et commit sur gh-pages
- name: Deploy to GitHub Pages
  uses: peaceiris/actions-gh-pages@v4
  with:
    github_token: ${{ secrets.GITHUB_TOKEN }}
    publish_dir: ./dist
```

**Inconvénient** : Pollue l'historique

---

# Déploiement Pages - Méthode 2

## GitHub Actions (recommandé)

```yaml
permissions:
  contents: read
  pages: write
  id-token: write

jobs:
  deploy:
    environment:
      name: github-pages
      url: ${{ steps.deployment.outputs.page_url }}
    steps:
      - uses: actions/configure-pages@v4
      - uses: actions/upload-pages-artifact@v3
        with:
          path: './dist'
      - uses: actions/deploy-pages@v4
```

---

# Workflow Pages complet

```yaml
name: Deploy to Pages

on:
  push:
    branches: [main]

permissions:
  contents: read
  pages: write
  id-token: write

jobs:
  build:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      - uses: actions/setup-node@v4
        with:
          node-version: '20'
          cache: 'npm'
      - run: npm ci
      - run: npm run build
```

---

# Workflow Pages (suite)

```yaml
      - name: Setup Pages
        uses: actions/configure-pages@v4

      - name: Upload artifact
        uses: actions/upload-pages-artifact@v3
        with:
          path: './dist'

  deploy:
    needs: build
    runs-on: ubuntu-latest
    environment:
      name: github-pages
      url: ${{ steps.deployment.outputs.page_url }}
    steps:
      - name: Deploy to GitHub Pages
        id: deployment
        uses: actions/deploy-pages@v4
```

---

# Configuration Vite pour Pages

```javascript
// vite.config.js
export default defineConfig({
  base: '/taskflow/',  // Nom du repo!
  // ...
});
```

**Important** : Le `base` doit correspondre au nom du repo

---

<!-- _class: lead -->

# Partie 4
## Pipeline CD complet

---

# Architecture cible TaskFlow

![Pipeline CD](https://mermaid.ink/img/Zmxvd2NoYXJ0IFRCCiAgICBQW1B1c2ggbWFpbl0gLS0+IExbTGludF0gLS0+IFRbVGVzdF0gLS0+IEJbQnVpbGRdCiAgICBCIC0tPiBEWyJEb2NrZXIgZ2hjci5pbyJdCiAgICBCIC0tPiBHWyJHaXRIdWIgUGFnZXMiXQ==)

---

# TP5 : Pipeline CD TaskFlow

## Objectifs

1. CI : Lint + Test + Build
2. Docker : Build et push vers ghcr.io
3. Pages : Déployer l'application
4. Vérifier le déploiement

---

# Étape 1 : Dockerfile

```dockerfile
# Dockerfile
FROM node:20-alpine AS build
WORKDIR /app
COPY package*.json ./
RUN npm ci
COPY . .
RUN npm run build

FROM nginx:alpine
COPY --from=build /app/dist /usr/share/nginx/html
COPY nginx.conf /etc/nginx/conf.d/default.conf
EXPOSE 80
```

---

# Étape 2 : Workflow CI/CD

```yaml
name: CI/CD

on:
  push:
    branches: [main]
    tags: ['v*']
  pull_request:
    branches: [main]

permissions:
  contents: read
  packages: write
  pages: write
  id-token: write
```

---

# Étape 3 : Jobs CI

```yaml
jobs:
  lint:
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
    runs-on: ubuntu-latest
    needs: lint
    steps:
      - uses: actions/checkout@v4
      - uses: actions/setup-node@v4
      - run: npm ci
      - run: npm run test:coverage
```

---

# Étape 4 : Job Build

```yaml
  build:
    runs-on: ubuntu-latest
    needs: test
    steps:
      - uses: actions/checkout@v4
      - uses: actions/setup-node@v4
      - run: npm ci
      - run: npm run build

      - uses: actions/upload-artifact@v4
        with:
          name: dist
          path: dist/
```

---

# Étape 5 : Job Docker

```yaml
  docker:
    runs-on: ubuntu-latest
    needs: build
    if: github.ref == 'refs/heads/main' || startsWith(github.ref, 'refs/tags/')
    steps:
      - uses: actions/checkout@v4
      - uses: docker/login-action@v3
        with:
          registry: ghcr.io
          username: ${{ github.actor }}
          password: ${{ secrets.GITHUB_TOKEN }}
      - uses: docker/metadata-action@v5
        id: meta
        with:
          images: ghcr.io/${{ github.repository }}
      - uses: docker/build-push-action@v5
        with:
          push: true
          tags: ${{ steps.meta.outputs.tags }}
```

---

# Étape 6 : Job Pages

```yaml
  pages:
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
      - uses: actions/deploy-pages@v4
        id: deployment
```

---

# Résultat final

```
Push sur main:

lint ──→ test ──→ build ──┬──→ docker (ghcr.io)
                          │
                          └──→ pages (GitHub Pages)

Tag v1.0.0:

lint ──→ test ──→ build ──→ docker (tag: v1.0.0)
```

---

<!-- _class: lead -->

# Partie 5
## Bonnes pratiques

---

# Optimisation des workflows

## 1. Cache des dépendances

```yaml
- uses: actions/setup-node@v4
  with:
    cache: 'npm'
```

## 2. Réutilisation des artifacts

```yaml
- uses: actions/upload-artifact@v4
- uses: actions/download-artifact@v4
```

---

# Optimisation (suite)

## 3. Conditions pour éviter les builds inutiles

```yaml
on:
  push:
    paths-ignore:
      - '**.md'
      - 'docs/**'
```

## 4. Concurrency pour annuler les builds obsolètes

```yaml
concurrency:
  group: ${{ github.workflow }}-${{ github.ref }}
  cancel-in-progress: true
```

---

# Sécurité

## 1. Permissions minimales

```yaml
permissions:
  contents: read  # Seulement ce qui est nécessaire
```

## 2. Pin des versions d'actions

```yaml
uses: actions/checkout@v4.1.1  # Version spécifique
# ou
uses: actions/checkout@abc123  # SHA du commit
```

---

# Sécurité (suite)

## 3. Secrets rotatifs

- Changer les secrets régulièrement
- Utiliser des tokens à durée limitée

## 4. Review des dépendances

```yaml
- uses: github/codeql-action/analyze@v3
```

---

# Monitoring

## 1. Notifications

```yaml
- name: Notify on failure
  if: failure()
  uses: slackapi/slack-github-action@v1
  with:
    channel-id: 'deployments'
    slack-message: 'Deploy failed! ${{ github.server_url }}/${{ github.repository }}/actions/runs/${{ github.run_id }}'
```

## 2. Métriques de build

- Durée moyenne des builds
- Taux de succès
- Temps de déploiement

---

# Récapitulatif Semaine

| Jour | Thème | Compétences |
|------|-------|-------------|
| 1 | Introduction CI | Pipeline, jobs, premier workflow |
| 2 | Configuration | Variables, matrix, triggers |
| 3 | Tests | Vitest, coverage, artifacts |
| 4 | Branches | Git Flow, SemVer, releases |
| 5 | Déploiement | Docker, ghcr.io, Pages |

---

# Votre pipeline TaskFlow

```
┌───────────────────────────────────────────────────────────────┐
│  GitHub Actions Pipeline - TaskFlow                           │
├───────────────────────────────────────────────────────────────┤
│  ✅ Lint (ESLint)                                             │
│  ✅ Test (Vitest) + Coverage ≥ 70%                            │
│  ✅ Build (Vite)                                              │
│  ✅ Docker (ghcr.io)                                          │
│  ✅ Deploy (GitHub Pages)                                     │
│  ✅ Release (automatique sur tag)                             │
├───────────────────────────────────────────────────────────────┤
│  🔒 Branch protection                                         │
│  📋 PR required avec reviews                                  │
│  🏷️ SemVer avec Conventional Commits                         │
└───────────────────────────────────────────────────────────────┘
```

---

<!-- _class: lead -->

# QCM Final
## 30 questions - 45 minutes

---

# Avant le QCM

## Vérifiez que vous savez :

- ✅ Écrire un workflow GitHub Actions
- ✅ Configurer des jobs avec dépendances
- ✅ Utiliser secrets et variables
- ✅ Configurer matrix builds
- ✅ Écrire des tests avec Vitest
- ✅ Comprendre SemVer
- ✅ Builder une image Docker
- ✅ Déployer sur GitHub Pages

---

<!-- _class: lead -->

# Questions ?

---

# Ressources finales

## Documentation
- [GitHub Actions](https://docs.github.com/en/actions)
- [Docker](https://docs.docker.com/)
- [Vitest](https://vitest.dev/)

## Votre projet
- TaskFlow sur GitHub Pages
- Image Docker sur ghcr.io
- Pipeline CI/CD complet

---

<!-- _class: lead -->

# 🎉 Félicitations !

Vous avez complété la formation
**Initialisation CI avec GitHub Actions**

---

# Certificat de compétences

## Vous maîtrisez maintenant :

- 🔄 Intégration Continue (CI)
- 🧪 Tests automatisés
- 📦 Packaging Docker
- 🚀 Déploiement Continu (CD)
- 🏷️ Gestion des versions

**Compétences validées** : C30, C33, C34, C35

---

<!-- _class: lead -->

# Merci !

**Formation CI/CD - ForEach Academy**

Fabrice Claeys
fabrice.claeys@groupe-bao.fr
