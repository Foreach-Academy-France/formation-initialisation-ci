# TP Jour 3 : Tests et qualité de code

> **Durée** : 2h | **Objectif** : Intégrer les tests automatisés avec coverage ≥ 70%

## Point de départ

Vous devez avoir complété le TP Jour 2 avec :
- ✅ Matrix build (Node 18, 20, 22)
- ✅ Jobs lint → build → deploy
- ✅ Variables et workflow_dispatch

---

## Étape 1 : Configurer Vitest (20 min)

### 1.1 Vérifier l'installation

Le projet starter inclut déjà Vitest. Vérifier dans `package.json` :

```json
{
  "devDependencies": {
    "vitest": "^1.0.0",
    "@vitest/coverage-v8": "^1.0.0"
  },
  "scripts": {
    "test": "vitest run",
    "test:watch": "vitest",
    "test:coverage": "vitest run --coverage"
  }
}
```

Si Vitest n'est pas installé :
```bash
npm install -D vitest @vitest/coverage-v8
```

### 1.2 Créer/Vérifier la configuration Vitest

Créer ou modifier `vitest.config.js` :

```javascript
import { defineConfig } from 'vitest/config';

export default defineConfig({
  test: {
    globals: true,
    environment: 'jsdom',
    include: ['tests/**/*.test.js'],
    coverage: {
      provider: 'v8',
      reporter: ['text', 'html', 'lcov'],
      reportsDirectory: './coverage',
      exclude: [
        'node_modules/',
        'tests/',
        '*.config.js',
      ],
      thresholds: {
        lines: 70,
        branches: 70,
        functions: 70,
        statements: 70,
      },
    },
  },
});
```

### 1.3 Tester localement

```bash
npm test
```

---

## Étape 2 : Écrire les tests (40 min)

### 2.1 Structure des tests

```
tests/
├── tasks.test.js      # Tests du module tasks
└── utils.test.js      # Tests des utilitaires (si applicable)
```

### 2.2 Créer les tests pour le module tasks

Créer `tests/tasks.test.js` :

```javascript
import { describe, test, expect, beforeEach } from 'vitest';

// Simuler localStorage pour les tests
const localStorageMock = {
  store: {},
  getItem(key) {
    return this.store[key] || null;
  },
  setItem(key, value) {
    this.store[key] = value;
  },
  clear() {
    this.store = {};
  },
};

global.localStorage = localStorageMock;

// Importer après le mock
import { TaskManager } from '../src/tasks.js';

describe('TaskManager', () => {
  let manager;

  beforeEach(() => {
    localStorage.clear();
    manager = new TaskManager();
  });

  describe('Initialization', () => {
    test('should start with empty tasks', () => {
      expect(manager.getTasks()).toEqual([]);
    });

    test('should have a unique instance ID', () => {
      expect(manager.id).toBeDefined();
    });
  });

  describe('addTask', () => {
    test('should add a task with title', () => {
      manager.addTask('Buy milk');

      const tasks = manager.getTasks();
      expect(tasks).toHaveLength(1);
      expect(tasks[0].title).toBe('Buy milk');
    });

    test('should generate unique ID for each task', () => {
      manager.addTask('Task 1');
      manager.addTask('Task 2');

      const tasks = manager.getTasks();
      expect(tasks[0].id).not.toBe(tasks[1].id);
    });

    test('should set completed to false by default', () => {
      manager.addTask('New task');

      const tasks = manager.getTasks();
      expect(tasks[0].completed).toBe(false);
    });

    test('should add createdAt timestamp', () => {
      const before = Date.now();
      manager.addTask('Timed task');
      const after = Date.now();

      const task = manager.getTasks()[0];
      expect(task.createdAt).toBeGreaterThanOrEqual(before);
      expect(task.createdAt).toBeLessThanOrEqual(after);
    });
  });

  describe('removeTask', () => {
    test('should remove task by ID', () => {
      manager.addTask('To remove');
      const taskId = manager.getTasks()[0].id;

      manager.removeTask(taskId);

      expect(manager.getTasks()).toHaveLength(0);
    });

    test('should not affect other tasks', () => {
      manager.addTask('Keep this');
      manager.addTask('Remove this');

      const tasks = manager.getTasks();
      const keepId = tasks[0].id;
      const removeId = tasks[1].id;

      manager.removeTask(removeId);

      expect(manager.getTasks()).toHaveLength(1);
      expect(manager.getTasks()[0].id).toBe(keepId);
    });
  });

  describe('toggleTask', () => {
    test('should toggle completed status', () => {
      manager.addTask('Toggle me');
      const taskId = manager.getTasks()[0].id;

      manager.toggleTask(taskId);
      expect(manager.getTasks()[0].completed).toBe(true);

      manager.toggleTask(taskId);
      expect(manager.getTasks()[0].completed).toBe(false);
    });
  });

  describe('getStats', () => {
    test('should return correct stats', () => {
      manager.addTask('Task 1');
      manager.addTask('Task 2');
      manager.addTask('Task 3');

      const taskId = manager.getTasks()[0].id;
      manager.toggleTask(taskId);

      const stats = manager.getStats();

      expect(stats.total).toBe(3);
      expect(stats.completed).toBe(1);
      expect(stats.pending).toBe(2);
    });

    test('should return zeros for empty list', () => {
      const stats = manager.getStats();

      expect(stats.total).toBe(0);
      expect(stats.completed).toBe(0);
      expect(stats.pending).toBe(0);
    });
  });

  describe('Persistence', () => {
    test('should save tasks to localStorage', () => {
      manager.addTask('Persistent task');

      const stored = JSON.parse(localStorage.getItem('taskflow-tasks'));
      expect(stored).toHaveLength(1);
      expect(stored[0].title).toBe('Persistent task');
    });

    test('should load tasks from localStorage on init', () => {
      // Préparer des données
      const existingTasks = [
        { id: '1', title: 'Existing', completed: false, createdAt: Date.now() }
      ];
      localStorage.setItem('taskflow-tasks', JSON.stringify(existingTasks));

      // Nouveau manager doit charger les données
      const newManager = new TaskManager();

      expect(newManager.getTasks()).toHaveLength(1);
      expect(newManager.getTasks()[0].title).toBe('Existing');
    });
  });
});
```

