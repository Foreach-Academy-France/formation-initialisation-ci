# TP Jour 4 : Branches et releases

> **Durée** : 2h | **Objectif** : Branch protection, SemVer, et release automatique v1.0.0

## Point de départ

Vous devez avoir complété le TP Jour 3 avec :
- ✅ Tests avec coverage ≥ 70%
- ✅ Pipeline lint → test → build → deploy

---

## Étape 1 : Branch Protection (25 min)

### 1.1 Accéder aux paramètres

1. Aller sur votre repository GitHub
2. **Settings** > **Branches**
3. Cliquer **Add branch protection rule**

### 1.2 Configurer la protection

**Branch name pattern** : `main`

**Cocher les options** :

- [x] **Require a pull request before merging**
  - [x] Require approvals: `1` (ou `0` pour le TP si vous êtes seul)
  - [x] Dismiss stale pull request approvals when new commits are pushed

- [x] **Require status checks to pass before merging**
  - [x] Require branches to be up to date before merging
  - Rechercher et ajouter : `Lint (Node 20)`, `Test`, `Build`

- [x] **Do not allow bypassing the above settings**

### 1.3 Sauvegarder

Cliquer **Create** ou **Save changes**

### 1.4 Tester la protection

```bash
# Essayer de pousser directement sur main
echo "test" >> README.md
git add README.md
git commit -m "test: direct push"
git push origin main
```

**Résultat attendu** : Erreur ! Push rejeté car la branche est protégée.

```bash
# Annuler le commit
git reset HEAD~1
git checkout README.md
```

---

## Étape 2 : Workflow via Pull Request (20 min)

### 2.1 Créer une branche feature

```bash
git checkout -b feature/filter-tasks
```

### 2.2 Ajouter une fonctionnalité

Modifier `src/tasks.js` pour ajouter une méthode de filtrage :

```javascript
  // Ajouter dans la classe TaskManager
  filterByStatus(status) {
    if (status === 'all') return this.getTasks();
    if (status === 'completed') return this.tasks.filter(t => t.completed);
    if (status === 'pending') return this.tasks.filter(t => !t.completed);
    return this.getTasks();
  }
```

### 2.3 Ajouter le test correspondant

Dans `tests/tasks.test.js` :

```javascript
  describe('filterByStatus', () => {
    beforeEach(() => {
      manager.addTask('Task 1');
      manager.addTask('Task 2');
      manager.addTask('Task 3');
      manager.toggleTask(manager.getTasks()[0].id); // Complete first task
    });

    test('should return all tasks with "all" filter', () => {
      const filtered = manager.filterByStatus('all');
      expect(filtered).toHaveLength(3);
    });

    test('should return only completed tasks', () => {
      const filtered = manager.filterByStatus('completed');
      expect(filtered).toHaveLength(1);
      expect(filtered[0].completed).toBe(true);
    });

    test('should return only pending tasks', () => {
      const filtered = manager.filterByStatus('pending');
      expect(filtered).toHaveLength(2);
      filtered.forEach(t => expect(t.completed).toBe(false));
    });
  });
```

### 2.4 Vérifier localement

```bash
npm test
npm run lint
```

### 2.5 Créer la Pull Request

```bash
git add .
git commit -m "feat(tasks): add filter by status"
git push -u origin feature/filter-tasks
```

Créer la PR :

```bash
gh pr create \
  --title "feat(tasks): add filter by status" \
  --body "## Summary
- Add filterByStatus method to TaskManager
- Support 'all', 'completed', 'pending' filters
- Add unit tests for filter functionality

## Test plan
- [x] Run npm test locally
- [x] Verify coverage maintained"
```

### 2.6 Observer le CI sur la PR

1. Aller sur la PR
2. Section "Checks" : le CI s'exécute
3. Attendre que tous les checks passent ✅

### 2.7 Merger la PR

Via l'interface GitHub :
1. Cliquer **Squash and merge**
2. Confirmer le merge
3. Supprimer la branche

Ou via CLI :
```bash
gh pr merge --squash --delete-branch
```

---

## Étape 3 : Conventional Commits (15 min)

### 3.1 Format des commits

À partir de maintenant, utiliser le format :

```
<type>(<scope>): <description>

[optional body]

[optional footer]
```

**Types** :
| Type | Description | Bump version |
|------|-------------|--------------|
| `feat` | Nouvelle fonctionnalité | MINOR |
| `fix` | Correction de bug | PATCH |
| `docs` | Documentation | - |
| `style` | Formatage | - |
| `refactor` | Refactoring | - |
| `test` | Tests | - |
| `chore` | Maintenance | - |
| `feat!` | Breaking change | MAJOR |

### 3.2 Exemples

```bash
# Feature
git commit -m "feat(auth): add login page"

# Bug fix
git commit -m "fix(api): handle null response"

# Breaking change
git commit -m "feat!: change API response format

BREAKING CHANGE: API now returns object instead of array"
```

