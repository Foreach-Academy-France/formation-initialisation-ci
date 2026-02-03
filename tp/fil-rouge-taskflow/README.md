# Projet Fil Rouge : TaskFlow

> Application de gestion de tâches évoluant sur les 5 jours de formation

## Concept

TaskFlow est une application web simple que les étudiants enrichissent chaque jour avec de nouvelles fonctionnalités CI/CD. Le même repository est utilisé tout au long de la formation.

## Stack technique

- **Frontend** : Vanilla JS (ou React si niveau avancé)
- **Stockage** : localStorage (pas de backend)
- **Build** : Vite
- **Tests** : Vitest
- **Linting** : ESLint + Prettier
- **Container** : Docker
- **Registry** : GitHub Container Registry (ghcr.io)
- **Déploiement** : GitHub Pages

## Progression jour par jour

### Jour 1 : Premier workflow CI

**Objectif** : Mettre en place le premier pipeline CI

**Étapes** :
1. Fork du repository starter `taskflow-starter`
2. Créer `.github/workflows/ci.yml`
3. Workflow basique : checkout → install → lint

**Livrable** :
```yaml
name: CI
on: [push, pull_request]
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

### Jour 2 : Configuration avancée

**Objectif** : Jobs multiples, matrix, secrets

**Étapes** :
1. Ajouter un job `build` séparé
2. Matrix pour tester Node 18, 20, 22
3. Configurer un secret `APP_VERSION`
4. Ajouter un trigger `workflow_dispatch`

**Livrable** :
- Job `lint` + Job `build` avec dépendance
- Matrix 3 versions Node.js
- Build conditionnel sur main uniquement

---

### Jour 3 : Tests et qualité

**Objectif** : Intégrer les tests automatisés

**Étapes** :
1. Écrire 5+ tests unitaires (Vitest)
2. Ajouter job `test` avec coverage
3. Upload du rapport coverage en artifact
4. Faire échouer le CI si coverage < 70%

**Livrable** :
- Tests unitaires fonctionnels
- Rapport coverage uploadé
- Badge coverage dans README

---

### Jour 4 : Branches et releases

**Objectif** : Workflow de branches et versioning

**Étapes** :
1. Configurer branch protection sur `main`
2. Créer workflow de release sur tag
3. Générer changelog automatique
4. Créer une release v1.0.0

**Livrable** :
- Branch protection active (require PR, require CI pass)
- Workflow release déclenché par tag `v*`
- Release GitHub avec changelog

---

### Jour 5 : Pipeline complet avec déploiement

**Objectif** : CI/CD complet jusqu'au déploiement

**Étapes** :
1. Build image Docker
2. Push sur ghcr.io (GitHub Container Registry)
3. Déployer sur GitHub Pages (staging sur PR, prod sur main)
4. Environnements GitHub avec protection

**Livrable** :
- Image Docker sur ghcr.io
- Site déployé sur GitHub Pages
- Environnement `production` avec approval (optionnel)

---

## Structure du repository final

```
taskflow/
├── .github/
│   └── workflows/
│       ├── ci.yml           # Lint + Test + Build
│       ├── release.yml      # Release sur tag
│       └── deploy.yml       # Build Docker + Deploy Pages
├── src/
│   ├── index.html
│   ├── app.js
│   ├── tasks.js             # Logique métier
│   └── styles.css
├── tests/
│   └── tasks.test.js
├── Dockerfile
├── .dockerignore
├── vite.config.js
├── vitest.config.js
├── package.json
└── README.md                # Avec badges CI/coverage
```

## Évaluation

L'évaluation porte sur l'état final du repository :

| Critère | Points | Vérifié par |
|---------|--------|-------------|
| Workflow CI fonctionnel | 20 | Badge vert |
| Tests passent (coverage ≥ 70%) | 20 | Rapport artifact |
| Branch protection configurée | 15 | Settings repo |
| Au moins 1 release taguée | 15 | Releases GitHub |
| Image Docker sur ghcr.io | 15 | Packages repo |
| Déploiement GitHub Pages | 15 | Site accessible |
| **Total** | **100** | |

## Ressources

- [Starter repository](./starter/) - Point de départ Jour 1
- [Cheatsheet GitHub Actions](../../ressources/cheatsheet.md)
- [GitHub Container Registry docs](https://docs.github.com/en/packages/working-with-a-github-packages-registry/working-with-the-container-registry)