### 2.3 Adapter le module tasks si nécessaire

Vérifier que `src/tasks.js` exporte une classe `TaskManager` :

```javascript
// src/tasks.js
export class TaskManager {
  constructor() {
    this.id = crypto.randomUUID();
    this.tasks = this.loadTasks();
  }

  loadTasks() {
    const stored = localStorage.getItem('taskflow-tasks');
    return stored ? JSON.parse(stored) : [];
  }

  saveTasks() {
    localStorage.setItem('taskflow-tasks', JSON.stringify(this.tasks));
  }

  getTasks() {
    return [...this.tasks];
  }

  addTask(title) {
    const task = {
      id: crypto.randomUUID(),
      title,
      completed: false,
      createdAt: Date.now(),
    };
    this.tasks.push(task);
    this.saveTasks();
    return task;
  }

  removeTask(id) {
    this.tasks = this.tasks.filter(t => t.id !== id);
    this.saveTasks();
  }

  toggleTask(id) {
    const task = this.tasks.find(t => t.id === id);
    if (task) {
      task.completed = !task.completed;
      this.saveTasks();
    }
  }

  getStats() {
    const total = this.tasks.length;
    const completed = this.tasks.filter(t => t.completed).length;
    return {
      total,
      completed,
      pending: total - completed,
    };
  }
}
```

### 2.4 Lancer les tests

```bash
npm test
```

Tous les tests doivent passer ✅

---

## Étape 3 : Coverage (20 min)

### 3.1 Générer le rapport de coverage

```bash
npm run test:coverage
```

### 3.2 Vérifier les seuils

Le terminal affiche :

```
 % Coverage report from v8
-----------------------------|---------|----------|---------|---------|
File                         | % Stmts | % Branch | % Funcs | % Lines |
-----------------------------|---------|----------|---------|---------|
All files                    |   85.00 |   80.00 |   90.00 |   85.00 |
 src/tasks.js                |   85.00 |   80.00 |   90.00 |   85.00 |
-----------------------------|---------|----------|---------|---------|
```

### 3.3 Visualiser le rapport HTML

```bash
# Ouvrir le rapport
open coverage/index.html    # macOS
xdg-open coverage/index.html  # Linux
```

