# Grille d'Évaluation - Projet Fil Rouge TaskFlow

## Évaluation finale (100 points)

L'évaluation porte sur l'état du repository TaskFlow à la fin du Jour 5.

---

## Critères d'évaluation

### 1. Workflow CI fonctionnel (20 points)

| Élément | Points | Vérification |
|---------|--------|--------------|
| Fichier `ci.yml` présent et valide | 5 | `.github/workflows/ci.yml` |
| Workflow se déclenche sur push/PR | 5 | Events configurés |
| Jobs lint + test + build | 5 | Jobs définis |
| Badge CI vert sur main | 5 | Actions tab |

### 2. Tests et coverage (20 points)

| Élément | Points | Vérification |
|---------|--------|--------------|
| Au moins 5 tests unitaires | 5 | `tests/*.test.js` |
| Tests passent | 5 | Job test vert |
| Coverage ≥ 70% | 5 | Rapport coverage |
| Artifact coverage uploadé | 5 | Artifacts tab |

### 3. Branch protection (15 points)

| Élément | Points | Vérification |
|---------|--------|--------------|
| Protection activée sur main | 5 | Settings > Branches |
| Require PR before merge | 5 | Rule configurée |
| Require status checks (CI) | 5 | Rule configurée |

### 4. Release et versioning (15 points)

| Élément | Points | Vérification |
|---------|--------|--------------|
| Workflow release sur tag | 5 | `release.yml` |
| Au moins 1 tag SemVer (v1.x.x) | 5 | Tags |
| Release GitHub créée | 5 | Releases tab |

### 5. Image Docker (15 points)

| Élément | Points | Vérification |
|---------|--------|--------------|
| Dockerfile fonctionnel | 5 | Build sans erreur |
| Image poussée sur ghcr.io | 5 | Packages tab |
| Tag versionné (v1.x.x) | 5 | Package versions |

### 6. Déploiement GitHub Pages (15 points)

| Élément | Points | Vérification |
|---------|--------|--------------|
| Workflow deploy configuré | 5 | `deploy.yml` |
| Site accessible | 5 | URL GitHub Pages |
| Déploiement automatique sur main | 5 | Workflow history |

---

## Récapitulatif

| Critère | Points |
|---------|--------|
| Workflow CI | /20 |
| Tests et coverage | /20 |
| Branch protection | /15 |
| Release et versioning | /15 |
| Image Docker | /15 |
| Déploiement | /15 |
| **TOTAL** | **/100** |

---

## Barème de validation

| Note | Résultat |
|------|----------|
| ≥ 80 | Excellent |
| 65-79 | Bien |
| 50-64 | Validé |
| < 50 | Non validé |

**Seuil de validation** : 50 points minimum

---

## Bonus (hors barème)

| Élément bonus | Points |
|---------------|--------|
| Environnement staging séparé | +5 |
| Changelog automatique | +5 |
| Badge coverage dans README | +2 |
| Tests e2e (Playwright/Cypress) | +5 |
| Dependabot configuré | +3 |

---

## Checklist formateur

Pour chaque étudiant, vérifier :

- [ ] Repository forké et accessible
- [ ] Actions tab : workflows visibles
- [ ] Dernier commit sur main : CI passe
- [ ] Settings > Branches : protection active
- [ ] Releases : au moins v1.0.0
- [ ] Packages : image Docker présente
- [ ] GitHub Pages : site accessible

---

*Formation CI/CD - ForEach Academy*
