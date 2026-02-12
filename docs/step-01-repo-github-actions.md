# Étape 1 : Configuration du Repository et GitHub Actions

## Objectif de l'étape

Mettre en place une fondation solide pour la CI/CD de BobApp en créant un workflow GitHub Actions minimal mais extensible. Cette première étape établit une base propre qui sera enrichie dans les étapes suivantes avec l'analyse de qualité (Sonar), la couverture de code, et la containerisation Docker.

## Ce que j'ai trouvé dans le repo

### Arborescence globale

```
v1_projet_cicd/
├── .github/
│   └── workflows/
│       └── ci.yml              # Nouveau workflow CI/CD
├── back/                       # Backend Spring Boot
│   ├── src/
│   │   ├── main/java/          # Code source
│   │   └── test/java/          # Tests unitaires
│   ├── pom.xml                 # Configuration Maven
│   ├── mvnw / mvnw.cmd         # Maven Wrapper
│   └── Dockerfile              # Pour containerisation future
├── front/                      # Frontend Angular
│   ├── src/
│   │   ├── app/                # Code source Angular
│   │   └── test.ts             # Configuration tests
│   ├── package.json            # Dépendances npm
│   ├── package-lock.json       # Lock file npm
│   ├── angular.json            # Configuration Angular CLI
│   ├── karma.conf.js           # Configuration tests Karma
│   └── Dockerfile              # Pour containerisation future
├── docs/                       # Documentation (nouveau)
└── README.md                   # Documentation existante
```

### Commandes importantes repérées

**Backend (Maven + Spring Boot):**
- `mvn clean install` : Build complet avec tests
- `mvn spring-boot:run` : Lancer l'application en local
- `mvn test` : Exécuter uniquement les tests

**Frontend (npm + Angular):**
- `npm install` : Installation des dépendances
- `npm run start` : Serveur de développement (port 4200 par défaut)
- `npm run build` : Build de production
- `npm run test` : Exécuter les tests avec Karma

## Workflow CI v1

### Déclencheurs

```yaml
on:
  pull_request:
    branches: [ main, develop ]
  push:
    branches: [ main ]
```

**Pourquoi ce choix :**
- **`pull_request`** : Valide chaque PR avant merge (essentiel pour la qualité)
- **`push` sur `main`** : Garantit que la branche principale reste toujours verte
- Support de `develop` en PR : permet d'avoir une branche d'intégration si besoin

**Alternative considérée :**
- Déclencher aussi sur tous les pushs de toutes les branches → rejeté car trop coûteux en runners et inutile pour les branches de travail personnelles

**Impact :**
- Chaque PR doit passer le CI avant merge
- Feedback rapide aux développeurs (< 5 min attendu)
- Protection de la branche main

### Découpage des jobs

**3 jobs distincts :**

1. **`backend`** : Build et test du backend Spring Boot
2. **`frontend`** : Build et test du frontend Angular  
3. **`summary`** : Agrégation des résultats (dépend de backend + frontend)

**Pourquoi ce choix :**
- **Parallélisation** : backend et frontend s'exécutent simultanément → gain de temps (~50%)
- **Isolation** : un problème frontend ne bloque pas le backend et vice-versa
- **Lisibilité** : logs séparés, plus facile de diagnostiquer les échecs
- **Extensibilité** : facile d'ajouter d'autres jobs (Sonar, Docker) plus tard

**Alternative considérée :**
- Un seul job séquentiel → rejeté car temps d'exécution doublé et logs mélangés

**Impact :**
- Temps total : ~5 min au lieu de ~10 min (estimation)
- Meilleure UX pour les développeurs

### Choix des versions

**Backend :**
- **Java 11** (JDK Temurin)
  - **Pourquoi :** Version spécifiée dans `pom.xml` (`<java.version>11</java.version>`)
  - **Distribution Temurin :** Distribution LTS gratuite, maintenue par Adoptium, recommandée par la communauté
  - **Alternative :** OpenJDK Zulu → moins populaire dans Actions