---

## Étape 4 : Workflow de Release (30 min)

### 4.1 Créer le workflow release

Créer `.github/workflows/release.yml` :

```yaml
name: Release

on:
  push:
    tags:
      - 'v*'

permissions:
  contents: write

jobs:
  release:
    name: Create Release
    runs-on: ubuntu-latest
    steps:
      - name: Checkout
        uses: actions/checkout@v4
        with:
          fetch-depth: 0

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

      - name: Create Release
        uses: softprops/action-gh-release@v2
        with:
          generate_release_notes: true
          files: |
            dist/*
        env:
          GITHUB_TOKEN: ${{ secrets.GITHUB_TOKEN }}
```

### 4.2 Commit le workflow

```bash
git checkout main
git pull origin main
git add .github/workflows/release.yml
git commit -m "ci: add release workflow on tags"
git push origin main
```

---

## Étape 5 : Créer la release v1.0.0 (20 min)

### 5.1 Mettre à jour package.json

```bash
# Mettre la version à 1.0.0
npm version 1.0.0 --no-git-tag-version
```

Ou modifier manuellement `package.json` :
```json
{
  "version": "1.0.0"
}
```

### 5.2 Créer une branche pour la release

```bash
git checkout -b release/v1.0.0
git add package.json
git commit -m "chore: bump version to 1.0.0"
git push -u origin release/v1.0.0
```

### 5.3 Créer et merger la PR

```bash
gh pr create \
  --title "chore: release v1.0.0" \
  --body "## Release v1.0.0

### Features
- Task management (add, remove, toggle)
- Filter by status
- localStorage persistence

### CI/CD
- Lint with ESLint
- Tests with Vitest (coverage > 70%)
- Automated builds"
```

Merger après que le CI passe :
```bash
gh pr merge --squash --delete-branch
```

### 5.4 Créer le tag

```bash
git checkout main
git pull origin main
git tag -a v1.0.0 -m "Release v1.0.0 - First stable release"
git push origin v1.0.0
```

### 5.5 Vérifier la release

1. Aller dans **Actions** : le workflow "Release" doit s'exécuter
2. Aller dans **Releases** : la release v1.0.0 doit apparaître avec :
   - Release notes générées automatiquement
   - Assets (fichiers du build)

---

## Étape 6 : Badge release (10 min)

### 6.1 Ajouter le badge au README

```markdown
# TaskFlow

[![CI](https://github.com/VOTRE-USER/taskflow-starter/actions/workflows/ci.yml/badge.svg)](https://github.com/VOTRE-USER/taskflow-starter/actions)
[![Release](https://img.shields.io/github/v/release/VOTRE-USER/taskflow-starter)](https://github.com/VOTRE-USER/taskflow-starter/releases)
[![Coverage](https://img.shields.io/badge/coverage-%3E70%25-brightgreen)](./coverage/)
```

### 6.2 Commit

```bash
git checkout -b docs/badges
git add README.md
git commit -m "docs: add release badge"
git push -u origin docs/badges
gh pr create --title "docs: add release badge" --body "Add release version badge"
gh pr merge --squash --delete-branch
```

---

## Checklist de validation

Avant de passer au Jour 5, vérifiez :

- [ ] Branch protection active sur `main`
  - [ ] Require PR
  - [ ] Require status checks (lint, test, build)
- [ ] Au moins une PR créée et mergée
- [ ] Workflow `release.yml` créé
- [ ] Tag `v1.0.0` créé et poussé
- [ ] Release GitHub visible avec notes
- [ ] Badge release dans README

---

## Erreurs courantes

### Push rejeté sur main

**Cause** : Branch protection active (c'est normal !)

**Solution** : Toujours passer par une PR

### Le workflow release ne se déclenche pas

**Vérifier** :
1. Le tag est bien poussé : `git tag -l` et `git ls-remote --tags`
2. Le format du tag correspond : `v*` (ex: v1.0.0)

### Release sans notes

**Cause** : `fetch-depth: 0` manquant dans checkout

**Vérifier** :
```yaml
- uses: actions/checkout@v4
  with:
    fetch-depth: 0    # Important pour l'historique !
```

### Pas de permission pour créer la release

**Vérifier** :
```yaml
permissions:
  contents: write    # Nécessaire pour créer des releases
```

---

## Ressources

- [Branch protection rules](https://docs.github.com/en/repositories/configuring-branches-and-merges-in-your-repository/defining-the-mergeability-of-pull-requests/about-protected-branches)
- [Semantic Versioning](https://semver.org/)
- [Conventional Commits](https://www.conventionalcommits.org/)
- [GitHub Releases](https://docs.github.com/en/repositories/releasing-projects-on-github/about-releases)

---

**Prochain TP** : [Jour 5 - Déploiement complet](./jour5-deploiement-complet.md)
