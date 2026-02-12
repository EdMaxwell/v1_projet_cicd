# Frontend BobApp - Vue d'ensemble

## But du composant

Le frontend de BobApp est une application Angular qui sert d'interface utilisateur pour afficher des blagues (jokes). C'est une Single Page Application (SPA) qui communique avec le backend Spring Boot pour récupérer et afficher des blagues aléatoires.

**Fonctionnalités principales :**
- Interface utilisateur Material Design (Angular Material)
- Récupération de blagues depuis l'API backend
- Affichage dynamique sans rechargement de page
- Interface responsive et moderne

## Stack & versions

### Framework et langage
- **Angular :** 14.2.0
- **TypeScript :** 4.8.2
- **Node.js :** 16+ recommandé (compatible avec Angular 14)
- **npm :** 7+ (livré avec Node 16)

### Bibliothèques principales

**UI Framework :**
- **@angular/material :** 14.2.0 (Material Design components)
- **@angular/cdk :** 14.2.0 (Component Dev Kit)

**Core Angular :**
- **@angular/core :** 14.2.0
- **@angular/common :** 14.2.0
- **@angular/router :** 14.2.0
- **@angular/forms :** 14.2.0

**Dépendances runtime :**
- **rxjs :** 7.5.6 (Reactive Extensions for JS)
- **zone.js :** 0.11.8 (Change detection)
- **tslib :** 2.4.0 (TypeScript runtime library)

### Outils de développement

**Build & CLI :**
- **@angular/cli :** 14.2.1
- **@angular-devkit/build-angular :** 14.2.1

**Tests :**
- **Karma :** 6.4.0 (Test runner)
- **Jasmine :** 4.4.0 (Test framework)
- **karma-jasmine :** 5.1.0
- **karma-chrome-launcher :** 3.1.1 (Pour tests headless)
- **karma-coverage :** 2.2.0 (Code coverage)

**Configuration :**
- **angular.json :** Configuration Angular CLI
- **tsconfig.json :** Configuration TypeScript
- **karma.conf.js :** Configuration tests unitaires

## Comment le lancer en local

### Prérequis
1. **Installer Node.js 16+**
   ```bash
   node -v  # Vérifier version >= 16.x
   npm -v   # Vérifier version >= 7.x
   ```

2. **Cloner le repository et naviguer vers le frontend**
   ```bash
   git clone <repo-url>
   cd v1_projet_cicd/front
   ```

### Installation des dépendances

```bash
# Installation standard (développement)
npm install

# Ou installation stricte (CI/reproduction exacte)
npm ci
```

**Différence :**
- `npm install` : Met à jour `package-lock.json` si nécessaire
- `npm ci` : Installation exacte basée sur le lock file (recommandé pour CI)

### Lancer le serveur de développement

```bash
npm run start
# ou
npm start
```

**Résultat :**
- Serveur de dev démarre sur `http://localhost:4200`
- Hot-reload activé (recharge automatique lors des modifications)
- Compilation incrémentale pour feedback rapide

**Options avancées :**
```bash
# Changer le port
ng serve --port 4300

# Ouvrir automatiquement le navigateur
ng serve --open

# Mode production local
ng serve --configuration=production
```

### Accéder à l'application

Ouvrir un navigateur : **http://localhost:4200**

**Note :** Si le backend n'est pas lancé, les appels API échoueront. Voir `docs/back-overview.md` pour lancer le backend.

## Comment le builder / tester

### Build de développement

```bash
# Build rapide sans optimisations
npm run build

# Équivalent
ng build --configuration=development
```

**Sortie :** `dist/bobapp/` (non optimisé, avec source maps)

### Build de production

```bash
# Build optimisé pour production
npm run build -- --configuration=production

# Équivalent
ng build --configuration=production
```

**Optimisations appliquées :**
- Minification du code
- Tree-shaking (suppression du code mort)
- Bundling et optimisation des assets
- AOT compilation (Ahead-Of-Time)
- Pas de source maps

