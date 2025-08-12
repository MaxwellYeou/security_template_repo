# 🚀 Guide de Configuration des Workflows de Sécurité

## 📋 Prérequis

Avant d'intégrer les workflows de sécurité dans votre repository, assurez-vous d'avoir :

### 🔑 Secrets GitHub Requis

Configurez ces secrets dans **Settings > Secrets and variables > Actions** :

| Secret | Description | Requis | Exemple |
|--------|-------------|--------|---------|
| `SONAR_TOKEN` | Token d'authentification SonarQube | ✅ | `squ_abc123...` |
| `SONAR_HOST_URL` | URL de votre instance SonarQube | ✅ | `https://sonarqube.company.com` |
| `NVD_API_KEY` | Clé API NVD pour Dependency Check | ⭕ | `12345678-1234-1234-1234-123456789012` |

### 🏗️ Structure des Branches

Assurez-vous que votre repository utilise cette structure :
```
main (production)
├── dev_test (intégration)
│   ├── feature/[nom-feature]
│   ├── bugfix/[nom-bugfix]
│   └── hotfix/[nom-hotfix]
```

## 🔧 Configuration par Type de Projet

### 📁 1. Projet Angular

Créez le fichier `.github/workflows/security.yml` :

```yaml
name: Angular Security Pipeline

on:
  push:
    branches: ['**']
  pull_request:
    branches: [dev_test, main]
    types: [opened, synchronize, reopened]

jobs:
  security-analysis:
    uses: Tower-IS/global-workflows/.github/workflows/security-checks.yml@main
    secrets:
      SONAR_TOKEN: ${{ secrets.SONAR_TOKEN }}
      SONAR_HOST_URL: ${{ secrets.SONAR_HOST_URL }}
      NVD_API_KEY: ${{ secrets.NVD_API_KEY }}
    with:
      cvss_threshold: "7.0"
      skip_zap_scan: false  # ZAP activé pour Angular
```

**✅ Analyses incluses :**
- 📦 Dependency Check (npm packages)
- 🔍 SonarQube (qualité + sécurité)
- 🛡️ ZAP Security Testing (tests de pénétration)

### 🐍 2. Projet Python

Créez le fichier `.github/workflows/security.yml` :

```yaml
name: Python Security Pipeline

on:
  push:
    branches: ['**']
  pull_request:
    branches: [dev_test, main]
    types: [opened, synchronize, reopened]

jobs:
  security-analysis:
    uses: Tower-IS/global-workflows/.github/workflows/security-checks.yml@main
    secrets:
      SONAR_TOKEN: ${{ secrets.SONAR_TOKEN }}
      SONAR_HOST_URL: ${{ secrets.SONAR_HOST_URL }}
      NVD_API_KEY: ${{ secrets.NVD_API_KEY }}
    with:
      cvss_threshold: "7.0"
      skip_zap_scan: true  # Pas de ZAP pour Python pur
```

**✅ Analyses incluses :**
- 📦 Dependency Check (pip packages)
- 🔍 SonarQube (qualité + sécurité)

### 🌐 3. Projet Full-Stack (Angular + Python/Node)

```yaml
name: Full-Stack Security Pipeline

on:
  push:
    branches: ['**']
  pull_request:
    branches: [dev_test, main]
    types: [opened, synchronize, reopened]

jobs:
  security-analysis:
    uses: Tower-IS/global-workflows/.github/workflows/security-checks.yml@main
    secrets:
      SONAR_TOKEN: ${{ secrets.SONAR_TOKEN }}
      SONAR_HOST_URL: ${{ secrets.SONAR_HOST_URL }}
      NVD_API_KEY: ${{ secrets.NVD_API_KEY }}
    with:
      cvss_threshold: "7.0"
      skip_zap_scan: false  # ZAP activé pour le front-end
      force_full_scan: true  # Scan complet pour projets complexes
```

## ⚙️ Options de Configuration Avancées

### 🎛️ Paramètres Disponibles

| Paramètre | Type | Défaut | Description |
|-----------|------|--------|-------------|
| `force_full_scan` | boolean | `false` | Force un scan complet même sur les branches feature |
| `cvss_threshold` | string | `"7.0"` | Seuil CVSS pour considérer une vulnérabilité comme critique |
| `skip_zap_scan` | boolean | `false` | Désactive les tests ZAP (pour projets sans front-end) |

### 🔧 Configuration Personnalisée

Pour des besoins spécifiques, utilisez cette configuration :

