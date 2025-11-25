# DV Git Hooks Template

Un template réutilisable pour mettre en place des git hooks professionnels avec Husky et lint-staged.

## Structure du Projet

```
DV-Git-Hooks/
├── .github/
│   ├── pull_request_template.md
│   └── workflows/
│       ├── ci.yml
│       └── validate-squash-commit.yml
├── .husky/
│   ├── pre-commit
│   ├── commit-msg (optionnel)
│   ├── pre-push
│   └── post-merge
├── eslint.config.js
├── .prettierrc
├── .node-version
├── commitlint.config.js
├── lint-staged.config.js
├── package.json
├── README.md
└── SETUP.md
```

---

## 1. package.json

```json
{
  "name": "dv-git-hooks",
  "version": "1.0.0",
  "description": "Git hooks template with best practices (ESLint, Prettier, Conventional Commits)",
  "type": "module",
  "engines": {
    "node": ">=22.14.0",
    "pnpm": ">=9.0.0"
  },
  "scripts": {
    "prepare": "husky install",
    "lint": "eslint . --ext .js,.jsx,.ts,.tsx",
    "lint:fix": "eslint . --ext .js,.jsx,.ts,.tsx --fix",
    "format": "prettier --write .",
    "format:check": "prettier --check ."
  },
  "devDependencies": {
    "husky": "^9.1.7",
    "lint-staged": "^15.2.11",
    "eslint": "^9.15.0",
    "@eslint/js": "^9.15.0",
    "eslint-config-prettier": "^9.1.0",
    "eslint-config-airbnb-base": "^15.0.0",
    "eslint-plugin-import": "^2.29.1",
    "prettier": "^3.3.3",
    "@commitlint/cli": "^19.6.0",
    "@commitlint/config-conventional": "^19.6.0"
  },
  "devDependencies-alternatives": {
    "eslint-config-google": "^0.14.0",
    "eslint-config-standard": "^17.1.0",
    "eslint-plugin-n": "^16.6.0"
  }
}
```

**Points clés :**

- `prepare` script s'exécute automatiquement après `pnpm install` pour configurer Husky
- `@commitlint/cli` valide les messages de commit au format Conventional Commits
- ESLint et Prettier sont les standards du marché

---

## 2. eslint.config.js

**Configuration ESLint 9 avec Airbnb (par défaut)**

```javascript
// eslint-disable-next-line import/no-unresolved
import airbnbBase from 'eslint-config-airbnb-base';
import prettier from 'eslint-config-prettier';
import importPlugin from 'eslint-plugin-import';

export default [
  {
    ignores: ['node_modules/', 'dist/', 'build/', '.next/', 'coverage/'],
  },
  {
    files: ['**/*.{js,jsx,ts,tsx}'],
    languageOptions: {
      ecmaVersion: 'latest',
      sourceType: 'module',
      globals: {
        browser: true,
        es2024: true,
        node: true,
      },
    },
    plugins: {
      import: importPlugin,
    },
    rules: {
      ...airbnbBase.rules,
      'no-console': ['warn', { allow: ['warn', 'error'] }],
      'no-unused-vars': ['warn', { argsIgnorePattern: '^_' }],
      'import/extensions': [
        'error',
        'ignorePackages',
        { js: 'never', jsx: 'never', ts: 'never', tsx: 'never' },
      ],
    },
  },
  prettier,
];
```

**Explications :**

- ESLint 9 utilise un système de config plat (flat config)
- `airbnb-base` — les règles strictes d'Airbnb (utilisées avec `...airbnbBase.rules`)
- `prettier` — désactive les règles ESLint en conflit avec Prettier
- `no-console` en warn — Permet les `console.warn` et `console.error` mais pas les `console.log`
- `argsIgnorePattern: "^_"` — Permet les variables non utilisées si elles commencent par `_`

---

## 3. .prettierrc