**Sortie :** `dist/bobapp/` (optimisé, prêt pour déploiement)

**Budgets configurés :**
- Initial bundle : max 1 MB (erreur si dépassé)
- Styles par composant : max 4 KB

### Exécuter les tests unitaires

```bash
# Mode watch (développement)
npm run test
# Lance Karma en mode watch, reruns automatiques

# Mode CI (headless, single run)
npm run test -- --watch=false --browsers=ChromeHeadless
```

**Framework de test :**
- **Karma :** Test runner
- **Jasmine :** Syntaxe des tests (describe, it, expect)

**Configuration :** `karma.conf.js`
- Navigateur par défaut : Chrome
- Coverage activé : `karma-coverage`
- Rapports : dans `coverage/` (après exécution)

**Tests existants :**
- `app.component.spec.ts` : Test du composant principal
- `services/jokes.service.spec.ts` : Test du service de blagues

### Vérifier le build

```bash
# Vérifier que le build fonctionne
npm run build -- --configuration=production

# Vérifier la taille du bundle
ls -lh dist/bobapp/

# Servir le build localement (nécessite http-server)
npx http-server dist/bobapp -p 8080
```

### Linter et formatage (si configuré)

```bash
# Vérifier le code TypeScript
ng lint  # Si ESLint configuré

# Formater le code (si Prettier installé)
npx prettier --write "src/**/*.{ts,html,css,scss}"
```

**Note :** Pas de linter apparent dans `package.json` actuel → à ajouter pour qualité de code.

## Spécificités & risques

### Dépendances sensibles

**Angular Material :**
- Version 14.2.0 liée à Angular 14.2.0
- ⚠️ **Risque :** Mise à jour Angular nécessite mise à jour Material en parallèle
- **Impact :** Changements potentiels d'API, breaking changes

**RxJS :**
- Version 7.5.6 (observable patterns)
- ⚠️ **Risque :** Code asynchrone complexe, difficile à débugger si mal maîtrisé
- **Mitigation :** Utiliser des opérateurs RxJS standards (map, switchMap, etc.)

**Zone.js :**
- Patch le comportement asynchrone du navigateur pour Angular
- ⚠️ **Risque :** Peut causer des problèmes avec certaines libs externes
- **Mitigation :** Tester les intégrations tierces

### Scripts particuliers

**package.json scripts :**
```json
{
  "start": "ng serve",
  "build": "ng build",
  "test": "ng test",
  "watch": "ng build --watch --configuration development"
}
```

**Script `watch` :**
- Build incrémental continu
- Utile pour développement avec backend séparé
- Surveille les changements et recompile automatiquement

**Pas de script :**
- Pas de `lint` → linter non configuré
- Pas de `e2e` → tests end-to-end absents (normal pour cette phase)

### Points d'attention CI

#### 1. Tests nécessitent Chrome

**Problème :**
- Karma utilise Chrome/Chromium pour exécuter les tests
- En CI, nécessite `ChromeHeadless`

**Solution appliquée :**
- GitHub Actions `ubuntu-latest` inclut Chrome
- Commande CI : `npm run test -- --watch=false --browsers=ChromeHeadless`

**Risque résiduel :**
- Versions de Chrome peuvent varier entre local et CI
- Tests peuvent passer en local mais échouer en CI (ou inverse)

**Mitigation :**
- Utiliser `--headless=new` si besoin (Chrome moderne)
- Considérer Puppeteer pour contrôle version Chrome

#### 2. Configuration proxy (proxy.config.json)

**Fichier présent :** `src/proxy.config.json`

**But :** Rediriger les appels API vers le backend en développement
```json
// Exemple probable :
{
  "/api": {
    "target": "http://localhost:8080",
    "secure": false
  }
}
```

**Point d'attention :**
- Fonctionne uniquement avec `ng serve`
- En production, le frontend doit connaître l'URL réelle du backend
- **CI Impact :** Pas d'impact en CI car on ne lance que build/tests (pas de serveur)