**Frontend :**
- **Node.js 16**
  - **Pourquoi :** Angular 14.2.0 nécessite Node 14.x minimum, mais 16 est une version LTS stable compatible
  - **Alternative considérée :** Node 14 (minimum) → rejeté car proche de fin de vie
  - **Alternative considérée :** Node 18 → pas testé avec Angular 14, risque de breaking changes
  - **Note :** Pas de `.nvmrc` dans le repo, donc choix basé sur compatibilité Angular

**Impact :**
- Reproductibilité : mêmes versions en CI et en local (si devs suivent la doc)
- Stabilité : versions LTS, support long terme

### Stratégie de cache

**Backend (Maven) :**
```yaml
cache: 'maven'
```
- Cache automatique de `~/.m2/repository` par `actions/setup-java@v4`
- Évite de re-télécharger toutes les dépendances Maven à chaque run
- **Gain estimé :** 1-2 minutes par exécution

**Frontend (npm) :**
```yaml
cache: 'npm'
cache-dependency-path: './front/package-lock.json'
```
- Cache automatique de `~/.npm` par `actions/setup-node@v4`
- Utilise le `package-lock.json` comme clé de cache
- **Gain estimé :** 30-60 secondes par exécution

**Pourquoi ce choix :**
- Actions officielles GitHub gèrent le cache intelligemment (clé basée sur hash des lock files)
- Invalidation automatique si dépendances changent
- Simple à maintenir (pas de configuration YAML complexe)

**Alternative considérée :**
- `actions/cache@v3` manuel → rejeté car plus verbeux et déjà intégré dans les actions setup

**Impact :**
- Réduction de ~50% du temps d'installation des dépendances
- Moins de charge sur les registres npm/Maven Central

### Étapes de résumé (GITHUB_STEP_SUMMARY)

Chaque job génère un résumé Markdown qui s'affiche dans l'onglet Summary de l'exécution :

- **Backend :** Statut des tests, présence de rapports Surefire
- **Frontend :** Statut du build, répertoire dist créé
- **Summary :** Tableau récapitulatif des statuts backend/frontend

**Pourquoi ce choix :**
- Feedback visuel immédiat sans ouvrir les logs détaillés
- Utilise `if: always()` pour afficher même en cas d'échec
- Format Markdown enrichi (emojis, tableaux)

## Comment exécuter localement l'équivalent

### Prérequis
- **Java 11** (vérifier : `java -version`)
- **Maven 3.6+** (inclus via `mvnw`)
- **Node.js 16** (vérifier : `node -v`)
- **npm 7+** (vérifier : `npm -v`)

### Backend

```bash
cd back

# Installation et build complet avec tests
./mvnw clean install

# Ou si Maven global est installé
mvn clean install

# Uniquement les tests
./mvnw test

# Lancer l'application
./mvnw spring-boot:run
```

**Note :** Le `mvnw` (Maven Wrapper) garantit l'utilisation d'une version Maven fixe. Préférable pour la reproductibilité.

### Frontend

```bash
cd front

# Installation des dépendances (équivalent CI)
npm ci

# Build de production (comme CI)
npm run build -- --configuration=production

# Tests headless (comme CI)
npm run test -- --watch=false --browsers=ChromeHeadless

# Pour développement local
npm install          # Installation classique
npm run start        # Dev server avec hot-reload
npm run test         # Tests avec watch mode
```

**Différence `npm install` vs `npm ci` :**
- `npm ci` : Installation stricte basée sur `package-lock.json` (CI/CD)
- `npm install` : Peut mettre à jour le lock file (développement local)

### Vérifier que tout fonctionne

```bash
# Backend : vérifier les tests
cd back && ./mvnw test
# ✅ Doit afficher "BUILD SUCCESS" avec les tests exécutés

# Frontend : vérifier le build
cd front && npm ci && npm run build
# ✅ Doit créer le répertoire dist/bobapp/

# Frontend : vérifier les tests
cd front && npm run test -- --watch=false --browsers=ChromeHeadless
# ✅ Doit afficher "Executed X of X SUCCESS"
```

## Limites connues / dettes techniques

### Tests