```json
{
  "semi": true,
  "trailingComma": "es5",
  "singleQuote": true,
  "printWidth": 100,
  "tabWidth": 2,
  "useTabs": false,
  "bracketSpacing": true,
  "arrowParens": "always",
  "endOfLine": "lf"
}
```

**Explications :**

- `printWidth: 100` — limite les lignes à 100 caractères (standard actuel)
- `trailingComma: "es5"` — ajoute des virgules où c'est valide en ES5 (lisibilité + diffs clairs)
- `endOfLine: "lf"` — normalise les fins de ligne (important cross-platform)

---

## 4. .node-version

```
22.14.0
```

Indique la version Node exacte à utiliser. Utilisé par nvm, fnm, etc. Spécifier la version précise garantit une reproductibilité parfaite entre machines et en CI/CD.

---

## 5. commitlint.config.js

```javascript
export default {
  extends: ['@commitlint/config-conventional'],
  rules: {
    'type-enum': [
      2,
      'always',
      ['feat', 'fix', 'docs', 'style', 'refactor', 'perf', 'test', 'chore', 'ci'],
    ],
    'type-case': [2, 'always', 'lowercase'],
    'type-empty': [2, 'never'],
    'scope-case': [2, 'always', 'kebab-case'],
    'subject-empty': [2, 'never'],
    'subject-full-stop': [2, 'never', '.'],
    'header-max-length': [2, 'always', 72],
    'body-leading-blank': [1, 'always'],
    'footer-leading-blank': [1, 'always'],
  },
};
```

**Explications :**

- `header-max-length: 72` — le header limité à 72 caractères (50 pour description + type/scope)
- `scope-case: kebab-case` — les scopes en kebab-case (hyphens)
- Énumère les types acceptés
- Supprimé les règles `subject-case` et `subject-period` qui causaient des conflits en v19

---

## 6. lint-staged.config.js

```javascript
export default {
  '*.{js,jsx,ts,tsx}': ['eslint --fix', 'prettier --write'],
  '*.{json,md,yml,yaml}': ['prettier --write'],
};
```

**Explications :**

- Sur les fichiers JS/TS : lance ESLint puis Prettier
- Sur les autres fichiers : juste Prettier
- Le `--fix` applique automatiquement les corrections

---

## 7. .husky/pre-commit

```bash
#!/bin/sh
. "$(dirname "$0")/_/husky.sh"

# Run lint-staged (ESLint + Prettier on staged files)
pnpm lint-staged

# Check if there are any errors
if [ $? -ne 0 ]; then
  echo "❌ Pre-commit checks failed. Fix the issues and try again."
  exit 1
fi

echo "✅ Pre-commit checks passed"
```

**Qu'est-ce que ça fait :**

- Lance `lint-staged` qui exécute ESLint et Prettier sur les fichiers staged uniquement
- Bloque le commit si ça échoue
- Affiche un message clair

---

## 8. .husky/commit-msg (OPTIONNEL)

⚠️ **Ce hook est optionnel.** Avec un workflow squash + PR template, tes commits locaux n'importent pas.

```bash
#!/bin/sh
. "$(dirname "$0")/_/husky.sh"

# Validate commit message format with commitlint
pnpm commitlint --edit $1

if [ $? -ne 0 ]; then
  echo "❌ Commit message does not follow Conventional Commits format"
  echo ""
  echo "Format: type(scope): description (max 50 chars)"
  echo "Example: feat(auth): add JWT token validation"
  echo ""
  echo "Types: feat, fix, docs, style, refactor, perf, test, chore, ci"
  exit 1
fi

echo "✅ Commit message is valid"
```

**À savoir :** Si tu utilises un workflow avec squash + PR template, tu n'as pas besoin de ce hook. Tu peux le désactiver en supprimant le fichier `.husky/commit-msg`.

---

## 9. .husky/pre-push

