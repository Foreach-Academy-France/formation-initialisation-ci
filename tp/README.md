# Travaux Pratiques - Projet Fil Rouge TaskFlow

## Concept

Plutôt que 5 TPs indépendants, les étudiants travaillent sur un **projet unique** qu'ils enrichissent chaque jour. Cela simule un vrai workflow DevOps où le CI/CD évolue avec le projet.

## Le projet : TaskFlow

Application web de gestion de tâches (todo list avancée).

**Stack technique :**
- Frontend : Vanilla JS + Vite
- Tests : Vitest
- Linting : ESLint + Prettier
- Container : Docker
- Déploiement : GitHub Pages + ghcr.io

**Voir** : [Documentation complète TaskFlow](./fil-rouge-taskflow/README.md)

---

## Progression

| Jour | Module | Objectif TP | Livrable | Instructions |
|------|--------|-------------|----------|--------------|
| **1** | Introduction CI | Jenkins + GitHub Actions | Comparer CI self-hosted vs cloud | [TP Jour 1](./jour1-premier-workflow.md) |
| **2** | Configuration | Jobs avancés | Matrix + secrets + triggers | *À venir* |
| **3** | Tests | Qualité de code | Tests + coverage ≥ 70% | *À venir* |
| **4** | Branches | Versioning | Branch protection + release v1.0.0 | *À venir* |
| **5** | Déploiement | Pipeline complet | Docker + ghcr.io + GitHub Pages | *À venir* |

---

## Point de départ

Les étudiants **forkent** le repository starter au Jour 1 :

```bash
gh repo fork Foreach-Academy-France/taskflow-starter --clone
cd taskflow-starter
npm install
npm run dev
```

**Repository starter** : https://github.com/Foreach-Academy-France/taskflow-starter

Le starter contient :
- Application TaskFlow fonctionnelle (sans CI)
- Structure de base (src/, tests/, package.json)
- Dockerfile vide (à compléter J5)
- README avec instructions

---

## Évaluation finale (100 points)

L'évaluation se fait sur l'état du repository à la fin du Jour 5 :

| Critère | Points | Comment vérifier |
|---------|--------|------------------|
| Workflow CI passe (badge vert) | 20 | Actions tab |
| Tests passent, coverage ≥ 70% | 20 | Artifact coverage |
| Branch protection sur main | 15 | Settings > Branches |
| Au moins 1 release (v1.x.x) | 15 | Releases tab |
| Image Docker sur ghcr.io | 15 | Packages tab |
| Site déployé sur GitHub Pages | 15 | URL accessible |
| **Total** | **100** | |

**Validation** : ≥ 50 points

---

## TP Bonus

| TP | Sujet | Objectif | Instructions |
|----|-------|----------|--------------|
| **Bonus** | Forge logicielle | Déployer une forge self-hosted Gitea + Woodpecker CI | [TP Bonus](./bonus-forge-gitea-woodpecker.md) |

Ce TP est autonome et ne modifie pas le fil rouge GitHub Actions. Il permet de comparer l'approche cloud (GitHub) avec l'approche self-hosted.

---

## Ressources

- [Starter TaskFlow](./fil-rouge-taskflow/starter/)
- [Solution complète](./fil-rouge-taskflow/solution/) *(accès formateur)*
- [Cheatsheet GitHub Actions](../ressources/cheatsheet.md)
- [Guide Docker + ghcr.io](../ressources/docker-ghcr.md)

---

## Capitalisation

Ce projet réutilise les compétences du cours **Virtualisation et Containerisation avec Docker** :
- Dockerfile multi-stage
- .dockerignore
- Build et push d'images

*Formation CI/CD - ForEach Academy*
