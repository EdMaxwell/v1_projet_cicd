# Step 02 : Tests et Couverture de Code

## 1. Objectif de l'étape

Améliorer la pipeline CI pour :
- Exécuter les tests backend (Java/Maven) et frontend (Angular/Karma)
- Générer des rapports de couverture de code :
  - Backend : JaCoCo (HTML + XML)
  - Frontend : Angular/Karma coverage (HTML + LCOV)
- Publier des rapports de tests lisibles dans GitHub Actions via l'action "Test Reporting"
- Uploader les artefacts (rapports) pour consultation ultérieure

⚠️ **Important** : Pas d'intégration SonarQube ni de build/push Docker à cette étape.

---

## 2. État initial

### Workflow existant
Le workflow `.github/workflows/ci.yml` initial comportait :
- Deux jobs séparés (`backend` et `frontend`) + un job `summary`
- **Backend** :
  - `mvn clean install -B` pour construire l'application
  - `mvn test -B` pour exécuter les tests (tests lancés **deux fois** inutilement)
  - Pas de génération de rapport de couverture
  - Pas d'upload d'artefacts
- **Frontend** :
  - Build de l'application Angular
  - `npm test` sans flag `--code-coverage`
  - Pas de rapport JUnit XML
  - Pas d'upload d'artefacts

