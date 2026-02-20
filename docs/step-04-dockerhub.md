# Étape 4 – Publication des images Docker sur Docker Hub

## 1. Objectif de l'étape

Automatiser la construction et la publication des images Docker du back-end et du front-end sur Docker Hub, **uniquement** lorsqu'un push est effectué sur la branche `main` et que tous les jobs de qualité (tests, couverture, SonarCloud) ont réussi.

---

## 2. Stratégie de publication

| Événement | Comportement |
|-----------|-------------|
| `push` sur `main` | Tests + Sonar + build Docker + push Docker Hub |
| `pull_request` vers `main` ou `develop` | Tests + Sonar uniquement – **aucun push Docker Hub** |

**Pourquoi cette distinction ?**  
Une pull request représente du code en cours de revue, pas encore validé. Publier une image à ce stade polluerait Docker Hub avec des artefacts potentiellement instables et exposerait des secrets inutilement. Le push sur `main` signifie que le code a été relu et mergé : c'est le bon moment pour livrer.

---

## 3. Secrets requis

| Nom du secret GitHub | Description |
|----------------------|-------------|
| `DOCKERHUB_USERNAME` | Votre nom d'utilisateur Docker Hub |
| `DOCKERHUB_TOKEN`    | Token d'accès personnel Docker Hub (pas le mot de passe) |

### Créer un token Docker Hub

1. Connectez-vous sur [hub.docker.com](https://hub.docker.com).
2. Allez dans **Account Settings → Security → Access Tokens**.
3. Cliquez **New Access Token**, donnez-lui un nom (ex : `github-actions`) et les permissions **Read & Write**.
4. Copiez le token généré.
5. Dans votre dépôt GitHub : **Settings → Secrets and variables → Actions → New repository secret**.
   - `DOCKERHUB_USERNAME` → votre login Docker Hub
   - `DOCKERHUB_TOKEN` → le token copié

---

## 4. Tagging strategy

| Tag | Condition | Utilité |
|-----|-----------|---------|
| `latest` | push sur `main` | Pointe toujours vers la dernière version stable |
| `sha-<short>` (7 chars) | push sur `main` | Traçabilité exacte du commit source |

**Pourquoi ces deux tags ?**  
- `latest` permet aux utilisateurs de toujours tirer la version courante sans connaître un tag précis.  
- `sha-<short>` garantit la reproductibilité et facilite les rollbacks : on sait exactement quel commit a produit quelle image.  
- Un tag de version sémantique (`v1.2.3`) serait pertinent si le projet utilise un système de release GitHub – à ajouter lors de l'implémentation d'un workflow de release dédié.

---

## 5. Build & Push : détails techniques

### Actions utilisées

| Action | Rôle |
|--------|------|
| `docker/setup-buildx-action@v3` | Active Docker Buildx (builder étendu) |
| `docker/login-action@v3` | Authentification Docker Hub via secrets |
| `docker/build-push-action@v6` | Build multi-étapes + push |

### Dockerfiles et contextes

| Image | Dockerfile | Contexte |
|-------|-----------|---------|
| `bobapp-back` | `./back/Dockerfile` | `./back` |
| `bobapp-front` | `./front/Dockerfile` | `./front` |

### Notes techniques

- **Buildx** est activé pour permettre des builds multi-plateformes à l'avenir (ex : `linux/amd64,linux/arm64`). Actuellement, une seule plateforme est ciblée.
- **QEMU** n'est pas configuré car nous ne faisons pas de cross-compilation pour l'instant.
- **Cache Docker** : non configuré explicitement dans cette version. Pour optimiser les temps de build, on pourrait ajouter `cache-from: type=gha` et `cache-to: type=gha,mode=max` (GitHub Actions cache).

---

## 6. Gates / Sécurité

### Dépendances (`needs`)

```yaml
needs: [backend, frontend, sonar]
```

Le job `docker` attend la réussite de **tous** les jobs amont. Si l'un échoue, GitHub Actions ne lance pas le job `docker`.

### Condition (`if`)

```yaml
if: github.event_name == 'push' && github.ref == 'refs/heads/main'
```

Double vérification :
- `github.event_name == 'push'` : exclut les `pull_request`, `schedule`, `workflow_dispatch`, etc.
- `github.ref == 'refs/heads/main'` : exclut tout push sur d'autres branches.

### Prévention des fuites de secrets

- Les secrets `DOCKERHUB_USERNAME` et `DOCKERHUB_TOKEN` ne sont jamais affichés en clair dans les logs (GitHub les masque automatiquement).
- Le token Docker Hub (accès **Read & Write**) n'est pas le mot de passe du compte ; sa portée est limitée.
- Le job `docker` ne tourne pas sur les PRs : aucun risque d'exfiltration par un fork malveillant.

---

## 7. Comment tester localement

### Build du back-end

```bash
cd back
docker build -t bobapp-back:local .
```

### Build du front-end

```bash
cd front
docker build -t bobapp-front:local .
```

### Run des conteneurs

```bash
# Back-end (port 8080)
docker run -p 8080:8080 bobapp-back:local

# Front-end (port 80)
docker run -p 4200:80 bobapp-front:local
```

Vérification : ouvrez `http://localhost:8080` pour le back et `http://localhost:4200` pour le front.

---

## 8. Limites connues / améliorations possibles

| Limite | Amélioration envisageable |
|--------|--------------------------|
| Pas de builds multi-plateformes | Ajouter `platforms: linux/amd64,linux/arm64` et QEMU |
| Pas de cache Docker explicite | Activer `cache-from/cache-to: type=gha` pour accélérer les rebuilds |
| Pas de tag de version sémantique | Créer un workflow `release.yml` déclenché sur les tags `v*.*.*` |
| `latest` sur `main` seulement | Acceptable pour un projet mono-branche ; adapter si `develop` devient une branche de pré-production |
| Pas de scan de vulnérabilités de l'image | Intégrer `aquasecurity/trivy-action` ou `anchore/scan-action` avant le push |
| Pas de notification en cas d'échec | Ajouter une étape de notification Slack/email sur `failure()` |
