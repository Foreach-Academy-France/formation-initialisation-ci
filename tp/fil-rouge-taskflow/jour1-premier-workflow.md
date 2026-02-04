# TP Jour 1 : Premier workflow CI

> **Durée** : 2h | **Objectif** : Mettre en place votre premier pipeline CI avec GitHub Actions

## Prérequis

- Compte GitHub
- Git installé localement
- Node.js 20+ installé
- VS Code (recommandé)

---

## Étape 1 : Fork du projet starter (15 min)

### 1.1 Fork via GitHub CLI

```bash
# Fork et clone en une commande
gh repo fork Foreach-Academy-France/taskflow-starter --clone

# Aller dans le dossier
cd taskflow-starter
```

### 1.2 Alternative : Fork via l'interface GitHub

1. Aller sur https://github.com/Foreach-Academy-France/taskflow-starter
2. Cliquer sur **Fork** (en haut à droite)
3. Cloner votre fork :
   ```bash
   git clone https://github.com/VOTRE-USERNAME/taskflow-starter.git
   cd taskflow-starter
   ```

### 1.3 Vérifier que le projet fonctionne

```bash
# Installer les dépendances
npm install

# Lancer en développement
npm run dev
```

Ouvrir http://localhost:5173 - Vous devez voir l'application TaskFlow.

### 1.4 Vérifier le linter

```bash
npm run lint
```

Le linter doit passer sans erreur (ou avec des warnings acceptables).

---

## Étape 2 : Créer le workflow CI (30 min)

### 2.1 Créer la structure

```bash
# Créer le dossier workflows
mkdir -p .github/workflows

# Créer le fichier workflow
touch .github/workflows/ci.yml
```

### 2.2 Écrire le workflow

Ouvrir `.github/workflows/ci.yml` et ajouter :

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

### 2.3 Comprendre chaque partie

| Section | Description |
|---------|-------------|
| `name: CI` | Nom affiché dans l'onglet Actions |
| `on: push/pull_request` | Déclencheurs du workflow |
| `jobs: lint:` | Définition du job "lint" |
| `runs-on: ubuntu-latest` | Machine virtuelle utilisée |
| `steps:` | Liste des étapes à exécuter |

---

## Étape 3 : Pousser et vérifier (15 min)

### 3.1 Commit et push

```bash
# Ajouter les fichiers
git add .github/workflows/ci.yml

# Committer
git commit -m "feat: add CI workflow with lint"

# Pousser
git push origin main
```

### 3.2 Vérifier sur GitHub

1. Aller sur votre repository GitHub
2. Cliquer sur l'onglet **Actions**
3. Vous devez voir le workflow "CI" en cours d'exécution
4. Cliquer dessus pour voir les détails

### 3.3 Résultat attendu

```
✅ CI
   └── ✅ lint
       ├── ✅ Checkout code
       ├── ✅ Setup Node.js
       ├── ✅ Install dependencies
       └── ✅ Run linter
```

---

## Étape 4 : Ajouter le badge (15 min)

### 4.1 Récupérer l'URL du badge

1. Dans l'onglet Actions, cliquer sur le workflow "CI"
2. Cliquer sur les "..." en haut à droite
3. Sélectionner "Create status badge"
4. Copier le code Markdown

### 4.2 Ajouter au README

Ouvrir `README.md` et ajouter en haut :

```markdown
# TaskFlow

[![CI](https://github.com/VOTRE-USERNAME/taskflow-starter/actions/workflows/ci.yml/badge.svg)](https://github.com/VOTRE-USERNAME/taskflow-starter/actions/workflows/ci.yml)

Application de gestion de tâches...
```

### 4.3 Commit

```bash
git add README.md
git commit -m "docs: add CI badge to README"
git push origin main
```

---

## Étape 5 : Tester avec une erreur (15 min)

### 5.1 Introduire une erreur de lint

Ouvrir `src/tasks.js` et ajouter une variable inutilisée :

```javascript
// En haut du fichier
const unusedVariable = 'This will fail lint';
```

### 5.2 Commit et observer

```bash
git add src/tasks.js
git commit -m "test: introduce lint error"
git push origin main
```

### 5.3 Vérifier l'échec

1. Aller dans l'onglet Actions
2. Le workflow doit être ❌ rouge
3. Cliquer pour voir l'erreur :
   ```
   error  'unusedVariable' is assigned a value but never used  no-unused-vars
   ```

### 5.4 Corriger

```bash
# Supprimer la ligne ajoutée
git checkout src/tasks.js
git add src/tasks.js
git commit -m "fix: remove unused variable"
git push origin main
```

Le workflow doit repasser au ✅ vert.

---

## Étape 6 : Bonus - Tester via Pull Request (30 min)

### 6.1 Créer une branche

```bash
git checkout -b feature/improve-styles
```

### 6.2 Faire une modification

Modifier `src/styles.css` :

```css
/* Ajouter à la fin */
.task-item:hover {
  background-color: #f0f0f0;
  transition: background-color 0.2s;
}
```

### 6.3 Créer la PR

```bash
git add src/styles.css
git commit -m "feat(ui): add hover effect on tasks"
git push -u origin feature/improve-styles
```

Créer la PR via GitHub CLI :

```bash
gh pr create --title "feat(ui): add hover effect" --body "Adds a subtle hover effect on task items"
```

### 6.4 Observer le CI sur la PR

1. Aller sur la PR
2. Voir la section "Checks"
3. Le CI s'exécute automatiquement sur la PR

---

## Checklist de validation

Avant de passer au Jour 2, vérifiez :

- [ ] Le repository est forké et cloné
- [ ] L'application fonctionne localement (`npm run dev`)
- [ ] Le fichier `.github/workflows/ci.yml` existe
- [ ] Le workflow CI passe (badge vert)
- [ ] Le badge est visible dans le README
- [ ] Vous comprenez le rôle de chaque section du workflow

---

## Erreurs courantes

### "npm ci" échoue

**Cause** : Pas de `package-lock.json`

**Solution** :
```bash
npm install
git add package-lock.json
git commit -m "chore: add package-lock.json"
```

### Le workflow ne se déclenche pas

**Cause** : Le fichier n'est pas au bon endroit

**Vérifier** :
```bash
ls -la .github/workflows/ci.yml
```

Le chemin doit être exactement `.github/workflows/ci.yml`

### Permission denied sur GitHub Actions

**Cause** : Actions désactivées sur le fork

**Solution** :
1. Settings > Actions > General
2. Sélectionner "Allow all actions"

---

## Ressources

- [GitHub Actions Quickstart](https://docs.github.com/en/actions/quickstart)
- [Workflow syntax](https://docs.github.com/en/actions/reference/workflow-syntax-for-github-actions)
- [actions/checkout](https://github.com/actions/checkout)
- [actions/setup-node](https://github.com/actions/setup-node)

---

**Prochain TP** : [Jour 2 - Configuration avancée](./jour2-configuration-avancee.md)
