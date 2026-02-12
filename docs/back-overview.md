# Backend BobApp - Vue d'ensemble

## But du composant

Le backend de BobApp est une API REST Spring Boot qui fournit des blagues (jokes) aléatoires. C'est le cœur métier de l'application, exposant des endpoints HTTP pour que le frontend Angular puisse récupérer et afficher des blagues.

**Fonctionnalités principales :**
- API REST pour récupérer des blagues
- Lecture de blagues depuis des données JSON statiques
- Service métier pour la gestion des blagues
- Architecture Spring Boot standard (Controller → Service → Data)

## Stack & versions

### Framework et langage
- **Spring Boot :** 2.6.1
- **Java :** 11 (LTS - Long Term Support)
- **Maven :** 3.6+ (via Maven Wrapper `mvnw`)

### Bibliothèques principales

**Spring Boot Starters :**
- **spring-boot-starter-webflux :** API réactive avec WebFlux
  - Framework web réactif de Spring
  - Alternative à spring-boot-starter-web (Servlet classique)
  - Utilise Reactor et Netty sous le capot

**Tests :**
- **spring-boot-starter-test :** Suite complète de tests Spring Boot
  - JUnit 5 (Jupiter)
  - Mockito
  - AssertJ
  - Spring Test & Spring Boot Test
  - Hamcrest

**Outils de qualité :**
- **JaCoCo :** 0.8.5 (Code coverage)
  - Plugin Maven configuré pour générer des rapports de couverture
  - Exécution automatique lors de la phase `test`

### Architecture du code

```
back/src/
├── main/java/com/openclassrooms/bobapp/
│   ├── BobappApplication.java          # Point d'entrée Spring Boot
│   ├── controller/
│   │   └── JokeController.java         # REST Controller (endpoints API)
│   ├── service/
│   │   └── JokeService.java            # Logique métier
│   ├── model/
│   │   └── Joke.java                   # Modèle de données
│   └── data/
│       └── JsonReader.java             # Lecture données JSON
└── test/java/com/openclassrooms/bobapp/
    └── BobappApplicationTests.java     # Tests Spring Boot
```

**Pattern MVC adapté :**
- **Controller :** Expose les endpoints REST
- **Service :** Contient la logique métier
- **Model :** Classes de données (POJO)
- **Data :** Accès aux données (fichiers JSON)

## Comment le lancer en local

