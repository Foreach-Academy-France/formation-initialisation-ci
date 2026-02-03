# Grille d'Évaluation des TPs - Formation CI/CD

## Critères généraux

| Critère | Poids | Description |
|---------|-------|-------------|
| Fonctionnalité | 40% | Le workflow fonctionne comme attendu |
| Bonnes pratiques | 30% | Respect des conventions et patterns recommandés |
| Qualité du code | 20% | YAML propre, lisible, bien structuré |
| Documentation | 10% | Commentaires et README si demandé |

---

## TP1 : Premier Workflow (10%)

### Objectifs
- Créer un workflow basique
- Comprendre la structure YAML
- Déclencher sur push

### Barème détaillé

| Élément | Points | Critères |
|---------|--------|----------|
| Structure valide | 2 | Fichier YAML valide dans `.github/workflows/` |
| Trigger push | 1 | `on: push` configuré correctement |
| Job défini | 2 | Job avec `runs-on` et steps |
| Steps fonctionnels | 3 | Checkout + au moins une commande |
| Bonnes pratiques | 2 | Nommage, indentation |
| **Total** | **10** | |

---

## TP2 : Multi-jobs et Matrix (10%)

### Objectifs
- Créer plusieurs jobs
- Utiliser une matrix
- Gérer les dépendances

### Barème détaillé

| Élément | Points | Critères |
|---------|--------|----------|
| Multi-jobs | 2 | Au moins 2 jobs distincts |
| Matrix | 3 | Matrix avec au moins 2 dimensions |
| Dépendances | 2 | `needs` utilisé correctement |
| Conditions | 1 | Au moins une condition `if` |
| Fonctionnement | 2 | Tous les jobs passent |
| **Total** | **10** | |

---

## TP3 : Pipeline de Tests (10%)

### Objectifs
- Intégrer des tests unitaires
- Configurer le linting
- Uploader les rapports

### Barème détaillé

| Élément | Points | Critères |
|---------|--------|----------|
| Tests unitaires | 3 | Tests exécutés dans le workflow |
| Linting | 2 | ESLint ou Prettier configuré |
| Coverage | 2 | Rapport de coverage généré |
| Artifacts | 2 | Upload des rapports en artifact |
| Échec correct | 1 | Workflow échoue si tests échouent |
| **Total** | **10** | |

---

## TP4 : Build et Cache (10%)

### Objectifs
- Builder une application
- Configurer le cache
- Optimiser les temps

### Barème détaillé

| Élément | Points | Critères |
|---------|--------|----------|
| Build fonctionnel | 3 | Application buildée correctement |
| Cache configuré | 3 | Cache des dépendances actif |
| Artifact de build | 2 | Build uploadé en artifact |
| Optimisation | 1 | Temps de build < 5 min |
| Docker (bonus) | 1 | Build Docker fonctionnel |
| **Total** | **10** | |

---

## TP5 : Projet Final (10%)

### Objectifs
- Pipeline CI/CD complet
- Tests + Build + Deploy
- Bonnes pratiques

### Barème détaillé

| Élément | Points | Critères |
|---------|--------|----------|
| Pipeline complet | 3 | Tests → Build → Deploy enchaînés |
| Tests intégrés | 2 | Au moins unit tests |
| Déploiement | 2 | GitHub Pages ou autre target |
| Secrets utilisés | 1 | Secrets pour données sensibles |
| Documentation | 1 | README expliquant le pipeline |
| Qualité globale | 1 | Code propre, bonnes pratiques |
| **Total** | **10** | |

---

## Notation globale

| Note | Appréciation |
|------|--------------|
| 18-20 | Excellent - Maîtrise complète |
| 15-17 | Très bien - Bonne compréhension |
| 12-14 | Bien - Compétences acquises |
| 10-11 | Satisfaisant - Minimum requis atteint |
| < 10 | Insuffisant - Compétence non validée |

---

## Critères de bonnes pratiques

### À vérifier systématiquement

- [ ] Versions fixes pour les actions (`@v4` pas `@main`)
- [ ] Permissions minimales définies
- [ ] Secrets utilisés pour les données sensibles
- [ ] Cache configuré quand applicable
- [ ] Noms descriptifs pour jobs et steps
- [ ] Concurrency configurée pour éviter les conflits
- [ ] YAML bien indenté et lisible

### Pénalités possibles

| Erreur | Pénalité |
|--------|----------|
| Secrets en clair dans le workflow | -2 points |
| Permissions trop larges | -1 point |
| YAML mal formaté | -1 point |
| Pas de commentaires sur code complexe | -0.5 point |

---

*Formation CI/CD - ForEach Academy*
