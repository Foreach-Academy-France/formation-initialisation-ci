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

# 🧪 Jour 3
## Tests automatisés

**Formation CI/CD - GitHub Actions**
M2 ESTD - Architecte Web
ForEach Academy

---

## 👋 Rappel Jour 2

**Ce qu'on a appris** :
- ✅ Variables, contextes et secrets
- ✅ Jobs multiples et matrix builds
- ✅ Triggers avancés et conditions

**Aujourd'hui (Jour 3)** :
1. Types de tests et pyramide
2. Vitest pour les tests unitaires
3. Coverage et qualité de code
4. Artifacts et rapports CI

---

# Programme du jour

| Horaire | Contenu |
|---------|---------|
| 9h00-10h30 | **Types de tests et frameworks** |
| 10h45-12h15 | **Vitest : configuration et écriture** |
| 13h15-15h00 | **Coverage et qualité de code** |
| 15h15-17h00 | **TP : Tests dans TaskFlow** |

---

# Objectifs du jour

À la fin de cette journée, vous serez capables de :

- ✅ Comprendre la pyramide des tests
- ✅ Écrire des tests avec Vitest
- ✅ Configurer le coverage (≥ 70%)
- ✅ Intégrer les tests dans le CI
- ✅ Générer et publier des rapports

---

<!-- _class: lead -->

# Partie 1
## Types de tests

---

# Pourquoi tester ?

## Sans tests automatisés

- 😰 "Ça marche sur ma machine"
- 🐛 Bugs découverts en production
- 😱 Peur de modifier le code
- 🐌 Régression à chaque release

## Avec tests automatisés

- ✅ Confiance pour refactorer
- ✅ Documentation vivante
- ✅ Détection précoce des bugs
- ✅ Déploiement serein

---

# La pyramide des tests

<div style="display:flex;flex-direction:column;align-items:center;gap:4px;margin:20px auto;width:80%;">
  <div style="background:#fee2e2;border:2px solid #ef4444;color:#991b1b;padding:12px;text-align:center;width:40%;border-radius:6px;font-weight:bold;">🔴 E2E<br><small>Peu nombreux, Lents, Coûteux</small></div>
  <div style="background:#fef9c3;border:2px solid #eab308;color:#854d0e;padding:12px;text-align:center;width:60%;border-radius:6px;font-weight:bold;">🟡 Integration</div>
  <div style="background:#dcfce7;border:2px solid #22c55e;color:#166534;padding:12px;text-align:center;width:80%;border-radius:6px;font-weight:bold;">🟢 Unit Tests<br><small>Nombreux, Rapides, Peu coûteux</small></div>
</div>

**Plus on monte, plus c'est lent et coûteux**

---

# Tests unitaires

> Tester une unité de code isolée (fonction, classe)

```javascript
// math.js
export function add(a, b) {
  return a + b;
}

// math.test.js
import { add } from './math.js';

test('add returns sum of two numbers', () => {
  expect(add(2, 3)).toBe(5);
});
```

**Caractéristiques** : Rapides, isolés, nombreux

---

# Tests d'intégration

> Tester l'interaction entre plusieurs composants

```javascript
// userService.test.js
import { UserService } from './userService.js';
import { Database } from './database.js';

test('creates user in database', async () => {
  const db = new Database(':memory:');
  const service = new UserService(db);

  const user = await service.createUser('John');

  expect(user.id).toBeDefined();
  expect(await db.findUser(user.id)).toEqual(user);
});
```

---

# Tests End-to-End (E2E)

> Tester l'application complète comme un utilisateur

```javascript
// app.e2e.test.js (avec Playwright/Cypress)
test('user can create a task', async ({ page }) => {
  await page.goto('http://localhost:3000');

  await page.fill('[data-testid="task-input"]', 'Buy milk');
  await page.click('[data-testid="add-task"]');

  await expect(page.locator('.task-item')).toContainText('Buy milk');
});
```

**Caractéristiques** : Lents, fragiles, réalistes

---

# Répartition recommandée

| Type | % | Temps | Exemple TaskFlow |
|------|---|-------|------------------|
| Unit | 70% | ~ms | `addTask()`, `formatDate()` |
| Integration | 20% | ~100ms | API + Storage |
| E2E | 10% | ~s | Workflow utilisateur |

> **Règle** : Si un bug peut être détecté par un test unitaire, ne pas écrire de test E2E

---

# Frameworks de test JavaScript