```bash
#!/bin/sh
. "$(dirname "$0")/_/husky.sh"

echo "🔍 Running pre-push checks..."

# Get current branch name
BRANCH=$(git rev-parse --abbrev-ref HEAD)

# Validate branch name format: type/scope/description (with hyphens)
# Format: feat/auth-module/add-login or hotfix/critical-bug
if ! echo "$BRANCH" | grep -qE '^(feat|fix|docs|style|refactor|perf|test|chore|ci|hotfix|experiment)/[a-z0-9-]+/[a-z0-9-]+$'; then
  echo "❌ Branch name does not follow the convention"
  echo ""
  echo "Expected format: type/scope/description (all lowercase, hyphens only)"
  echo "Types: feat, fix, docs, style, refactor, perf, test, chore, ci, hotfix, experiment"
  echo "Scope: module/component name (e.g., auth-module, ui, api)"
  echo "Description: what the branch is about (e.g., add-login-validation)"
  echo ""
  echo "Example: feat/auth-module/add-login-validation"
  echo "Current branch: $BRANCH"
  exit 1
fi

echo "✅ Branch name is valid"

# If you want to add tests here in the future, uncomment:
# echo "🧪 Running tests..."
# pnpm test
# if [ $? -ne 0 ]; then
#   echo "❌ Tests failed. Fix them and try again."
#   exit 1
# fi
# echo "✅ Tests passed"

echo "✅ Pre-push checks passed"
```

**Qu'est-ce que ça fait :**

- Valide que le nom de branche suit le format `type/scope/description`
- Utilise une regex pour être strict
- Affiche un guide clair si le format est mauvais
- Section tests commentée (à activer quand tu auras des tests)

---

## 10. .husky/post-merge

```bash
#!/bin/sh
. "$(dirname "$0")/_/husky.sh"

echo "🔄 Running post-merge checks..."

# Check if pnpm-lock.yaml changed
if git diff-index --cached --name-only HEAD | grep -q "pnpm-lock.yaml"; then
  echo "📦 pnpm-lock.yaml has changed, installing dependencies..."
  pnpm install
  if [ $? -ne 0 ]; then
    echo "⚠️  pnpm install failed. Run 'pnpm install' manually."
    exit 1
  fi
  echo "✅ Dependencies installed"
fi

# Check if .node-version or .nvmrc changed (optional warning)
if git diff-index --cached --name-only HEAD | grep -qE "(\.node-version|\.nvmrc)"; then
  NODE_VERSION=$(cat .node-version 2>/dev/null || cat .nvmrc 2>/dev/null)
  echo "⚠️  Node version has changed to: $NODE_VERSION"
  echo "Consider running 'nvm use' or updating your Node version"
fi

echo "✅ Post-merge checks completed"
```

**Qu'est-ce que ça fait :**

- Détecte si `pnpm-lock.yaml` a changé et lance `pnpm install` automatiquement
- Détecte les changements de version Node et te prévient
- Non-bloquant (il continue même si ça échoue, mais te prévient)

---

## 11. .github/pull_request_template.md

```markdown
## Description

<!-- Décris les changements apportés -->

## Type de changement

- [ ] 🎉 Nouvelle fonctionnalité (feat)
- [ ] 🐛 Correction de bug (fix)
- [ ] 📚 Documentation (docs)
- [ ] 🎨 Style/Formatage (style)
- [ ] ♻️ Refactoring (refactor)
- [ ] ⚡ Performance (perf)
- [ ] ✅ Tests (test)
- [ ] 🔧 CI/Config (ci)

## Lié à

<!-- Remplace #123 par le numéro de ton issue -->

Closes #

## Checklist

- [ ] Mon code suit les conventions du projet
- [ ] J'ai validé les changements localement
- [ ] J'ai ajouté des tests si applicable
- [ ] La documentation est à jour
```

**À savoir :** Ce template s'affiche automatiquement quand on crée une PR. Utile pour guider et documenter.

---

## 12. .github/workflows/ci.yml