```yaml
name: Custom Security Pipeline

on:
  push:
    branches: ['**']
  pull_request:
    branches: [dev_test, main]
  workflow_dispatch:  # Déclenchement manuel
    inputs:
      scan_level:
        description: 'Niveau de scan'
        required: true
        type: choice
        options: ['basic', 'standard', 'full', 'compliance']

jobs:
  security-analysis:
    uses: Tower-IS/global-workflows/.github/workflows/security-checks.yml@main
    secrets:
      SONAR_TOKEN: ${{ secrets.SONAR_TOKEN }}
      SONAR_HOST_URL: ${{ secrets.SONAR_HOST_URL }}
      NVD_API_KEY: ${{ secrets.NVD_API_KEY }}
    with:
      force_full_scan: ${{ github.event.inputs.scan_level == 'full' || github.event.inputs.scan_level == 'compliance' }}
      cvss_threshold: ${{ github.event.inputs.scan_level == 'compliance' && '5.0' || '7.0' }}
      skip_zap_scan: ${{ github.event.inputs.scan_level == 'basic' }}
```

## 🛡️ Protection des Branches

### 📋 Configuration Recommandée

Pour maximiser la sécurité, configurez ces protections dans **Settings > Branches** :

#### 🚀 Branche `main` (Production)
```yaml
Protection Rules:
  ✅ Require a pull request before merging
  ✅ Require approvals: 2
  ✅ Dismiss stale PR approvals when new commits are pushed
  ✅ Require review from CODEOWNERS
  ✅ Require status checks to pass before merging
      - security-analysis
      - Production Security Gate
  ✅ Require branches to be up to date before merging
  ✅ Require conversation resolution before merging
  ✅ Restrict pushes that create files
```

#### 🧪 Branche `dev_test` (Intégration)
```yaml
Protection Rules:
  ✅ Require a pull request before merging
  ✅ Require approvals: 1
  ✅ Require status checks to pass before merging
      - security-analysis
      - Dev Test Security Gate
  ✅ Require branches to be up to date before merging
```

## 📊 Monitoring et Alertes

### 🔔 Configuration des Notifications

1. **Équipes GitHub** : Créez des équipes pour les notifications
   - `@company/security-team`
   - `@company/dev-team-[project]`
   - `@company/devsecops`

2. **Intégrations Slack/Teams** (optionnel) :
   ```yaml
   # Ajout dans le workflow si besoin
   - name: Notify Slack on Critical Issues
     if: contains(needs.*.result, 'failure')
     uses: 8398a7/action-slack@v3
     with:
       status: failure
       channel: '#security-alerts'
   ```

### 📈 Dashboard de Sécurité

Le dashboard automatique sera disponible chaque lundi dans les Issues avec le label `security-dashboard`.

## 🚨 Gestion des Incidents

### 🔴 Vulnérabilités Critiques

Lorsque des vulnérabilités critiques sont détectées :

1. **Blocage automatique** des PR vers `main`
2. **Issue créée automatiquement** avec priorité `critical`
3. **Notification** des équipes de sécurité
4. **Checklist d'action** fournie dans l'issue

### 📞 Escalade

- **Niveau 1** : Développeur (feature branches)
- **Niveau 2** : Tech Lead (dev_test)
- **Niveau 3** : Security Team (main branch)
- **Niveau 4** : CISO (incidents critiques)

## 🔧 Dépannage

### ❌ Problèmes Courants

#### 1. "Secret not found"
```bash
# Vérifiez que les secrets sont configurés :
# Settings > Secrets and variables > Actions
SONAR_TOKEN=your_token
SONAR_HOST_URL=https://your-sonarqube.com
```

#### 2. "Quality Gate failed"
```bash
# Vérifiez la configuration SonarQube :
# - Quality Gate existe
# - Projet configuré correctement
# - Métriques définies
```

#### 3. "ZAP scan failed"
```bash
# Vérifiez que l'application Angular build correctement :
# - package.json présent
# - npm install réussit
# - npm run build réussit
```

### 🔍 Debugging

Pour débugger les workflows :

1. **Activez les logs détaillés** :
   ```yaml
   env:
     ACTIONS_RUNNER_DEBUG: true
     ACTIONS_STEP_DEBUG: true
   ```

2. **Consultez les artefacts** téléchargés après chaque run
3. **Vérifiez les logs** de chaque étape dans l'interface GitHub

## 📞 Support

- **Documentation complète** : Repository `Tower-IS/global-workflows`
- **Issues** : Utilisez les labels `security`, `ci-cd`, `bug`
- **Équipe DevSecOps** : `@Tower-IS/devsecops-team`
- **Urgences sécurité** : Suivre le processus d'incident response

---

## ✅ Checklist de Mise en Place

Avant d'activer les workflows de sécurité :

- [ ] Secrets GitHub configurés
- [ ] Structure des branches respectée
- [ ] Protections de branches activées
- [ ] SonarQube projet créé et configuré
- [ ] Documentation projet mise à jour

---

*Guide de configuration - Workflows de sécurité Tower-IS*
*Version 1.0 - 2025*