### 3.4 Améliorer le coverage si < 70%

Si le coverage est insuffisant, ajouter des tests pour les cas non couverts (voir les lignes en rouge dans le rapport HTML).

---

## Étape 4 : Intégrer au CI (25 min)

### 4.1 Ajouter le job test

Modifier `.github/workflows/ci.yml` pour ajouter un job `test` :

```yaml
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

      - name: Run tests with coverage
        run: npm run test:coverage

      - name: Upload coverage report
        uses: actions/upload-artifact@v4
        with:
          name: coverage-report
          path: coverage/
          retention-days: 14
```

### 4.2 Mettre à jour les dépendances

Le job `build` doit maintenant attendre `test` :

```yaml
  build:
    name: Build
    runs-on: ubuntu-latest
    needs: [lint, test]           # Attend lint ET test
    steps:
      # ... reste identique
```

### 4.3 Workflow complet mis à jour

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
      - name: Run tests with coverage
        run: npm run test:coverage
      - name: Upload coverage report
        uses: actions/upload-artifact@v4
        with:
          name: coverage-report
          path: coverage/

  build:
    name: Build
    runs-on: ubuntu-latest
    needs: [lint, test]
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
      - run: |
          echo "🚀 Deploying..."
          ls -la dist/
```

### 4.4 Commit et vérifier

```bash
git add .
git commit -m "feat(ci): add test job with coverage"
git push origin main
```

**Pipeline attendu** :
```
✅ CI
   ├── ✅ Lint (Node 18)
   ├── ✅ Lint (Node 20)
   ├── ✅ Lint (Node 22)
   ├── ✅ Test
   ├── ✅ Build
   └── ✅ Deploy
```

---

## Étape 5 : Badge coverage (15 min)

### 5.1 Option A : Badge simple (basé sur le CI)

Ajouter dans le README :

```markdown
[![CI](https://github.com/VOTRE-USER/taskflow-starter/actions/workflows/ci.yml/badge.svg)](https://github.com/VOTRE-USER/taskflow-starter/actions)
[![Coverage](https://img.shields.io/badge/coverage-%3E70%25-brightgreen)](./coverage/)
```

### 5.2 Option B : Codecov (avancé)

1. Aller sur https://codecov.io et se connecter avec GitHub
2. Ajouter le repository
3. Récupérer le token `CODECOV_TOKEN`
4. Ajouter le secret dans GitHub

Modifier le workflow :

```yaml
      - name: Upload to Codecov
        uses: codecov/codecov-action@v4
        with:
          token: ${{ secrets.CODECOV_TOKEN }}
          files: ./coverage/lcov.info
          fail_ci_if_error: true
```

Badge Codecov :
```markdown
[![codecov](https://codecov.io/gh/VOTRE-USER/taskflow-starter/branch/main/graph/badge.svg)](https://codecov.io/gh/VOTRE-USER/taskflow-starter)
```

---

## Checklist de validation

Avant de passer au Jour 4, vérifiez :

- [ ] `vitest.config.js` configuré avec seuils 70%
- [ ] Au moins 10 tests écrits
- [ ] Tous les tests passent (`npm test`)
- [ ] Coverage ≥ 70% (`npm run test:coverage`)
- [ ] Job `test` dans le CI
- [ ] Artifact coverage uploadé
- [ ] Badge coverage dans le README

---

## Erreurs courantes

### "Cannot find module '../src/tasks.js'"

**Cause** : Le chemin d'import est incorrect

**Solution** : Vérifier le chemin relatif depuis `tests/`

### Coverage < 70%

**Vérifier** :
1. Ouvrir `coverage/index.html`
2. Identifier les lignes non couvertes (en rouge)
3. Ajouter des tests pour ces cas

### "localStorage is not defined"

**Cause** : Les tests s'exécutent en Node.js, pas dans un navigateur

**Solution** : Ajouter le mock localStorage (voir étape 2.2)

---

## Ressources

- [Vitest Documentation](https://vitest.dev/)
- [Coverage v8](https://vitest.dev/guide/coverage.html)
- [Testing Best Practices](https://github.com/goldbergyoni/javascript-testing-best-practices)
- [Codecov](https://codecov.io/)

---

**Prochain TP** : [Jour 4 - Branches et releases](./jour4-branches-releases.md)
