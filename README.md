# Formation DevOps - Initialisation de l'intégration du système CI

> Formation de 5 jours (35h) sur l'intégration continue avec GitHub Actions pour architectes web

**Public**: M2 ESTD - Expert en stratégie et transformation digitale - Architecte Web
**Durée**: 35 heures (5 jours x 7h)
**Institution**: ForEach Academy (certification IEF2I)
**Formateur**: Fabrice Claeys
**Référent IEF2I**: Michael MAVRODIS
**Prérequis**: Admin système et réseaux, Virtualisation et Containerisation avec Docker

---

## Objectifs du cours

Ce cours vise à fournir aux étudiants les compétences nécessaires pour comprendre, configurer et utiliser un système d'intégration continue (CI) pour automatiser les processus de build, de test et de déploiement.

**À l'issue de la formation, les participants seront capables de :**

- Comprendre les concepts clés de l'intégration continue (CI)
- Configurer un système CI pour automatiser les processus de build, test et déploiement
- Acquérir des compétences pratiques pour la gestion des branches, versions et résolution de conflits
- Maîtriser les outils populaires de CI (GitHub Actions, avec comparaison Jenkins/GitLab CI)
- Explorer les meilleures pratiques pour l'intégration continue et l'automatisation des tests

**Compétences visées** : C30, C33, C34, C35

---

## Organisation

**Structure de chaque journée :**
- **Matin (9h-12h15)** : Théorie et démonstrations
- **Après-midi (13h15-17h)** : Travaux pratiques

**1 module = 1 journée**

### Projet fil rouge : TaskFlow

Les étudiants travaillent sur le **même projet** pendant les 5 jours, en l'enrichissant progressivement :

| Jour | Enrichissement du projet |
|------|--------------------------|
| J1 | Fork starter → Premier workflow CI (lint) |
| J2 | Jobs multiples, matrix Node 18/20/22, secrets |
| J3 | Tests unitaires Vitest, coverage ≥ 70%, artifacts |
| J4 | Branch protection, release auto sur tag SemVer |
| J5 | Build Docker → push ghcr.io → deploy GitHub Pages |

**Stack** : Vanilla JS + Vite + Vitest + Docker + GitHub Pages

**Évaluation** : État final du repository TaskFlow (100 points)

---

## Programme de formation détaillé

### Module 1 : Introduction au Continuous Integration (CI)

**Jour 1 (7h)**

| Horaire | Contenu |
|---------|---------|
| **MATIN - Théorie** | |
| 9h00-9h30 | Accueil et introduction |
| 9h30-11h00 | **Principes et avantages du CI** : Qu'est-ce que CI/CD ? Historique, avantages |
| 11h15-12h15 | **Workflow et concepts** : Pipeline, Job, Step, Artifact, Triggers, Runners |
| **APRÈS-MIDI - Pratique** | |
| 13h15-15h00 | **Outils CI populaires** : Jenkins, Travis, CircleCI, GitLab CI, GitHub Actions |
| 15h15-17h00 | **TP1 : Premier workflow GitHub Actions** |

**Slides**: [Jour 1](./slides/jour1-introduction-ci.md) | **TP1**: [Premier workflow](./tp/tp1-premier-workflow/)

---

### Module 2 : Configuration avancée du système CI

**Jour 2 (7h)**

| Horaire | Contenu |
|---------|---------|
| **MATIN - Théorie** | |
| 9h00-10h30 | **Gestion des configurations** : Structure workflow, variables, contextes, secrets |
| 10h45-12h15 | **Jobs de build** : Jobs multiples, parallélisation, matrix builds, dépendances |
| **APRÈS-MIDI - Pratique** | |
| 13h15-15h00 | **Déclencheurs et paramètres** : Triggers, workflow_dispatch, filtres, conditions |
| 15h15-17h00 | **TP2 : Configuration avancée** |

**Slides**: [Jour 2](./slides/jour2-configuration.md) | **TP2**: [Configuration avancée](./tp/tp2-configuration-avancee/)

---

### Module 3 : Automatisation des tests dans le CI

**Jour 3 (7h)**

