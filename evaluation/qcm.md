# QCM Final - Formation CI/CD

**Durée** : 30-45 minutes
**Questions** : 30
**Validation** : Note >= 10/20

---

## Section 1 : Concepts CI/CD (6 questions)

### Question 1
Que signifie CI dans CI/CD ?
- A) Code Integration
- B) Continuous Integration
- C) Container Integration
- D) Cloud Infrastructure

### Question 2
Quel est le principal avantage de l'intégration continue ?
- A) Réduire les coûts d'hébergement
- B) Détecter les erreurs plus tôt dans le cycle de développement
- C) Augmenter la taille des équipes
- D) Éliminer le besoin de tests manuels

### Question 3
Quelle est la différence entre Continuous Delivery et Continuous Deployment ?
- A) Aucune différence
- B) Delivery nécessite une approbation manuelle, Deployment est automatique
- C) Deployment est plus lent
- D) Delivery ne concerne que les tests

### Question 4
Qu'est-ce qu'un "pipeline" en CI/CD ?
- A) Un outil de gestion de versions
- B) Une séquence automatisée d'étapes pour build, test et déployer
- C) Un type de conteneur Docker
- D) Un protocole de communication

### Question 5
Lequel n'est PAS un outil de CI/CD ?
- A) Jenkins
- B) GitHub Actions
- C) GitLab CI
- D) Visual Studio Code

### Question 6
Quel outil CI nécessite un serveur self-hosted ?
- A) GitHub Actions
- B) Jenkins
- C) Travis CI (cloud)
- D) CircleCI (cloud)

---

## Section 2 : Configuration CI (6 questions)

### Question 7
Dans GitHub Actions, où doivent être placés les fichiers workflow ?
- A) À la racine du projet
- B) Dans `.github/workflows/`
- C) Dans `.actions/`
- D) Dans `workflows/`

### Question 8
Dans Jenkins, quel fichier définit le pipeline as code ?
- A) workflow.yml
- B) Jenkinsfile
- C) pipeline.json
- D) .jenkins

### Question 9
Quelle syntaxe définit une variable d'environnement dans GitHub Actions ?
- A) `${MY_VAR}`
- B) `$MY_VAR`
- C) `${{ env.MY_VAR }}`
- D) `%MY_VAR%`

### Question 10
Qu'est-ce qu'un "job" dans un système CI ?
- A) Un fichier de configuration
- B) Une unité de travail indépendante dans un pipeline
- C) Un type de test
- D) Un serveur de build

### Question 11
Comment définir un déclencheur sur push dans GitHub Actions ?
- A) `trigger: push`
- B) `on: push`
- C) `when: push`
- D) `event: push`

### Question 12
Quelle est la fonction d'un "runner" en CI ?
- A) Écrire les tests
- B) Exécuter les jobs du pipeline
- C) Gérer les branches Git
- D) Déployer en production

---

## Section 3 : Tests automatisés (5 questions)

### Question 13
Dans la pyramide des tests, quel type de test est le plus rapide à exécuter ?
- A) Tests end-to-end
- B) Tests d'intégration
- C) Tests unitaires
- D) Tests manuels

### Question 14
Qu'est-ce que le "code coverage" ?
- A) Le nombre de lignes de code
- B) Le pourcentage de code exécuté par les tests
- C) La documentation du code
- D) Le nombre de développeurs

### Question 15
Comment uploader un rapport de tests comme artifact dans GitHub Actions ?
- A) `uses: actions/upload-tests@v4`
- B) `uses: actions/upload-artifact@v4`
- C) `run: upload tests`
- D) `artifact: upload`

### Question 16
Si un test échoue dans le CI, que se passe-t-il par défaut ?
- A) Le pipeline continue
- B) Le pipeline échoue et s'arrête
- C) Un email est envoyé
- D) Le test est ignoré

### Question 17
Quel framework de test JavaScript est couramment utilisé ?
- A) JUnit
- B) Jest
- C) PyTest
- D) RSpec

---

## Section 4 : Gestion des branches (5 questions)

### Question 18
Dans Git Flow, quelle branche contient le code de production ?
- A) develop
- B) feature
- C) main/master
- D) release

### Question 19
Qu'est-ce qu'une "pull request" (ou merge request) ?
- A) Une demande de suppression de branche
- B) Une demande d'intégration de code avec review
- C) Un type de déploiement
- D) Un rapport de tests

### Question 20
Qu'est-ce que le "Semantic Versioning" (SemVer) ?
- A) Un système de nommage de branches
- B) Un format de version MAJOR.MINOR.PATCH
- C) Un outil de CI
- D) Un type de test

### Question 21
Dans GitHub, qu'est-ce qu'une "branch protection rule" ?
- A) Un backup de branche
- B) Des règles pour protéger une branche (reviews obligatoires, etc.)
- C) Un type de merge
- D) Un workflow automatique

### Question 22
Quelle stratégie de merge conserve un historique linéaire ?
- A) Merge commit
- B) Squash and merge
- C) Rebase and merge
- D) Les trois

---

## Section 5 : Déploiement continu (5 questions)

### Question 23
Qu'est-ce qu'un déploiement "Blue/Green" ?
- A) Déployer sur deux serveurs de couleurs différentes
- B) Maintenir deux environnements identiques et basculer entre eux
- C) Un déploiement progressif
- D) Un type de test

### Question 24
Qu'est-ce qu'un "environment" dans GitHub Actions ?
- A) Le système d'exploitation du runner
- B) Un contexte de déploiement avec ses propres secrets et règles
- C) Les variables d'environnement
- D) Le workspace du job

### Question 25
Quel outil est mentionné dans le syllabus pour l'automatisation des déploiements ?
- A) Terraform
- B) Ansible
- C) Puppet
- D) Salt

### Question 26
Comment protéger un déploiement en production ?
- A) Utiliser des mots de passe faibles
- B) Configurer des reviewers obligatoires et des wait timers
- C) Déployer manuellement
- D) Désactiver les tests

### Question 27
Quel est l'avantage d'un déploiement "Canary" ?
- A) Déployer sur tous les serveurs en même temps
- B) Déployer progressivement sur un petit pourcentage d'utilisateurs
- C) Ne jamais déployer
- D) Déployer uniquement la nuit

---

## Section 6 : Bonnes pratiques (3 questions)

### Question 28
Pourquoi utiliser des secrets plutôt que des variables en clair ?
- A) Les secrets sont plus rapides
- B) Les secrets sont chiffrés et masqués dans les logs
- C) Les secrets sont obligatoires
- D) Les variables ne fonctionnent pas

### Question 29
Quelle bonne pratique permet d'accélérer les builds ?
- A) Désactiver les tests
- B) Utiliser le cache des dépendances
- C) Exécuter sur un seul runner
- D) Ignorer les erreurs

### Question 30
Pourquoi versionner les actions utilisées (ex: `@v4` plutôt que `@main`) ?
- A) Pour avoir les dernières fonctionnalités
- B) Pour garantir la reproductibilité et la stabilité
- C) Parce que c'est obligatoire
- D) Pour réduire les coûts

---

*Formation CI/CD - ForEach Academy*