### Prérequis
1. **Installer Java 11**
   ```bash
   java -version  # Vérifier version = 11.x
   ```
   
   **Installation rapide :**
   - Linux : `sudo apt install openjdk-11-jdk`
   - macOS : `brew install openjdk@11`
   - Windows : Télécharger depuis [Adoptium](https://adoptium.net/)

2. **Maven pas nécessaire** (Maven Wrapper inclus)

### Cloner et naviguer

```bash
git clone <repo-url>
cd v1_projet_cicd/back
```

### Lancer l'application

**Option 1 : Avec Maven Wrapper (recommandé)**
```bash
./mvnw spring-boot:run
```

**Option 2 : Avec Maven global (si installé)**
```bash
mvn spring-boot:run
```

**Option 3 : Build puis exécuter le JAR**
```bash
# Build
./mvnw clean package

# Exécuter
java -jar target/bobapp-0.0.1-SNAPSHOT.jar
```

### Vérifier que le backend fonctionne

**Serveur démarre sur :** `http://localhost:8080`

**Tester l'API :**
```bash
# Appel simple
curl http://localhost:8080/api/jokes

# Ou avec un navigateur
open http://localhost:8080/api/jokes
```

**Résultat attendu :**
- JSON avec des blagues
- Status HTTP 200 OK

### Configuration

**Port par défaut :** 8080

**Changer le port (optionnel) :**
```bash
# Via variable d'environnement
SERVER_PORT=9090 ./mvnw spring-boot:run

# Ou créer application.properties
echo "server.port=9090" > src/main/resources/application.properties
```

**Profils Spring :**
- Pas de profils configurés actuellement (dev/prod)
- **Recommandation :** Ajouter `application-dev.properties` et `application-prod.properties`

## Comment le builder / tester

### Installation des dépendances

```bash
# Maven Wrapper télécharge automatiquement les dépendances
./mvnw dependency:resolve
```

**Note :** Cette étape est implicite dans toutes les commandes Maven suivantes.

### Build complet (sans tests)

```bash
./mvnw clean package -DskipTests
```

**Sortie :** `target/bobapp-0.0.1-SNAPSHOT.jar`

### Build complet avec tests

```bash
./mvnw clean install
```

**Ce que ça fait :**
1. **clean :** Supprime le répertoire `target/`
2. **compile :** Compile le code source
3. **test :** Exécute les tests unitaires
4. **package :** Crée le JAR
5. **install :** Installe le JAR dans le repo Maven local (`~/.m2/repository`)

**Sortie :**
- `target/bobapp-0.0.1-SNAPSHOT.jar` (JAR exécutable)
- `target/surefire-reports/` (Rapports de tests)
- `target/site/jacoco/` (Rapports de couverture JaCoCo)

### Exécuter uniquement les tests

```bash
./mvnw test
```

**Framework de test :**
- **JUnit 5** (Jupiter)
- **Spring Boot Test** (@SpringBootTest)

**Tests existants :**
- `BobappApplicationTests.java` : Test de chargement du contexte Spring

**Vérifier les résultats :**
```bash
# Résumé dans la console
# Rapports détaillés XML
ls target/surefire-reports/

# Rapport de couverture JaCoCo (HTML)
open target/site/jacoco/index.html
```

### Générer le rapport de couverture

```bash
# Tests + rapport JaCoCo
./mvnw clean test

# Rapport généré dans
ls target/site/jacoco/
# ├── index.html       # Page principale
# ├── jacoco.xml       # Format XML (pour Sonar)
# └── jacoco.csv       # Format CSV
```

**Configuration JaCoCo :**
- Définie dans `pom.xml`
- Exécution automatique avec `mvn test`
- Génère rapports HTML et XML

### Vérifier le build

```bash
# Build complet
./mvnw clean install

# Vérifier le JAR créé
ls -lh target/*.jar

# Tester l'exécution
java -jar target/bobapp-0.0.1-SNAPSHOT.jar
# Devrait démarrer le serveur sur port 8080
```

### Maven phases principales

| Phase | Commande | Description |
|-------|----------|-------------|
| **clean** | `mvn clean` | Supprime le répertoire `target/` |
| **compile** | `mvn compile` | Compile le code source |
| **test** | `mvn test` | Exécute les tests unitaires |
| **package** | `mvn package` | Crée le JAR (inclut compile + test) |
| **install** | `mvn install` | Installe dans le repo Maven local |
| **verify** | `mvn verify` | Exécute les tests d'intégration |

### Commandes utiles pour le dev

```bash
# Voir les dépendances
./mvnw dependency:tree

# Mettre à jour les dépendances
./mvnw versions:display-dependency-updates

# Vérifier les plugins
./mvnw versions:display-plugin-updates

# Nettoyer les caches Maven
rm -rf ~/.m2/repository
```

## Spécificités & risques

### Dépendances sensibles

#### spring-boot-starter-webflux

**Particularité :**
- Framework **réactif** (Reactor + Netty)
- Différent du Spring MVC classique (Servlet)

**Pourquoi WebFlux ici :**
- API simple, pas besoin de hautes performances réactives
- ⚠️ **Risque :** Overkill pour une app de blagues simple
- **Alternative :** `spring-boot-starter-web` suffirait (Tomcat + Servlet)

**Impact :**
- Courbe d'apprentissage plus raide (Mono, Flux)
- Dépendances plus légères (pas de Tomcat)
- Serveur Netty au lieu de Tomcat

**Recommandation :**
- Acceptable pour l'apprentissage
- Migrer vers `spring-boot-starter-web` si l'équipe n'est pas familière avec WebFlux

#### JaCoCo 0.8.5

**Version :** 0.8.5 (Mars 2020)
- ⚠️ Pas la dernière (0.8.11+ disponible en 2024)
- Compatible avec Java 11 (OK)

**Risque :**
- Bugs corrigés dans les versions récentes
- **Recommandation :** Mettre à jour vers 0.8.11+ lors de la phase 2 (Sonar)

### Scripts particuliers

**Maven Wrapper :**
- `mvnw` (Linux/macOS) et `mvnw.cmd` (Windows)
- Télécharge automatiquement Maven 3.8.3 (défini dans `.mvn/wrapper/maven-wrapper.properties`)

**Avantages :**
- Pas besoin d'installer Maven globalement
- Version Maven fixe pour toute l'équipe
- Reproductibilité parfaite

**Utilisation :**
```bash
# Au lieu de "mvn ..."
./mvnw clean install
```

### Points d'attention CI

#### 1. Version Java stricte

**Problème :**
- `pom.xml` spécifie Java 11
- Code ne compile pas avec Java 8 ou 17+ si incompatibilités

**Solution appliquée en CI :**
```yaml
- uses: actions/setup-java@v4
  with:
    java-version: '11'
    distribution: 'temurin'
```

**Risque résiduel :**
- Développeurs locaux peuvent avoir Java 8/17/21
- Peut causer des surprises si code utilise des features Java 11+

**Mitigation :**
- Documenter la version Java requise (fait dans ce doc)
- Ajouter un `.sdkmanrc` ou `.java-version` pour les outils comme SDKMAN

#### 2. Maven Wrapper et permissions

**Problème potentiel :**
- `mvnw` doit être exécutable (`chmod +x mvnw`)
- Git peut perdre les permissions sur Windows

**Solution préventive :**
- CI utilise `mvnw` directement (pas de `chmod` nécessaire sous Linux)
- GitHub Actions gère les permissions automatiquement

**Vérifier localement :**
```bash
ls -l mvnw
# Si pas exécutable :
chmod +x mvnw
```

#### 3. Tests et JaCoCo

**Configuration actuelle :**
- JaCoCo configuré avec 2 exécutions :
  1. `prepare-agent` : Prépare l'agent JaCoCo avant les tests
  2. `report` : Génère le rapport après les tests

**CI Impact :**
- `mvn clean install` exécute automatiquement JaCoCo
- Rapport généré dans `target/site/jacoco/`

**⚠️ Manque actuel :**
- Rapports non publiés en artifacts CI
- Pas de visualisation de la couverture
- Pas de seuil minimum de couverture

**Prochaine étape (Étape 2) :**
- Publier rapport JaCoCo avec `actions/upload-artifact`
- Intégrer avec SonarCloud
- Définir seuils minimums (ex: 80%)

#### 4. Double déclaration plugin (Dette technique)

**Problème identifié :**
```xml
<plugin>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-maven-plugin</artifactId>
</plugin>
<!-- Ligne 38 : DOUBLON -->
<plugin>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-maven-plugin</artifactId>
</plugin>
```

**Impact :**
- Pas de bug fonctionnel (Maven ignore le doublon)
- Mais mauvaise pratique, confusion pour les développeurs

**Recommandation :**
- Supprimer une des deux déclarations
- Nettoyer le `pom.xml`

#### 5. Pas de tests d'intégration

**Constat :**
- Seulement `BobappApplicationTests.java` (test de contexte)
- Pas de tests de controllers (`@WebFluxTest` ou `@SpringBootTest`)
- Pas de tests de services (`@ExtendWith(MockitoExtension.class)`)

**Impact :**
- Couverture de code faible
- Logique métier non testée
- Risque de régressions

**Recommandation :**
- Ajouter tests unitaires pour `JokeService`
- Ajouter tests d'intégration pour `JokeController`
- Exemple :
  ```java
  @SpringBootTest(webEnvironment = SpringBootTest.WebEnvironment.RANDOM_PORT)
  class JokeControllerIntegrationTest {
      @Test
      void testGetRandomJoke() {
          // ...
      }
  }
  ```

#### 6. Pas de gestion des erreurs visible

**Constat :**
- Pas de `@ControllerAdvice` pour gestion globale des exceptions
- Pas de validation des entrées (`@Valid`, Bean Validation)

**Impact :**
- Erreurs 500 brutes renvoyées au client
- Pas de messages d'erreur structurés

**Recommandation :**
- Ajouter un `GlobalExceptionHandler` avec `@ControllerAdvice`
- Retourner des erreurs JSON structurées
- Exemple :
  ```java
  @ControllerAdvice
  public class GlobalExceptionHandler {
      @ExceptionHandler(JokeNotFoundException.class)
      public ResponseEntity<ErrorResponse> handleNotFound(JokeNotFoundException ex) {
          return ResponseEntity.status(404).body(new ErrorResponse(ex.getMessage()));
      }
  }
  ```

### Risques de sécurité

**Dépendances :**
- ⚠️ Spring Boot 2.6.1 (Décembre 2021) → pas la dernière
- Peut contenir des CVE (Common Vulnerabilities and Exposures)

**Vérifier les vulnérabilités :**
```bash
./mvnw org.owasp:dependency-check-maven:check
```

**Recommandations :**
- Activer Dependabot sur GitHub
- Ajouter OWASP Dependency Check en CI
- Planifier mise à jour vers Spring Boot 2.7.x ou 3.x

**Endpoints exposés :**
- ⚠️ Pas d'authentification visible
- API publique → acceptable pour une app de blagues
- **Attention :** Si données sensibles ajoutées plus tard, implémenter Spring Security

**CORS (Cross-Origin Resource Sharing) :**
- Pas de configuration visible
- Peut causer des erreurs si frontend sur un domaine différent
- **Vérifier :** Ajouter `@CrossOrigin` ou configuration globale si besoin

### Performance

**WebFlux :**
- Serveur Netty → hautes performances en asynchrone
- Surcapacité pour une simple API de blagues
- Mais pas de problème de performance

**Données en mémoire :**
- Lecture de JSON statique (probablement dans `JsonReader`)
- Pas de base de données
- **Avantage :** Très rapide, pas de I/O réseau
- **Limitation :** Données non persistantes

**Optimisations possibles :**
- Cache des blagues en mémoire (si lecture fichier à chaque requête)
- Compression des réponses HTTP (Gzip)

### Compatibilité

**Java 11 :**
- LTS (Long Term Support) jusqu'en 2026
- Largement supporté
- Compatible avec la plupart des bibliothèques

**Spring Boot 2.6.1 :**
- Compatible Java 8-17
- Pas de problème de compatibilité attendu

**Migration future :**
- Spring Boot 3.x nécessite Java 17+
- À considérer dans 1-2 ans

---

## Checklist de vérification locale

Avant de committer du code backend :

```bash
cd back

# 1. Vérifier la version Java
java -version
# Doit être Java 11

# 2. Nettoyer et builder avec tests
./mvnw clean install

# 3. Vérifier que les tests passent
./mvnw test

# 4. Vérifier la couverture (optionnel)
open target/site/jacoco/index.html

# 5. Lancer l'application localement
./mvnw spring-boot:run
# Tester : curl http://localhost:8080/api/jokes

# 6. (Optionnel) Vérifier les vulnérabilités
./mvnw org.owasp:dependency-check-maven:check

# 7. (Optionnel) Vérifier le formatage
# Ajouter un plugin de formatage (google-java-format, spotless)
```

✅ Si toutes ces étapes passent, le code est prêt pour commit/PR.

---

## Commandes Maven de référence

| Objectif | Commande | Description |
|----------|----------|-------------|
| **Build rapide** | `./mvnw clean package -DskipTests` | Build sans tests |
| **Build complet** | `./mvnw clean install` | Build avec tests et installation |
| **Tests seuls** | `./mvnw test` | Exécute les tests |
| **Lancer l'app** | `./mvnw spring-boot:run` | Démarre le serveur |
| **Nettoyer** | `./mvnw clean` | Supprime `target/` |
| **Dépendances** | `./mvnw dependency:tree` | Affiche l'arbre des dépendances |
| **Mise à jour** | `./mvnw versions:display-dependency-updates` | Vérifie les mises à jour |
| **Coverage** | `./mvnw clean test` | Tests + rapport JaCoCo |

**Note :** Remplacer `./mvnw` par `mvn` si Maven est installé globalement.