```yaml
name: CI - Git Hooks & Code Quality

on:
  push:
    branches: [main, develop]
  pull_request:
    branches: [main, develop]

jobs:
  validate:
    runs-on: ubuntu-latest

    steps:
      - uses: actions/checkout@v4
        with:
          fetch-depth: 0

      - uses: pnpm/action-setup@v2
        with:
          version: latest

      - uses: actions/setup-node@v4
        with:
          node-version-file: '.node-version'
          cache: 'pnpm'

      - name: Install dependencies
        run: pnpm install

      - name: Run ESLint
        run: pnpm lint

      - name: Check formatting with Prettier
        run: pnpm format:check

      - name: Branch name validation
        if: github.event_name == 'push'
        run: |
          BRANCH_NAME="${{ github.ref_name }}"
          if ! echo "$BRANCH_NAME" | grep -qE '^(feat|fix|docs|style|refactor|perf|test|chore|ci|hotfix|experiment)/[a-z0-9-]+/[a-z0-9-]+$'; then
            echo "❌ Branch name does not follow convention: $BRANCH_NAME"
            exit 1
          fi
          echo "✅ Branch name is valid: $BRANCH_NAME"
```

**Qu'est-ce que ça fait :**

- **Sur les PRs** : lint le code et valide la formatage
- **Sur les pushes** : valide les noms de branche
- Utilise Node 22.14.0 et pnpm
- C'est bloquant — les PRs ne peuvent pas merger si ça échoue

---

## 13. .github/workflows/validate-squash-commit.yml

```yaml
name: Validate Squash Commit Message

on:
  pull_request:
    types: [closed]

jobs:
  validate-commit:
    # Seulement si la PR a été mergée
    if: github.event.pull_request.merged == true
    runs-on: ubuntu-latest

    steps:
      - uses: actions/checkout@v4
        with:
          fetch-depth: 0

      - uses: pnpm/action-setup@v2
        with:
          version: latest

      - uses: actions/setup-node@v4
        with:
          node-version-file: '.node-version'
          cache: 'pnpm'

      - name: Install dependencies
        run: pnpm install

      - name: Validate merge commit message
        run: |
          # Récupère le message du commit de merge
          COMMIT_MSG=$(git log -1 --pretty=%B)

          echo "Commit message: $COMMIT_MSG"

          # Valide avec commitlint
          echo "$COMMIT_MSG" | pnpm commitlint

          if [ $? -ne 0 ]; then
            echo "❌ Merge commit message does not follow Conventional Commits"
            echo ""
            echo "Expected format: type(scope): description (#PR_NUMBER)"
            echo "Example: feat(auth): add JWT token validation (#35)"
            exit 1
          fi

          echo "✅ Merge commit message is valid"
```

**À savoir :**

- Ce workflow s'exécute **après un merge** (quand une PR est fermée et mergée)
- Il valide que le message de commit squash suit Conventional Commits
- C'est informatif (il ne bloque pas le merge, mais il te prévient si c'est mauvais)
- Utile pour garder l'historique propre sur `main`

---

## Guide Complet des Hooks

### Pre-Commit

**Quand :** Avant de créer un commit
**Qu'est-ce :** Valide l'ESLint et formate avec Prettier sur les fichiers staged
**Bloquant :** Oui
**Quoi faire si ça échoue :** Fixes les erreurs que ESLint affiche, les corrections se font souvent automatiquement avec `--fix`

### Commit-Msg (OPTIONNEL)

**Quand :** Après avoir écrit le message de commit
**Qu'est-ce :** Valide que le message suit Conventional Commits
**Bloquant :** Oui
**À savoir :** Ce hook est optionnel car avec un workflow squash + PR, tes commits locaux n'importent pas. Tu peux le désactiver en supprimant `.husky/commit-msg`.

### Pre-Push

**Quand :** Avant de pusher le code
**Qu'est-ce :** Valide le nom de branche au format `type/scope/description`
**Bloquant :** Oui
**Exemple valide :** `feat/auth-module/add-login-validation`

### Post-Merge

