---
name: drupal-composer — private repositories
description: Configurer des repositories Composer privés pour Drupal - GitLab/GitHub privés, Satis, authentification COMPOSER_AUTH, et modules custom en packages.
---

# Repositories Privés Composer — Référence Complète

## Repository Git Privé (GitLab / GitHub)

```json
// composer.json — ajouter le repository VCS
{
  "repositories": [
    {
      "type": "vcs",
      "url": "https://gitlab.example.com/mon-org/mon-module-drupal.git"
    }
  ],
  "require": {
    "mon-org/mon-module-drupal": "^1.0"
  }
}
```

**Structure du module custom pour qu'il soit un package Composer :**
```json
// web/modules/custom/mon_module/composer.json
{
  "name": "mon-org/mon-module-drupal",
  "type": "drupal-module",
  "description": "Module Drupal custom",
  "version": "1.0.0",
  "require": {
    "drupal/core": "^10 || ^11"
  }
}
```

---

## Authentification — `auth.json`

```json
// ~/.composer/auth.json (global) — OU
// auth.json (projet, dans .gitignore)
{
  "github-oauth": {
    "github.com": "TOKEN_GITHUB_PERSONAL_ACCESS"
  },
  "gitlab-token": {
    "gitlab.example.com": "TOKEN_GITLAB_PERSONAL_ACCESS"
  },
  "http-basic": {
    "packages.example.com": {
      "username": "mon-user",
      "password": "mon-password"
    }
  }
}
```

**Via variable d'environnement (CI/CD) :**
```bash
# GitHub
export COMPOSER_AUTH='{"github-oauth":{"github.com":"TOKEN"}}'

# GitLab privé
export COMPOSER_AUTH='{"gitlab-token":{"gitlab.example.com":"TOKEN"}}'

# Plusieurs tokens
export COMPOSER_AUTH='{
  "github-oauth": {"github.com": "TOKEN1"},
  "gitlab-token": {"gitlab.example.com": "TOKEN2"}
}'
```

---

## Satis — Repository Composer Privé

Satis génère un repository Composer statique depuis une liste de packages.

```bash
# Installation Satis
composer create-project composer/satis --stability=dev

# Configuration satis.json
cat > satis.json << 'EOF'
{
  "name": "Mon Repository Privé",
  "homepage": "https://packages.example.com",
  "repositories": [
    {
      "type": "vcs",
      "url": "https://gitlab.example.com/mon-org/mon-module.git"
    },
    {
      "type": "vcs",
      "url": "https://gitlab.example.com/mon-org/mon-theme.git"
    }
  ],
  "require-all": true,
  "archive": {
    "directory": "dist",
    "format": "tar",
    "prefix-url": "https://packages.example.com"
  }
}
EOF

# Générer le repository
php bin/satis build satis.json public/

# Utiliser dans composer.json des projets
"repositories": [
  {
    "type": "composer",
    "url": "https://packages.example.com"
  }
]
```

---

## Module Custom en Monorepo

Si le module custom est dans le même dépôt que le site :

```json
// composer.json — path repository
{
  "repositories": [
    {
      "type": "path",
      "url": "web/modules/custom/mon_module",
      "options": {
        "symlink": true    // true = symlink, false = copie
      }
    }
  ],
  "require": {
    "mon-org/mon-module": "*"  // * = toujours la version locale
  }
}
```

---

## Packages.drupal.org — Repository Drupal

```json
// Toujours inclus par défaut dans les projets Drupal
// Pas besoin de l'ajouter manuellement
{
  "repositories": [
    {
      "type": "composer",
      "url": "https://packages.drupal.org/8"
    }
  ]
}
```

---

## CI/CD — Variables d'Environnement

```yaml
# .gitlab-ci.yml — passer COMPOSER_AUTH en variable secrète
variables:
  COMPOSER_AUTH: $COMPOSER_AUTH_SECRET    # Défini dans CI/CD Settings

before_script:
  - composer install --no-interaction --no-progress

# GitLab CI — accès au registry GitLab interne
variables:
  COMPOSER_AUTH: |
    {
      "gitlab-token": {
        "gitlab.example.com": "${CI_JOB_TOKEN}"
      }
    }
```

```yaml
# GitHub Actions
- name: Install Composer dependencies
  env:
    COMPOSER_AUTH: '{"github-oauth":{"github.com":"${{ secrets.COMPOSER_TOKEN }}"}}'
  run: composer install --no-interaction
```