### Limites identifiées
1. Tests backend exécutés deux fois (gaspillage de temps CI)
2. Absence de rapports de couverture de code
3. Pas de rapports JUnit XML côté frontend (impossibilité d'utiliser "Test Reporting")
4. Aucun artefact uploadé en cas d'échec (diagnostic difficile)
5. GitHub Step Summary limité (pas d'infos sur la couverture)

---

## 3. Back-end : Tests + JaCoCo

### Modifications apportées

#### A) Configuration Maven (`back/pom.xml`)
- **Supprimé** : Duplication du plugin Spring Boot Maven (présent deux fois)
- **Mis à jour** : Configuration JaCoCo
  - Version : `0.8.5` → `0.8.8` (version plus stable et récente)
  - Phase du rapport : `test` → `verify` (génération après l'exécution des tests)
  - Ajout de l'exécution `prepare-agent` avec un ID explicite

```xml
<plugin>
    <groupId>org.jacoco</groupId>
    <artifactId>jacoco-maven-plugin</artifactId>
    <version>0.8.8</version>
    <executions>
        <execution>
            <id>prepare-agent</id>
            <goals>
                <goal>prepare-agent</goal>
            </goals>
        </execution>
        <execution>
            <id>report</id>
            <phase>verify</phase>
            <goals>
                <goal>report</goal>
            </goals>
        </execution>
    </executions>
</plugin>
```

#### B) Workflow CI (`.github/workflows/ci.yml`)
- **Remplacé** les deux commandes Maven par une seule :
  ```bash
  mvn clean verify -B
  ```
  Cette commande :
  - Nettoie le projet (`clean`)
  - Compile le code
  - Exécute les tests (phase `test`)
  - Génère le rapport JaCoCo (phase `verify`)
  - Évite la double exécution des tests

- **Ajouté** l'upload des artefacts :
  - Rapports JaCoCo : `back/target/site/jacoco/`
  - Rapports de tests Surefire : `back/target/surefire-reports/*.xml`
  - Retention : 30 jours
  - Condition : `if: always()` (upload même en cas d'échec)

- **Ajouté** Test Reporting avec `dorny/test-reporter@v1` :
  - Path : `back/target/surefire-reports/*.xml`
  - Reporter : `java-junit`
  - `fail-on-error: false` pour ne pas bloquer le workflow en cas de tests KO

- **Amélioré** le GitHub Step Summary :
  - Affichage des chemins des rapports de tests
  - Affichage des chemins des rapports de couverture
  - Indication du statut du job

### Commandes exactes exécutées en CI

```bash
mvn clean verify -B
```

### Chemins des rapports générés

| Type | Format | Chemin |
|------|--------|--------|
| Tests | JUnit XML | `back/target/surefire-reports/*.xml` |
| Tests | TXT Summary | `back/target/surefire-reports/*.txt` |
| Coverage | HTML | `back/target/site/jacoco/index.html` |
| Coverage | XML | `back/target/site/jacoco/jacoco.xml` |
| Coverage | CSV | `back/target/site/jacoco/jacoco.csv` |

---

## 4. Front-end : Tests + Coverage

### Modifications apportées

#### A) Installation de dépendances
```bash
npm install --save-dev karma-junit-reporter
```

**Pourquoi ?**
- Karma par défaut ne génère pas de rapport JUnit XML
- `karma-junit-reporter` permet de produire un fichier XML compatible avec GitHub Actions Test Reporting
- Alternative : `karma-mocha-reporter` ou configuration custom, mais `karma-junit-reporter` est le plus simple et maintenu

#### B) Configuration Karma (`front/karma.conf.js`)

**1. Ajout du plugin JUnit :**
```javascript
plugins: [
  require('karma-jasmine'),
  require('karma-chrome-launcher'),
  require('karma-jasmine-html-reporter'),
  require('karma-coverage'),
  require('karma-junit-reporter'),  // ← Nouveau
  require('@angular-devkit/build-angular/plugins/karma')
],
```

**2. Configuration du reporter JUnit :**
```javascript
junitReporter: {
  outputDir: require('path').join(__dirname, './test-results'),
  outputFile: 'junit.xml',
  useBrowserName: false,
  nameFormatter: undefined,
  classNameFormatter: undefined,
  properties: {},
  xmlVersion: null
},
```

**3. Ajout du reporter `lcovonly` pour la couverture :**
```javascript
coverageReporter: {
  dir: require('path').join(__dirname, './coverage/bobapp'),
  subdir: '.',
  reporters: [
    { type: 'html' },
    { type: 'text-summary' },
    { type: 'lcovonly' }  // ← Nouveau (format standard pour Sonar)
  ]
},
```

**4. Activation du reporter JUnit :**
```javascript
reporters: ['progress', 'kjhtml', 'junit'],  // ← 'junit' ajouté
```

#### C) Workflow CI (`.github/workflows/ci.yml`)

- **Ajouté** le flag `--code-coverage` à la commande de test :
  ```bash
  npm run test -- --watch=false --browsers=ChromeHeadless --code-coverage
  ```

- **Ajouté** l'upload des artefacts :
  - Rapports de couverture : `front/coverage/`
  - Rapports de tests : `front/test-results/*.xml`
  - Retention : 30 jours
  - Condition : `if: always()`

- **Ajouté** Test Reporting avec `dorny/test-reporter@v1` :
  - Path : `front/test-results/*.xml`
  - Reporter : `java-junit` (compatible avec JUnit XML)
  - `fail-on-error: false`

- **Amélioré** le GitHub Step Summary :
  - Affichage des chemins des rapports de tests
  - Affichage des chemins des rapports de couverture
  - Indication du statut du job

### Commande exacte exécutée en CI

```bash
npm run test -- --watch=false --browsers=ChromeHeadless --code-coverage
```

**Explications des flags :**
- `--watch=false` : Désactive le mode watch (nécessaire en CI)
- `--browsers=ChromeHeadless` : Utilise Chrome en mode headless (pas d'interface graphique)
- `--code-coverage` : Active la génération des rapports de couverture

### Chemins des rapports générés

| Type | Format | Chemin |
|------|--------|--------|
| Tests | JUnit XML | `front/test-results/junit.xml` |
| Coverage | HTML | `front/coverage/bobapp/index.html` |
| Coverage | LCOV | `front/coverage/bobapp/lcov.info` |

---

## 5. Test Reporting GitHub Actions

### Action utilisée
[dorny/test-reporter@v1](https://github.com/dorny/test-reporter)

**Pourquoi cette action ?**
- Action Marketplace officielle et maintenue (4k+ stars)
- Support natif de JUnit XML (Java et JavaScript)
- Génère des annotations dans la PR
- Affiche les résultats dans l'onglet "Checks"
- Gratuit et open-source

### Configuration Backend
```yaml
- name: Publish backend test report
  if: always()
  uses: dorny/test-reporter@v1
  with:
    name: Backend Test Results
    path: back/target/surefire-reports/*.xml
    reporter: java-junit
    fail-on-error: false
```

### Configuration Frontend
```yaml
- name: Publish frontend test report
  if: always()
  uses: dorny/test-reporter@v1
  with:
    name: Frontend Test Results
    path: front/test-results/*.xml
    reporter: java-junit
    fail-on-error: false
```

### Où les fichiers JUnit sont-ils localisés ?
- **Backend** : `back/target/surefire-reports/*.xml` (générés automatiquement par Maven Surefire)
- **Frontend** : `front/test-results/junit.xml` (généré par karma-junit-reporter)

### Comment ça s'affiche dans le run ?
1. **Onglet "Checks"** : Liste des tests exécutés avec statut (✓ ou ✗)
2. **Annotations** : Les tests en échec apparaissent comme annotations dans les fichiers
3. **GitHub Step Summary** : Résumé textuel avec chemins des rapports
4. **Artifacts** : Rapports téléchargeables pour analyse détaillée

---

## 6. Artefacts uploadés

### Liste des artefacts

| Nom | Contenu | Raison |
|-----|---------|--------|
| `backend-jacoco-reports` | `back/target/site/jacoco/` | Rapports de couverture backend (HTML + XML + CSV) pour analyse et intégration future avec Sonar |
| `backend-test-results` | `back/target/surefire-reports/*.xml` | Rapports de tests backend JUnit pour diagnostic en cas d'échec |
| `frontend-coverage-reports` | `front/coverage/` | Rapports de couverture frontend (HTML + LCOV) pour analyse et intégration future avec Sonar |
| `frontend-test-results` | `front/test-results/*.xml` | Rapports de tests frontend JUnit pour diagnostic en cas d'échec |

### Pourquoi ces artefacts ?

1. **Traçabilité** : Conservation des rapports pour audit et historique
2. **Diagnostic** : En cas d'échec, possibilité de télécharger les rapports pour analyse locale
3. **Préparation Sonar** : Les rapports XML (JaCoCo + LCOV) seront utilisés pour l'intégration SonarQube (étape future)
4. **Documentation** : Les rapports HTML permettent une lecture facile sans outils spécialisés
5. **Persistance** : Même si les tests échouent, les artefacts sont uploadés (`if: always()`)

### Retention des artefacts
- **Durée** : 30 jours
- **Raison** : Compromis entre coût de stockage et besoin d'accès aux rapports récents

---

## 7. Limites connues / Points fragiles

### Backend
- ✅ Pas de limitation majeure
- ⚠️ JaCoCo 0.8.8 : version stable mais pas la toute dernière (0.8.12 disponible). Choix de 0.8.8 pour compatibilité prouvée.

### Frontend
- ⚠️ **Tests existants avec erreurs Angular Material** : Les tests génèrent des erreurs sur les composants Material (`mat-toolbar`, `mat-card`, etc.) mais les tests passent quand même. Ces erreurs étaient présentes avant cette étape et ne sont **pas** de notre responsabilité.
- ⚠️ **Chrome Headless** : Nécessite un environnement Linux avec Chrome installé (OK sur `ubuntu-latest` de GitHub Actions)
- ⚠️ **Coverage à 76%** : Couverture actuelle du code frontend. À améliorer dans les prochaines étapes.

### CI/CD
- ⚠️ **Permissions `checks: write`** : Requises pour que `dorny/test-reporter` puisse publier les résultats. Si le workflow échoue avec une erreur de permission, vérifier les permissions du workflow et du token GITHUB_TOKEN.
- ⚠️ **Path patterns** : Les wildcards (`*.xml`) doivent correspondre à des fichiers existants. Si aucun fichier ne correspond, l'upload d'artefact ne créera pas d'artefact (comportement normal).

### Sécurité
- ✅ Aucune vulnérabilité introduite par les changements de cette étape
- ⚠️ `npm install` a remonté 66 vulnérabilités dans les dépendances frontend (9 low, 15 moderate, 39 high, 3 critical). Ces vulnérabilités sont **pré-existantes** et non introduites par cette étape. Elles devront être traitées dans une étape ultérieure via `npm audit fix`.

---

## 8. Prochaines évolutions (Étape 3+)

### Étape 3 : Intégration SonarQube
- Analyse de code statique (qualité + sécurité)
- Utilisation des rapports de couverture générés (XML)
- Dashboard SonarQube pour suivi de la dette technique

### Étape 4 : Docker Build & Push
- Création d'images Docker pour backend et frontend
- Push sur un registre (Docker Hub, GitHub Container Registry, etc.)
- Tagging sémantique (version, SHA, latest)

### Étape 5 : Déploiement continu
- Déploiement automatique sur environnement de staging
- Tests d'intégration post-déploiement
- Rollback automatique en cas d'échec

### Améliorations continues
- **Backend** : Augmenter la couverture de code (ajouter des tests)
- **Frontend** : Corriger les erreurs Angular Material dans les tests
- **Frontend** : Augmenter la couverture de code (actuellement 76%)
- **CI** : Parallélisation des jobs backend/frontend (déjà le cas mais peut être optimisé)
- **Sécurité** : Traiter les vulnérabilités npm remontées par `npm audit`
- **Performance** : Caching plus agressif des dépendances Maven et npm
- **Monitoring** : Ajouter des métriques de performance des builds

---

## Conclusion

Cette étape a permis de :
- ✅ Générer des rapports de tests et de couverture pour backend et frontend
- ✅ Publier des rapports lisibles dans GitHub Actions
- ✅ Uploader des artefacts pour diagnostic et traçabilité
- ✅ Optimiser le workflow CI (suppression des doubles exécutions)
- ✅ Préparer l'intégration future avec SonarQube

**Points d'attention** : Les tests frontend génèrent des erreurs Angular Material mais passent. Ces erreurs sont pré-existantes et ne bloquent pas la CI.

**Prochaine étape** : Intégration SonarQube pour l'analyse de code statique et de sécurité.