**Quand :** Après un merge (pull ou rebase)
**Qu'est-ce :** Installe les dépendances si `pnpm-lock.yaml` a changé, prévient si Node version change
**Bloquant :** Non (informatif)

---

## Workflow Recommandé : Squash + PR Template

**Le workflow optimal pour un historique clean :**

```
1. Crée une branche : feat/auth-module/add-login
2. Commite autant que tu veux (pas besoin de format parfait)
3. Crée une PR (pré-remplie automatiquement)
4. Review + merge en squash
5. À la demande de GitHub, écris un message Conventional Commits : feat(auth): add JWT token validation (#35)
6. Résultat : 1 commit dans main = 1 feature clean
```

**Avantages :**

- Liberté dans les commits locaux
- Historique de main propre et clair
- Release notes automatiques (1 commit = 1 version)
- Messages de commit centrés au bon endroit (au merge)

---

## Installation & Setup

### Pour ce projet template :

```bash
# 1. Initialiser le git si pas encore fait
git init

# 2. Installer les dépendances
pnpm install

# 3. Husky s'initialise automatiquement (script prepare)
# Les hooks sont créés dans .husky/

# 4. Rendre les hooks exécutables (Mac/Linux)
chmod +x .husky/*

# 5. Test les hooks
# Fais un test commit pour voir si tout fonctionne
```

### Pour utiliser dans un autre projet :

```bash
# 1. Copie les fichiers suivants du template vers ton projet:
# - .husky/ (dossier complet)
# - eslint.config.js (remplace .eslintrc.json s'il existe)
# - .prettierrc
# - commitlint.config.js
# - lint-staged.config.js
# - .node-version

# 2. Supprime .eslintrc.json si tu l'as (ESLint 9 n'en a pas besoin)
rm .eslintrc.json

# 3. Installe les dépendances
pnpm install

# 4. Husky s'initialise automatiquement

# 5. Rendre les hooks exécutables (Mac/Linux)
chmod +x .husky/*
```

---

## ESLint 9 Standards Expliqués

### 🏆 Airbnb (Défaut - Recommandé)

**À installer :**

```bash
pnpm add -D eslint-config-airbnb-base eslint-plugin-import
```

**Config eslint.config.js :**

```javascript
import airbnbBase from 'eslint-config-airbnb-base';
import importPlugin from 'eslint-plugin-import';
import prettier from 'eslint-config-prettier';

export default [
  {
    plugins: { import: importPlugin },
    rules: airbnbBase.rules,
  },
  prettier,
];
```

**Caractéristiques :**

- Stricte et très structuré
- Le standard le plus utilisé dans l'industrie (France + international)
- Bon pour apprendre les vraies bonnes pratiques
- Beaucoup de règles, peut être "loud" au début
- **Quand l'utiliser :** Projets professionnels, template, quand tu veux montrer que tu maîtrises les standards

---

### 📦 ESLint Recommended (Léger)

**À installer :** Rien, c'est déjà inclus avec ESLint 9

**Config eslint.config.js :**

```javascript
import js from '@eslint/js';
import prettier from 'eslint-config-prettier';

export default [js.configs.recommended, prettier];
```

**Caractéristiques :**

- Les règles de base ESLint 9
- Zéro dépendance externe
- Couvert les vrais problèmes (pas de var, unused variables, etc)
- Permissif, pas strict sur le style
- **Quand l'utiliser :** Projets perso, prototypes, quand tu veux juste les basics

---

### 🔵 Google Standard

**À installer :**

```bash
pnpm add -D eslint-config-google
```

**Config eslint.config.js :**

```javascript
import google from 'eslint-config-google';
import prettier from 'eslint-config-prettier';

export default [google, prettier];
```

**Caractéristiques :**

- Standard de Google, équilibre entre stricte et léger
- Moins populaire qu'Airbnb
- Bon alternative si tu trouves Airbnb trop strict
- **Quand l'utiliser :** Si tu préfères une approche plus légère qu'Airbnb