| Framework | Type | Points forts |
|-----------|------|--------------|
| **Vitest** | Unit/Integration | Rapide, compatible Vite, moderne |
| Jest | Unit/Integration | Populaire, écosystème riche |
| Mocha | Unit/Integration | Flexible, configurable |
| Playwright | E2E | Multi-navigateur, moderne |
| Cypress | E2E | DX excellent, debugging |

**Pour ce cours : Vitest** (natif avec Vite)

---

<!-- _class: lead -->

# Partie 2
## Vitest

---

# Pourquoi Vitest ?

## Avantages

- ⚡ Ultra rapide (réutilise le bundler Vite)
- 🔄 HMR pour les tests (watch mode)
- 🎯 Compatible Jest (même API)
- 📦 Zero config avec Vite
- 🧩 TypeScript natif
- 🖼️ UI intégrée

---

# Installation

```bash
# Avec npm
npm install -D vitest

# Avec bun
bun add -D vitest
```

```json
// package.json
{
  "scripts": {
    "test": "vitest run",
    "test:watch": "vitest",
    "test:coverage": "vitest run --coverage"
  }
}
```

---

# Configuration

```javascript
// vite.config.js ou vitest.config.js
import { defineConfig } from 'vitest/config';

export default defineConfig({
  test: {
    globals: true,          // describe, test, expect globaux
    environment: 'jsdom',   // Pour tester du DOM
    include: ['**/*.{test,spec}.{js,ts}'],
    coverage: {
      provider: 'v8',
      reporter: ['text', 'html', 'lcov'],
      exclude: ['node_modules/', 'tests/'],
    },
  },
});
```

---

# Structure d'un test

```javascript
import { describe, test, expect, beforeEach } from 'vitest';
import { TaskManager } from '../src/taskManager.js';

describe('TaskManager', () => {
  let manager;

  beforeEach(() => {
    manager = new TaskManager();
  });

  test('should add a task', () => {
    manager.addTask('Buy milk');

    expect(manager.tasks).toHaveLength(1);
    expect(manager.tasks[0].title).toBe('Buy milk');
  });
});
```

---

# API de test

```javascript
// Égalité
expect(value).toBe(5);              // ===
expect(obj).toEqual({ a: 1 });      // Deep equal

// Vérité
expect(value).toBeTruthy();
expect(value).toBeFalsy();
expect(value).toBeNull();
expect(value).toBeDefined();

// Nombres
expect(num).toBeGreaterThan(3);
expect(num).toBeLessThanOrEqual(5);
expect(0.1 + 0.2).toBeCloseTo(0.3);

// Strings
expect(str).toMatch(/pattern/);
expect(str).toContain('substring');
```

---

# API de test (suite)

```javascript
// Arrays
expect(arr).toContain(item);
expect(arr).toHaveLength(3);

// Objects
expect(obj).toHaveProperty('key');
expect(obj).toMatchObject({ a: 1 });

// Exceptions
expect(() => fn()).toThrow();
expect(() => fn()).toThrow('message');
expect(() => fn()).toThrowError(TypeError);

// Async
await expect(promise).resolves.toBe(value);
await expect(promise).rejects.toThrow();
```

---

# Hooks de cycle de vie

```javascript
import { beforeAll, afterAll, beforeEach, afterEach } from 'vitest';

beforeAll(() => {
  // Avant tous les tests du fichier
  console.log('Setup global');
});

afterAll(() => {
  // Après tous les tests du fichier
  console.log('Cleanup global');
});

beforeEach(() => {
  // Avant chaque test
});

afterEach(() => {
  // Après chaque test
});
```

---

# Mocking

```javascript
import { vi, test, expect } from 'vitest';

// Mock d'une fonction
const mockFn = vi.fn();
mockFn('arg1');
expect(mockFn).toHaveBeenCalledWith('arg1');

// Mock avec valeur de retour
const mockFn = vi.fn().mockReturnValue(42);
expect(mockFn()).toBe(42);

// Mock async
const mockFn = vi.fn().mockResolvedValue('data');
await expect(mockFn()).resolves.toBe('data');
```

---

# Mocking de modules

```javascript
import { vi, test, expect } from 'vitest';

// Mock d'un module entier
vi.mock('./api.js', () => ({
  fetchUsers: vi.fn().mockResolvedValue([{ id: 1, name: 'John' }]),
}));

import { fetchUsers } from './api.js';
import { UserService } from './userService.js';

test('UserService uses API', async () => {
  const service = new UserService();
  const users = await service.getUsers();

  expect(fetchUsers).toHaveBeenCalled();
  expect(users).toHaveLength(1);
});
```

