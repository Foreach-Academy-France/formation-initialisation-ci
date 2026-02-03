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

## Programme de formation détaillé

### Module 1 : Introduction au Continuous Integration (CI)

**Jour 1 - Fondamentaux (7h)**

| Horaire | Contenu | Durée |
|---------|---------|-------|
| 9h00-9h30 | Accueil et introduction | 30min |
| 9h30-11h00 | **Principes et avantages du CI** | 1h30 |
| | - Qu'est-ce que l'intégration continue ? | |
| | - Historique : de l'intégration manuelle au CI/CD | |
| | - Avantages : détection précoce, feedback rapide | |
| 11h00-11h15 | Pause | 15min |
| 11h15-12h15 | **Workflow et concepts CI** | 1h |
| | - Pipeline, Job, Step, Artifact | |
| | - Triggers et événements | |
| | - Runners et environnements d'exécution | |
| 12h15-13h15 | Pause déjeuner | 1h |
| 13h15-15h00 | **Outils et technologies CI populaires** | 1h45 |
| | - Panorama : Jenkins, Travis CI, CircleCI, GitLab CI | |
| | - Focus GitHub Actions : avantages et écosystème | |
| | - Démo : Interface GitHub Actions | |
| 15h00-15h15 | Pause | 15min |
| 15h15-17h00 | **TP1 : Premier workflow GitHub Actions** | 1h45 |
| | - Création du premier workflow | |
| | - Structure YAML et syntaxe de base | |
| | - Comprendre les logs et le debugging | |

**Slides**: [Jour 1 - Introduction CI](./slides/jour1-introduction-ci.md)
**TP1**: [Premier workflow](./tp/tp1-premier-workflow/)

---

### Module 2 : Configuration avancée du système CI

**Jour 2 - Configuration (7h)**

| Horaire | Contenu | Durée |
|---------|---------|-------|
| 9h00-10h30 | **Gestion des configurations** | 1h30 |
| | - Structure des fichiers workflow | |
| | - Variables et contextes GitHub Actions | |
| | - Secrets et données sensibles | |
| 10h30-10h45 | Pause | 15min |
| 10h45-12h15 | **Création de projets CI et jobs de build** | 1h30 |
| | - Jobs multiples et parallélisation | |
| | - Matrix builds (multi-versions, multi-OS) | |
| | - Dépendances entre jobs (needs) | |
| 12h15-13h15 | Pause déjeuner | 1h |
| 13h15-15h00 | **Déclencheurs et paramètres** | 1h45 |
| | - Triggers : push, pull_request, schedule | |
| | - workflow_dispatch (déclenchement manuel) | |
| | - Filtres sur branches et paths | |
| | - Conditions d'exécution (if) | |
| 15h00-15h15 | Pause | 15min |
| 15h15-17h00 | **TP2 : Configuration avancée** | 1h45 |
| | - Projet CI avec jobs de build multiples | |
| | - Matrix de versions Node.js | |
| | - Déclencheurs et conditions | |

**Slides**: [Jour 2 - Configuration](./slides/jour2-configuration.md)
**TP2**: [Configuration avancée](./tp/tp2-configuration-avancee/)

---

### Module 3 : Automatisation des tests dans le CI

**Jour 3 - Tests (7h)**

| Horaire | Contenu | Durée |
|---------|---------|-------|
| 9h00-10h30 | **Introduction aux tests automatisés** | 1h30 |
| | - Types de tests : unitaires, intégration, e2e | |
| | - Pyramide des tests | |
| | - Frameworks de tests (Jest, Vitest, Cypress) | |
| 10h30-10h45 | Pause | 15min |
| 10h45-12h15 | **Intégration des frameworks de tests** | 1h30 |
| | - Configuration des tests dans le CI | |
| | - Exécution parallèle des tests | |
| | - Gestion des timeouts et retries | |
| 12h15-13h15 | Pause déjeuner | 1h |
| 13h15-15h00 | **Tests unitaires et fonctionnels** | 1h45 |
| | - Code coverage et rapports | |
| | - Upload des artifacts de test | |
| | - Intégration avec Codecov/Coveralls | |
| | - Linting et qualité de code | |
| 15h00-15h15 | Pause | 15min |
| 15h15-17h00 | **TP3 : Tests automatisés** | 1h45 |
| | - Configuration tests unitaires | |
| | - Génération de rapports de coverage | |
| | - Linting automatique | |

**Slides**: [Jour 3 - Tests](./slides/jour3-tests.md)
**TP3**: [Tests automatisés](./tp/tp3-tests-automatises/)

---

### Module 4 : Gestion des branches et des versions

**Jour 4 - Branches et versions (7h)**

| Horaire | Contenu | Durée |
|---------|---------|-------|
| 9h00-10h30 | **Stratégies de gestion des branches** | 1h30 |
| | - Git Flow : feature, develop, release, hotfix | |
| | - GitHub Flow : simplicité et déploiement continu | |
| | - Trunk-based development | |
| 10h30-10h45 | Pause | 15min |
| 10h45-12h15 | **Résolution des conflits et CI** | 1h30 |
| | - Pull requests et code review | |
| | - Branch protection rules | |
| | - Status checks obligatoires | |
| | - Merge strategies (merge, squash, rebase) | |
| 12h15-13h15 | Pause déjeuner | 1h |
| 13h15-15h00 | **Gestion des versions et tags** | 1h45 |
| | - Semantic versioning (SemVer) | |
| | - Tags Git et releases GitHub | |
| | - Changelog automatique | |
| | - Actions de release automatique | |
| 15h00-15h15 | Pause | 15min |
| 15h15-17h00 | **TP4 : Branches et versions** | 1h45 |
| | - Mise en place Git Flow | |
| | - Branch protection et status checks | |
| | - Release automatique avec tags | |

**Slides**: [Jour 4 - Branches](./slides/jour4-branches-versions.md)
**TP4**: [Branches et versions](./tp/tp4-branches-versions/)

---

### Module 5 : Déploiement et amélioration continue

**Jour 5 - Déploiement (7h)**

| Horaire | Contenu | Durée |
|---------|---------|-------|
| 9h00-10h30 | **Stratégies de déploiement continu (CD)** | 1h30 |
| | - Continuous Delivery vs Continuous Deployment | |
| | - Environnements : staging, production | |
| | - Blue/Green, Rolling, Canary deployments | |
| 10h30-10h45 | Pause | 15min |
| 10h45-12h15 | **Automatisation des déploiements** | 1h30 |
| | - Déploiement avec Docker | |
| | - GitHub Environments et protection rules | |
| | - Déploiement sur GitHub Pages, Vercel, Netlify | |
| | - Introduction à Ansible pour le déploiement | |
| 12h15-13h15 | Pause déjeuner | 1h |
| 13h15-15h30 | **TP5 : Pipeline de déploiement continu** | 2h15 |
| | - Pipeline complet : Test → Build → Deploy | |
| | - Déploiement multi-environnements | |
| | - Rollback et monitoring | |
| 15h30-15h45 | Pause | 15min |
| 15h45-16h30 | **Amélioration continue et rétroaction** | 45min |
| | - Métriques CI/CD | |
| | - Optimisation des pipelines | |
| | - Bonnes pratiques et patterns | |
| 16h30-17h00 | **QCM Final** | 30min |

**Slides**: [Jour 5 - Déploiement](./slides/jour5-deploiement.md)
**TP5**: [Pipeline CD](./tp/tp5-pipeline-deploiement/)
**Évaluation**: [QCM + Grille](./evaluation/)

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