---

### ⭐ Standard.js

**À installer :**

```bash
pnpm add -D eslint-config-standard eslint-plugin-n
```

**Config eslint.config.js :**

```javascript
import standard from 'eslint-config-standard';
import prettier from 'eslint-config-prettier';

export default [standard, prettier];
```

**Caractéristiques :**

- Standard communautaire "zero config"
- Léger et simple
- Moins populaire en France
- **Quand l'utiliser :** Projets communautaires, si tu aimes la philosophie "zero config"

---

## Comment Switcher de Standard

ESLint 9 utilise le "Flat Config" system. Pour switcher :

1. **Installe les dépendances** du standard que tu veux
2. **Modifie `eslint.config.js`** et change les imports + l'array export
3. **Relance** : `pnpm lint` pour voir les différences
4. **Fix** : `pnpm lint:fix` pour corriger automatiquement

Exemple : Passer d'Airbnb à ESLint Recommended

```javascript
// Avant (Airbnb)
import airbnbBase from 'eslint-config-airbnb-base';
export default [{ ...airbnbBase }, prettier];

// Après (Recommended)
import js from '@eslint/js';
export default [js.configs.recommended, prettier];
```

---

## Personnalisation

### Ajouter des types de branche supplémentaires

Modifie le regex dans `.husky/pre-push` et `.github/workflows/ci.yml` :

```bash
# Actuel :
'^(feat|fix|docs|style|refactor|perf|test|chore|ci|hotfix|experiment)/...'

# Ajoute un type, par exemple "wip" (work in progress) :
'^(feat|fix|docs|style|refactor|perf|test|chore|ci|hotfix|experiment|wip)/...'
```

### Ajouter des règles ESLint

Modifie `.eslintrc.json` dans la section `rules`.

### Ajouter des tests

Décommente la section tests dans `.husky/pre-push` et configure ton framework (Vitest, Jest, etc).

### Adapter Prettier

Modifie `.prettierrc` selon tes préférences (tabWidth, printWidth, etc).

---

## Troubleshooting

**Q: Les hooks ne s'exécutent pas**
R: Vérifie que `pnpm prepare` a bien créé le dossier `.husky`. Relance `pnpm install`.

**Q: "Permission denied" sur les hooks**
R: Sur Mac/Linux, les fichiers `.husky/*` doivent être exécutables. Lance : `chmod +x .husky/*`

**Q: ESLint/Prettier "command not found"**
R: Assure-toi que les dépendances sont installées. Relance `pnpm install`.

**Q: Je dois forcer un commit même si ça échoue**
R: `git commit --no-verify` (mais c'est une mauvaise idée, fixe le vrai problème)

**Q: Husky affiche un warning "DEPRECATED" sur la syntaxe des hooks**
R: C'est un warning pour Husky v10 qui sortira dans le futur. Les hooks actuels (`#!/usr/bin/env sh` + `. "$(dirname -- "$0")/_/husky.sh"`) vont être dépréciés. Pas urgent pour l'instant, mais tu devras updater cette syntaxe quand v10 sortira. Suis les recommandations de Husky à ce moment-là.

---

## Maintenance Future

### Husky v10 Migration

Quand Husky v10 sortira, les fichiers `.husky/*` devront être migrés vers la nouvelle syntaxe. À ce moment :

1. Consulte la [migration guide de Husky](https://typicode.github.io/husky/)
2. Mets à jour tous les fichiers dans `.husky/` avec la nouvelle syntaxe
3. Teste avec `pnpm install` et `pnpm lint`
4. Update ce template en conséquence

---

## Prochaines étapes

1. **Configure tes règles ESLint** selon ta préférence
2. **Adapte les types de branche** si nécessaire
3. **Ajoute des tests** quand tu as des tests à lancer
4. **Copie la structure** dans tes autres projets
5. **Raffine GitHub Actions** au fil du temps selon tes besoins réels
