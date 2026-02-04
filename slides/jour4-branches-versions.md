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

# 🌿 Jour 4
## Branches et versions

**Formation CI/CD - GitHub Actions**
M2 ESTD - Architecte Web
ForEach Academy

---

## 👋 Rappel Jour 3

**Ce qu'on a appris** :
- ✅ Pyramide des tests
- ✅ Vitest et coverage
- ✅ Artifacts et rapports CI

**Aujourd'hui (Jour 4)** :
1. Stratégies de branches
2. Pull requests et code review
3. SemVer et tags
4. Releases automatiques

---

# Programme du jour

| Horaire | Contenu |
|---------|---------|
| 9h00-10h30 | **Stratégies de branches** |
| 10h45-12h15 | **CI et Pull Requests** |
| 13h15-15h00 | **Versions et releases** |
| 15h15-17h00 | **TP : Branch protection et release** |

---

# Objectifs du jour

À la fin de cette journée, vous serez capables de :

- ✅ Choisir une stratégie de branches
- ✅ Configurer les branch protection rules
- ✅ Comprendre et appliquer SemVer
- ✅ Automatiser les releases avec tags
- ✅ Générer un changelog automatique

---

<!-- _class: lead -->

# Partie 1
## Stratégies de branches

---

# Pourquoi des stratégies ?

## Sans stratégie

- 😵 Commits directs sur main
- 🔀 Branches abandonnées
- 💥 Conflits constants
- ❓ "C'est quoi cette branche ?"

## Avec stratégie

- ✅ Workflow clair pour tous
- ✅ Code review systématique
- ✅ Historique propre
- ✅ Releases prévisibles

---

# Git Flow

```
main        ●────────────●────────────●────────────●
             \          /            /            /
release       \    ●───●            /            /
               \  /    \           /            /
develop    ●────●───────●─────────●────────────●
            \          / \       /            /
feature      ●────────●   ●─────●            /
                           \                /
hotfix                      ●──────────────●
```