#### 3. Environnements Angular

**Fichiers :**
- `src/environments/environment.ts` (dev)
- `src/environments/environment.prod.ts` (prod)

**Utilisation :**
- Contiennent typiquement l'URL de l'API backend
- Swappés automatiquement selon la configuration de build

**Risque :**
- Oublier de mettre à jour l'URL de prod → appels API échouent en production
- **Mitigation :** Valider les environnements lors du déploiement

#### 4. Budget de taille

**Configuré dans `angular.json` :**
- Initial bundle : warning à 500 KB, erreur à 1 MB
- Component styles : warning à 2 KB, erreur à 4 KB

**Impact CI :**
- Build échoue si budget dépassé (comportement souhaité)
- Oblige à optimiser ou justifier l'augmentation du budget

**Mitigation :**
- Lazy loading des modules Angular
- Code splitting
- Optimisation des assets (images, fonts)

#### 5. Pas de code coverage publié

**État actuel :**
- Tests exécutés en CI
- Coverage généré par Karma → `coverage/` directory
- ⚠️ **Manque :** Coverage non publié ni visualisé

**Prochaine étape (Étape 2) :**
- Publier coverage avec actions/upload-artifact
- Intégrer avec SonarCloud
- Ajouter badges de coverage dans README

#### 6. Pas de tests d'intégration (E2E)

**Constat :**
- Pas de Cypress, Playwright, ou Protractor configuré
- Seulement des tests unitaires Karma/Jasmine

**Impact :**
- Fonctionnalités critiques non testées end-to-end
- Risque de régressions UI non détectées

**Recommandation :**
- Ajouter Cypress pour tests E2E (Étape future)
- Tester les parcours utilisateurs critiques

### Risques de sécurité

**Dépendances :**
- ⚠️ Pas de scan de vulnérabilités automatique
- **Mitigation :** Ajouter `npm audit` dans CI (Étape 2)
- **Mitigation :** Activer Dependabot sur GitHub

**Versions :**
- Angular 14.2.0 (Août 2022) → pas la dernière
- ⚠️ Peut contenir des vulnérabilités connues
- **Recommandation :** Planifier mise à jour vers Angular 15+ (après stabilisation CI/CD)

**Secrets :**
- Pas de secrets apparents dans le code frontend (bon point)
- Vérifier que les API keys ne sont jamais committées

### Performance

**Points positifs :**
- AOT compilation activé en production
- Lazy loading possible via Angular Router
- Material Design optimisé

**Points d'amélioration :**
- Vérifier la taille des bundles (budget actuel : 1 MB max)
- Implémenter lazy loading si modules grossissent
- Optimiser les images dans `src/assets/`

### Compatibilité navigateurs

**Configuration :** `.browserslistrc`
- Chrome (dernière version)
- Firefox (dernière version)
- Edge (2 dernières versions majeures)
- Safari (2 dernières versions majeures)
- iOS Safari (2 dernières versions)
- Firefox ESR

**Impact :**
- Polyfills inclus via `polyfills.ts`
- Build adapte le code selon les cibles

**Risque :**
- Support IE11 abandonné (bon débarras)
- Tester sur Safari/iOS si utilisateurs Apple

---

## Checklist de vérification locale

Avant de committer du code frontend :

```bash
cd front

# 1. Installer les dépendances
npm ci

# 2. Vérifier que le build passe
npm run build -- --configuration=production

# 3. Vérifier que les tests passent
npm run test -- --watch=false --browsers=ChromeHeadless

# 4. Lancer l'app localement
npm run start
# Vérifier visuellement que tout fonctionne

# 5. (Optionnel) Vérifier les vulnérabilités
npm audit

# 6. (Optionnel) Formater le code
npx prettier --check "src/**/*.{ts,html,scss}"
```

✅ Si toutes ces étapes passent, le code est prêt pour commit/PR.