---

# Tests async

```javascript
// Avec async/await
test('fetches data', async () => {
  const data = await fetchData();
  expect(data).toBeDefined();
});

// Avec .resolves/.rejects
test('resolves to user', async () => {
  await expect(getUser(1)).resolves.toEqual({ id: 1, name: 'John' });
});

test('rejects with error', async () => {
  await expect(getUser(-1)).rejects.toThrow('Invalid ID');
});
```

---

# Test paramétré

```javascript
import { describe, test, expect } from 'vitest';

describe.each([
  { a: 1, b: 2, expected: 3 },
  { a: 5, b: 5, expected: 10 },
  { a: -1, b: 1, expected: 0 },
])('add($a, $b)', ({ a, b, expected }) => {
  test(`returns ${expected}`, () => {
    expect(add(a, b)).toBe(expected);
  });
});
```

**Sortie** :
- add(1, 2) returns 3 ✓
- add(5, 5) returns 10 ✓
- add(-1, 1) returns 0 ✓

---

<!-- _class: lead -->

# Partie 3
## Coverage

---

# Qu'est-ce que le coverage ?

> **Coverage** : Pourcentage de code exécuté pendant les tests

## Types de coverage

| Métrique | Description |
|----------|-------------|
| **Lines** | Lignes de code exécutées |
| **Statements** | Instructions exécutées |
| **Branches** | Chemins if/else couverts |
| **Functions** | Fonctions appelées |

---

# Activer le coverage

```bash
# Installation du provider
npm install -D @vitest/coverage-v8
```

```javascript
// vitest.config.js
export default defineConfig({
  test: {
    coverage: {
      provider: 'v8',
      reporter: ['text', 'html', 'lcov'],
      reportsDirectory: './coverage',
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

---

# Exécuter avec coverage

```bash
npm run test:coverage
```

**Coverage report from v8 :**

| File | % Stmts | % Branch | % Funcs | % Lines |
|------|---------|----------|---------|---------|
| **All files** | **85.71** | **83.33** | **90.00** | **85.71** |
| src/taskManager.js | 92.30 | 100.00 | 100.00 | 92.30 |
| src/utils/date.js | 75.00 | 66.67 | 80.00 | 75.00 |

---

# Rapport HTML

```
coverage/
├── index.html          # Rapport navigable
├── lcov.info           # Format lcov (pour Codecov)
└── src/
    ├── taskManager.js.html
    └── utils/
        └── date.js.html
```

**Ouvrir** : `open coverage/index.html`

![bg right:40% 90%](https://vitest.dev/coverage.png)

---

# Seuils de coverage

```javascript
// vitest.config.js
coverage: {
  thresholds: {
    lines: 70,
    branches: 70,
    functions: 70,
    statements: 70,
  },
}
```

**Si le seuil n'est pas atteint** :
```
ERROR: Coverage threshold for lines (70%) not met: 65%
```

Le build échoue ❌

---

# Exclure du coverage

```javascript
// vitest.config.js
coverage: {
  exclude: [
    'node_modules/',
    'tests/',
    '**/*.test.js',
    '**/*.config.js',
    'src/index.js',     // Point d'entrée
  ],
}
```

```javascript
// Dans le code : ignorer une ligne
/* v8 ignore next */
if (process.env.DEBUG) console.log('debug');
```

---

# Coverage dans CI

```yaml
jobs:
  test:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      - uses: actions/setup-node@v4
        with:
          node-version: '20'
      - run: npm ci
      - run: npm run test:coverage

      - name: Upload coverage
        uses: actions/upload-artifact@v4
        with:
          name: coverage-report
          path: coverage/
```

---

# Codecov integration

```yaml
- name: Upload to Codecov
  uses: codecov/codecov-action@v4
  with:
    token: ${{ secrets.CODECOV_TOKEN }}
    files: ./coverage/lcov.info
    fail_ci_if_error: true
```

**Avantages** :
- 📊 Dashboard en ligne
- 📈 Historique du coverage
- 💬 Commentaires sur PR
- 🚫 Bloquer si coverage diminue

---

<!-- _class: lead -->

# Partie 4
## CI et qualité

---

# Linting dans CI

```yaml
jobs:
  lint:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      - uses: actions/setup-node@v4
      - run: npm ci
      - run: npm run lint