**Branches** : main, develop, feature/*, release/*, hotfix/*

---

# Git Flow : Branches

| Branche | Durée | Source | Merge vers |
|---------|-------|--------|------------|
| `main` | Permanente | - | - |
| `develop` | Permanente | main | - |
| `feature/*` | Temporaire | develop | develop |
| `release/*` | Temporaire | develop | main + develop |
| `hotfix/*` | Temporaire | main | main + develop |

---

# Git Flow : Workflow

```bash
# Nouvelle feature
git checkout develop
git checkout -b feature/login
# ... développement ...
git checkout develop
git merge feature/login

# Release
git checkout develop
git checkout -b release/1.0.0
# ... stabilisation ...
git checkout main
git merge release/1.0.0
git tag v1.0.0
git checkout develop
git merge release/1.0.0
```

---

# Git Flow : Avantages / Inconvénients

<div class="columns">
<div>

## ✅ Avantages

- Releases bien définies
- Hotfixes isolés
- Historique clair
- Bon pour cycles longs

</div>
<div>

## ❌ Inconvénients

- Complexe
- Beaucoup de branches
- Merges fréquents
- Overkill pour petites équipes

</div>
</div>

---

# GitHub Flow

```
main    ●───●───●───●───●───●───●───●
         \     / \     / \       /
feature   ●───●   ●───●   ●─────●

        [PR]   [PR]    [PR]
```

**Simple** : main + feature branches + Pull Requests

---

# GitHub Flow : Règles

1. **main** est toujours déployable
2. Créer une branche pour chaque feature
3. Ouvrir une Pull Request
4. Review et discussion
5. Merge après approbation
6. Déployer immédiatement

---

# GitHub Flow : Workflow

```bash
# Nouvelle feature
git checkout main
git pull
git checkout -b feature/login

# Développement
git add .
git commit -m "feat: add login page"
git push -u origin feature/login

# → Ouvrir PR sur GitHub
# → Review
# → Merge
# → Delete branch
```

---

# GitHub Flow : Avantages / Inconvénients

<div class="columns">
<div>

## ✅ Avantages

- Simple à comprendre
- Déploiement continu
- Moins de branches
- Feedback rapide

</div>
<div>

## ❌ Inconvénients

- Pas de staging branch
- Nécessite CI robuste
- main doit être stable

</div>
</div>

**Recommandé pour** : Équipes agiles, déploiement fréquent

---

# Trunk-Based Development

```
main    ●───●───●───●───●───●───●───●
         \/ \/ \/   \/   \/ \/ \/
         Commits directs ou branches très courtes
         (< 1 jour)
```

**Extrême** : Tout le monde commit sur main

---

# Trunk-Based : Pratiques

- Commits directs sur main (petits changements)
- Branches ultra-courtes (< 24h)
- Feature flags pour code incomplet
- CI très rapide (< 10 min)
- Tests exhaustifs

```javascript
// Feature flag
if (featureFlags.newLogin) {
  return <NewLoginPage />;
}
return <OldLoginPage />;
```

---

# Comparatif

| Critère | Git Flow | GitHub Flow | Trunk-Based |
|---------|----------|-------------|-------------|
| Complexité | Élevée | Faible | Très faible |
| Cycles release | Longs | Courts | Très courts |
| Équipe | Grande | Moyenne | Petite/Experte |
| CI requis | Standard | Bon | Excellent |
| Risque main | Faible | Moyen | Élevé |

**Pour ce cours** : GitHub Flow

---

<!-- _class: lead -->

# Partie 2
## Pull Requests et CI

---

# Anatomie d'une Pull Request

```
┌─────────────────────────────────────────────┐
│  feat: Add login page  #42                  │
├─────────────────────────────────────────────┤
│  feature/login → main                       │
├─────────────────────────────────────────────┤
│  📝 Description                             │
│  ✅ Checks (CI)                             │
│  💬 Conversations                           │
│  📦 Commits                                 │
│  📄 Files changed                           │
└─────────────────────────────────────────────┘
```

---

# CI sur Pull Request

```yaml
on:
  pull_request:
    branches: [main]
    types: [opened, synchronize, reopened]

jobs:
  ci:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      - run: npm ci
      - run: npm run lint
      - run: npm test
      - run: npm run build
```

**Chaque push sur la PR déclenche le CI**

---

# Checks obligatoires

```
┌─────────────────────────────────────────────┐
│  Checks                                     │
├─────────────────────────────────────────────┤
│  ✅ lint (12s)                              │
│  ✅ test (45s)                              │
│  ✅ build (23s)                             │
│  ⏳ Required: All checks must pass          │
└─────────────────────────────────────────────┘
│  🔀 Merge pull request                      │
└─────────────────────────────────────────────┘
```

---

# Branch Protection Rules

## Settings > Branches > Add rule

**Pattern** : `main`

**Options** :
- ☑️ Require a pull request before merging
- ☑️ Require approvals (1+)
- ☑️ Require status checks to pass
- ☑️ Require branches to be up to date
- ☑️ Include administrators

---

# Configuration via UI

```
Branch protection rule: main

☑️ Require a pull request before merging
   ☑️ Require approvals: 1
   ☑️ Dismiss stale approvals

☑️ Require status checks to pass
   ☑️ Require branches to be up to date
   Status checks: lint, test, build

☑️ Do not allow bypassing the above settings
```

---

# Merge strategies

<div class="columns">
<div>

## Merge commit
```
main  ●───●───●───M
       \       /
feat    ●───●───●
```
Historique complet

</div>
<div>

## Squash merge
```
main  ●───●───●───S
       \
feat    ●───●───●
        (écrasés)
```
1 commit propre

</div>
</div>

---

# Squash and merge

```yaml
# Avant: Commits sur feature branch
feat: start login
wip
fix typo
review fixes
oops

# Après: Squash merge
feat: add login page (#42)
```

**Avantage** : Historique main propre

---

# Auto-merge

```yaml
# Activer auto-merge quand CI passe
jobs:
  ci:
    runs-on: ubuntu-latest
    steps:
      - run: npm test

  automerge:
    needs: ci
    runs-on: ubuntu-latest
    if: github.actor == 'dependabot[bot]'
    steps:
      - uses: fastify/github-action-merge-dependabot@v3
        with:
          github-token: ${{ secrets.GITHUB_TOKEN }}
```

---

<!-- _class: lead -->

# Partie 3
## Semantic Versioning

---

# SemVer

> **Semantic Versioning** : Convention de numérotation des versions

```
    MAJOR.MINOR.PATCH
       │     │     │
       │     │     └── Bug fixes (backward compatible)
       │     └──────── New features (backward compatible)
       └────────────── Breaking changes
```

**Exemple** : `2.4.1`

---

# Quand incrémenter ?

| Changement | Exemple | Version |
|------------|---------|---------|
| Bug fix | Correction d'un crash | 1.0.0 → 1.0.**1** |
| Feature | Nouvelle fonction | 1.0.1 → 1.**1**.0 |
| Breaking | API modifiée | 1.1.0 → **2**.0.0 |

```javascript
// v1.0.0
function greet(name) { return `Hello ${name}`; }

// v2.0.0 - BREAKING: paramètre changé
function greet(options) { return `Hello ${options.name}`; }
```

---

# Versions pre-release

```
1.0.0-alpha      # Très instable
1.0.0-alpha.1    # Alpha avec numéro
1.0.0-beta       # Plus stable
1.0.0-beta.2     # Beta avec numéro
1.0.0-rc.1       # Release Candidate
1.0.0            # Release finale
```

**Ordre** : alpha < beta < rc < release

---

# Tags Git

```bash
# Créer un tag
git tag v1.0.0
git push origin v1.0.0

# Tag annoté (recommandé)
git tag -a v1.0.0 -m "Release version 1.0.0"
git push origin v1.0.0

# Lister les tags
git tag -l "v1.*"

# Supprimer un tag
git tag -d v1.0.0
git push origin --delete v1.0.0
```

---

# Conventional Commits

> Standard pour les messages de commit

```
<type>(<scope>): <description>

[optional body]

[optional footer]
```

**Types** : feat, fix, docs, style, refactor, test, chore

---

# Exemples Conventional Commits

```bash
feat(auth): add login page

fix(api): handle null response

docs: update README

refactor(utils): simplify date formatting

chore(deps): update dependencies

feat!: change API response format
# Le ! indique un breaking change

BREAKING CHANGE: API now returns object instead of array
```

---

# Pourquoi Conventional Commits ?

## Avantages

- ✅ Changelog automatique
- ✅ Version automatique (SemVer)
- ✅ Historique lisible
- ✅ Communication claire

```bash
feat → MINOR bump
fix → PATCH bump
feat! ou BREAKING CHANGE → MAJOR bump
```

---

<!-- _class: lead -->

# Partie 4
## Releases automatiques

---

# Workflow de release manuelle

```bash
# 1. Mettre à jour la version
npm version patch  # ou minor, major

# 2. Pousser le tag
git push origin main --tags

# 3. Créer la release sur GitHub
gh release create v1.0.1 --notes "Bug fixes"
```

---

# Release automatique sur tag

```yaml
name: Release

on:
  push:
    tags:
      - 'v*'

jobs:
  release:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4

      - name: Create Release
        uses: softprops/action-gh-release@v2
        with:
          generate_release_notes: true
```

---

# Avec artifacts

```yaml
jobs:
  release:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      - uses: actions/setup-node@v4
      - run: npm ci
      - run: npm run build

      - name: Create Release
        uses: softprops/action-gh-release@v2
        with:
          files: |
            dist/*.js
            dist/*.css
          generate_release_notes: true
```

---

# Release-please (Google)

> Automatise versions et changelogs via Conventional Commits

```yaml
name: Release Please

on:
  push:
    branches: [main]

jobs:
  release-please:
    runs-on: ubuntu-latest
    steps:
      - uses: google-github-actions/release-please-action@v4
        with:
          release-type: node
```

---

# Comment fonctionne release-please

```
1. Push feat: add login  →  PR "chore: release 1.1.0"
2. Push fix: handle null →  PR mise à jour
3. Merge la PR           →  Tag v1.1.0 créé
                         →  Release créée
                         →  CHANGELOG.md mis à jour
```

**Avantage** : Tout est automatique basé sur les commits

---

# CHANGELOG automatique

```markdown
# Changelog

## [1.1.0](https://github.com/user/repo/compare/v1.0.0...v1.1.0) (2024-01-15)

### Features

* add login page ([#42](https://github.com/user/repo/pull/42))
* add password reset ([#45](https://github.com/user/repo/pull/45))

### Bug Fixes

* handle null response ([#43](https://github.com/user/repo/pull/43))
```

---

<!-- _class: lead -->

# Partie 5
## TP : Branches et releases

---

# TP4 : TaskFlow avec branches

## Objectifs

1. Configurer les branch protection rules
2. Créer une feature via PR
3. Configurer release automatique
4. Créer la release v1.0.0

---

# Étape 1 : Branch protection

**Settings > Branches > Add rule**

- Branch pattern: `main`
- ☑️ Require a pull request
- ☑️ Require status checks: `lint`, `test`, `build`
- ☑️ Require branches to be up to date

---

# Étape 2 : Feature via PR

```bash
# Créer une branche
git checkout -b feature/improve-ui

# Faire des changements
echo "/* New styles */" >> src/styles.css
git add .
git commit -m "feat(ui): improve task list styling"
git push -u origin feature/improve-ui