**Backend :**
- ✅ Tests présents : `BobappApplicationTests.java`
- ⚠️ **Couverture inconnue** : JaCoCo configuré dans `pom.xml` mais pas encore exploité
- ⚠️ Pas de tests d'intégration visibles au-delà des tests Spring Boot basiques

**Frontend :**
- ✅ Tests présents : `app.component.spec.ts`, `jokes.service.spec.ts`
- ⚠️ **Couverture inconnue** : Karma configuré mais rapport de couverture pas publié
- ⚠️ Tests dépendent de ChromeHeadless → nécessite installation Chrome en CI (géré par `ubuntu-latest`)

### Points bloquants possibles

1. **Tests frontend en échec potentiel :**
   - Les tests Angular nécessitent Chrome/Chromium
   - `ubuntu-latest` inclut Chrome mais versions peuvent diverger
   - **Mitigation actuelle :** Utilisation de `ChromeHeadless` standard

2. **Pas de validation de qualité de code :**
   - Pas de linter forcé dans le workflow
   - Pas de vérification de formatage
   - **Mitigation :** Prévu pour étape 2 avec Sonar

3. **Dépendances non auditées :**
   - Pas de scan de vulnérabilités (`npm audit`, Dependabot)
   - **Mitigation :** À ajouter dans étapes futures

4. **Build temps potentiellement long :**
   - Premier run sans cache : ~10 min estimé
   - **Mitigation :** Cache Maven/npm actif pour runs suivants

5. **Pas de notification d'échec :**
   - Workflow échoue mais pas d'alerte Slack/Email
   - **Mitigation :** Notifications GitHub natives suffisantes pour l'instant

### Dettes techniques identifiées

- **Backend :** Double déclaration `spring-boot-maven-plugin` dans `pom.xml` (lignes 33 et 38) → à nettoyer
- **Frontend :** Configuration `browserTarget` dans `angular.json` référence "project-name" au lieu de "bobapp" (ligne 72) → sans impact mais inconsistant
- **Général :** Fichiers `.DS_Store` commités (macOS metadata) → ajouter au `.gitignore`

## Prochaines évolutions prévues (Étape 2+)

### Étape 2 : Qualité et couverture de code

**Sonar :**
- Intégration SonarCloud ou SonarQube
- Analyse statique automatique (bugs, vulnérabilités, code smells)
- Quality Gate obligatoire pour merge

**Coverage :**
- Publication des rapports JaCoCo (backend)
- Publication des rapports Karma Coverage (frontend)
- Seuils minimums de couverture (ex: 80%)
- Badges dans README

**Linters :**
- ESLint pour frontend (si pas déjà en place)
- Checkstyle ou PMD pour backend
- Formatage automatique (Prettier, Google Java Format)

### Étape 3 : Containerisation et registry

**Docker Build :**
- Build des images Docker pour front et back
- Multi-stage builds pour optimiser la taille
- Tagging sémantique (version, commit SHA, latest)

**Docker Hub :**
- Push automatique des images sur Docker Hub (ou autre registry)
- Authentification sécurisée via secrets GitHub
- Push uniquement sur `main` pour les tags stables

**Docker Compose :**
- Configuration pour lancer l'ensemble du stack en local
- Utile pour tests d'intégration et environnements de dev

### Étape 4+ : Déploiement et production

- Déploiement automatique sur environnement de staging
- Tests end-to-end (Cypress, Selenium)
- Monitoring et alerting
- Rollback automatique en cas d'échec

---

## Validation

✅ **Workflow créé :** `.github/workflows/ci.yml` existe et est valide YAML  
✅ **Déclencheurs configurés :** `pull_request` et `push` sur `main`  
✅ **Jobs séparés :** backend et frontend parallélisés  
✅ **Cache activé :** Maven et npm  
✅ **Versions verrouillées :** Java 11, Node 16  
✅ **Résumés configurés :** `GITHUB_STEP_SUMMARY` dans chaque job  
✅ **Documentation complète :** Ce fichier + `front-overview.md` + `back-overview.md`

**Prochaine étape :** Créer une PR pour tester le workflow en conditions réelles.