| Horaire | Contenu |
|---------|---------|
| **MATIN - Théorie** | |
| 9h00-10h30 | **Tests automatisés** : Types de tests, pyramide, frameworks (Jest, Vitest, Cypress) |
| 10h45-12h15 | **Intégration dans CI** : Configuration, exécution parallèle, timeouts, retries |
| **APRÈS-MIDI - Pratique** | |
| 13h15-15h00 | **Coverage et qualité** : Rapports, artifacts, Codecov, linting |
| 15h15-17h00 | **TP3 : Tests automatisés** |

**Slides**: [Jour 3](./slides/jour3-tests.md) | **TP3**: [Tests automatisés](./tp/tp3-tests-automatises/)

---

### Module 4 : Gestion des branches et des versions

**Jour 4 (7h)**

| Horaire | Contenu |
|---------|---------|
| **MATIN - Théorie** | |
| 9h00-10h30 | **Stratégies de branches** : Git Flow, GitHub Flow, Trunk-based development |
| 10h45-12h15 | **CI et branches** : Pull requests, code review, branch protection, merge strategies |
| **APRÈS-MIDI - Pratique** | |
| 13h15-15h00 | **Versions et tags** : SemVer, releases GitHub, changelog automatique |
| 15h15-17h00 | **TP4 : Branches et versions** |

**Slides**: [Jour 4](./slides/jour4-branches-versions.md) | **TP4**: [Branches et versions](./tp/tp4-branches-versions/)

---

### Module 5 : Déploiement et amélioration continue

**Jour 5 (7h)**

| Horaire | Contenu |
|---------|---------|
| **MATIN - Théorie** | |
| 9h00-10h30 | **Stratégies CD** : Delivery vs Deployment, environnements, Blue/Green, Canary |
| 10h45-12h15 | **Automatisation** : Docker, GitHub Environments, Pages/Vercel/Netlify, Ansible |
| **APRÈS-MIDI - Pratique** | |
| 13h15-15h30 | **TP5 : Pipeline de déploiement continu** |
| 15h45-16h30 | **Amélioration continue** : Métriques, optimisation, bonnes pratiques |
| 16h30-17h00 | **QCM Final** |

**Slides**: [Jour 5](./slides/jour5-deploiement.md) | **TP5**: [Pipeline CD](./tp/tp5-pipeline-deploiement/) | **Évaluation**: [QCM](./evaluation/)

---

## Comparaison des outils CI

| Critère | GitHub Actions | Jenkins | GitLab CI |
|---------|----------------|---------|-----------|
| Hébergement | Cloud (GitHub) | Self-hosted | Cloud/Self-hosted |
| Configuration | YAML | Groovy/UI | YAML |
| Intégration Git | Native GitHub | Plugins | Native GitLab |
| Coût | Gratuit (limites) | Gratuit (infra) | Gratuit (limites) |
| Marketplace | 15,000+ actions | 1,800+ plugins | Templates |
| Courbe apprentissage | Facile | Moyenne | Facile |

**Pourquoi GitHub Actions pour ce cours ?**
- Intégration native avec GitHub (déjà utilisé par les étudiants)
- Syntaxe YAML moderne et lisible
- Pas de serveur à installer/maintenir
- Marketplace riche avec actions prêtes à l'emploi
- Concepts transférables aux autres outils CI

---

## Évaluation

| Type | Coefficient |
|------|-------------|
| TP (5 TPs) | 100% |
| QCM | 100% |

**Validation de la compétence** : Note >= 10/20

---

## Ressources

### Documentation officielle
- [GitHub Actions Documentation](https://docs.github.com/en/actions)
- [Workflow syntax reference](https://docs.github.com/en/actions/reference/workflow-syntax-for-github-actions)
- [GitHub Actions Marketplace](https://github.com/marketplace?type=actions)

### Outils recommandés
- **VSCode** + Extension GitHub Actions
- **act** - Exécution locale des workflows
- **GitHub CLI** (gh) - Gestion depuis le terminal

### Comparaison avec Jenkins
- [Jenkins Documentation](https://www.jenkins.io/doc/)
- [Migrating from Jenkins to GitHub Actions](https://docs.github.com/en/actions/migrating-to-github-actions/migrating-from-jenkins-to-github-actions)

---

## Contact

**Formateur ForEach** : Fabrice Claeys
**Référent IEF2I** : Michael MAVRODIS (michaelmavrodis@formateur.ief2i.fr)

---

**2026 - Formation CI/CD - ForEach Academy**