# Ouvrir PR sur GitHub
gh pr create --title "feat(ui): improve styling" --body "..."
```

---

# Étape 3 : Workflow release

```yaml
# .github/workflows/release.yml
name: Release

on:
  push:
    tags:
      - 'v*'

permissions:
  contents: write

jobs:
  release:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      - uses: actions/setup-node@v4
      - run: npm ci
      - run: npm run build
      - uses: softprops/action-gh-release@v2
        with:
          generate_release_notes: true
```

---

# Étape 4 : Créer v1.0.0

```bash
# Merger la PR (via GitHub)

# Créer le tag
git checkout main
git pull
git tag -a v1.0.0 -m "First stable release"
git push origin v1.0.0

# → Le workflow crée automatiquement la release
```

---

# Récapitulatif Jour 4

## Ce qu'on a appris

- ✅ Git Flow vs GitHub Flow
- ✅ Branch protection rules
- ✅ SemVer et Conventional Commits
- ✅ Releases automatiques avec tags

## Demain (Jour 5)

- Déploiement continu (CD)
- Docker et ghcr.io
- GitHub Pages
- QCM final

---

<!-- _class: lead -->

# Questions ?

---

# Ressources

## Documentation
- [GitHub Flow](https://docs.github.com/en/get-started/quickstart/github-flow)
- [SemVer](https://semver.org/)
- [Conventional Commits](https://www.conventionalcommits.org/)
- [Release Please](https://github.com/google-github-actions/release-please-action)

## Projet
- [TaskFlow - Votre fork](https://github.com/YOUR-USER/taskflow)

---

<!-- _class: lead -->

# Merci !

**Jour 5 : Déploiement continu**
Docker, ghcr.io, GitHub Pages