```

## ESLint config pour CI

```json
{
  "scripts": {
    "lint": "eslint src/",
    "lint:fix": "eslint src/ --fix"
  }
}
```

---

# Workflow complet qualité

```yaml
name: Quality

on: [push, pull_request]

jobs:
  quality:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      - uses: actions/setup-node@v4
        with:
          node-version: '20'
      - run: npm ci

      - name: Lint
        run: npm run lint

      - name: Test with coverage
        run: npm run test:coverage

      - name: Build
        run: npm run build
```

---

# Fail fast vs Rapport complet

```yaml
# Option 1: Fail fast (défaut)
- run: npm run lint
- run: npm test      # N'exécute pas si lint échoue

# Option 2: Rapport complet
- name: Lint
  run: npm run lint
  continue-on-error: true

- name: Test
  run: npm test
  continue-on-error: true

- name: Check results
  if: failure()
  run: exit 1
```

---

# Artifacts de test

```yaml
- name: Run tests
  run: npm test -- --reporter=junit --outputFile=results.xml

- name: Upload test results
  uses: actions/upload-artifact@v4
  if: always()          # Même si tests échouent
  with:
    name: test-results
    path: results.xml
    retention-days: 30
```

---

# Cache des dépendances

```yaml
- name: Cache node_modules
  uses: actions/cache@v4
  with:
    path: node_modules
    key: ${{ runner.os }}-node-${{ hashFiles('package-lock.json') }}
    restore-keys: |
      ${{ runner.os }}-node-

- name: Install dependencies
  run: npm ci
```

**Gain** : ~30s → ~5s sur les builds suivants

---

# Setup Node optimisé

```yaml
- uses: actions/setup-node@v4
  with:
    node-version: '20'
    cache: 'npm'          # Cache intégré !

- run: npm ci
```

**Équivalent** au cache manuel, plus simple !

---

<!-- _class: lead -->

# Partie 5
## TP : Tests TaskFlow

---

# TP3 : Tests dans TaskFlow

## Objectifs

1. Écrire des tests unitaires avec Vitest
2. Configurer le coverage (≥ 70%)
3. Intégrer les tests dans le CI
4. Générer et publier le rapport

---

# Structure des tests

```
taskflow/
├── src/
│   ├── taskManager.js
│   └── utils/
│       └── date.js
├── tests/
│   ├── taskManager.test.js
│   └── utils/
│       └── date.test.js
├── vitest.config.js
└── package.json
```

---

# Étape 1 : Configuration Vitest

```javascript
// vitest.config.js
import { defineConfig } from 'vitest/config';

export default defineConfig({
  test: {
    globals: true,
    environment: 'jsdom',
    coverage: {
      provider: 'v8',
      reporter: ['text', 'html', 'lcov'],
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

---

# Étape 2 : Tests TaskManager

```javascript
// tests/taskManager.test.js
import { describe, test, expect, beforeEach } from 'vitest';
import { TaskManager } from '../src/taskManager.js';

describe('TaskManager', () => {
  let manager;

  beforeEach(() => {
    manager = new TaskManager();
  });

  test('starts with empty tasks', () => {
    expect(manager.tasks).toEqual([]);
  });

  test('adds a task', () => {
    manager.addTask('Test task');
    expect(manager.tasks).toHaveLength(1);
  });
});
```

---

# Étape 3 : CI avec coverage

```yaml
# .github/workflows/ci.yml
jobs:
  test:
    runs-on: ubuntu-latest
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
```

---

# Critères de validation

| Critère | Requis |
|---------|--------|
| Tests passent | ✅ |
| Coverage ≥ 70% | ✅ |
| CI vert | ✅ |
| Artifact uploadé | ✅ |

---

# Récapitulatif Jour 3

## Ce qu'on a appris

- ✅ Pyramide des tests
- ✅ Vitest : config, écriture, mocking
- ✅ Coverage et seuils
- ✅ Intégration CI avec artifacts

## Demain (Jour 4)

- Stratégies de branches
- Git Flow et GitHub Flow
- SemVer et releases automatiques

---

<!-- _class: lead -->

# Questions ?

---

# Ressources

## Documentation
- [Vitest](https://vitest.dev/)
- [Testing Best Practices](https://github.com/goldbergyoni/javascript-testing-best-practices)
- [Codecov](https://codecov.io/)

## Projet
- [TaskFlow - Votre fork](https://github.com/YOUR-USER/taskflow)

---

<!-- _class: lead -->

# Merci !

**Jour 4 : Branches et versions**
Git Flow, SemVer, releases
