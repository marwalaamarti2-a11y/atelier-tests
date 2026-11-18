# Atelier 1 : la pyramide de tests sur une API Node.js

[![CI](https://github.com/marwalaamarti2-a11y/atelier-tests/actions/workflows/ci.yml/badge.svg)](https://github.com/marwalaamarti2-a11y/atelier-tests/actions/workflows/ci.yml)

Projet réalisé dans le cadre du module **DevOps** (ENSA Fès, cycle ingénieur, Génie Informatique). Une mini-boutique Node.js sert de fil rouge pour construire la pyramide de tests complète et l'automatiser dans un pipeline CI avec GitHub Actions.

## Objectifs

- Écrire les 3 niveaux de tests : unitaires, d'intégration et fonctionnels (E2E).
- Mesurer la couverture de code et dépasser 80 %.
- Automatiser les tests dans une CI et savoir quand lancer chaque niveau.

## La pyramide des tests

| Niveau | Outils | Ce qui est vérifié | Quand en CI |
|---|---|---|---|
| Unitaires | Jest | Fonctions isolées (`total`, `validerCommande`), mailer remplacé par un mock | À chaque commit |
| Intégration | Jest, Supertest, PostgreSQL (Docker) | API + accès aux données + vraie base : codes 200, 404, 400, résistance à l'injection SQL | À chaque pull request |
| E2E | Playwright (Chromium) | Parcours de connexion réussi et échoué dans un vrai navigateur | Sur staging, avant la production |

Plus on monte dans la pyramide, plus les tests sont lents et coûteux : on en écrit donc moins.

## Résultats

- **Couverture de code : 93,75 %** (tests unitaires, objectif ≥ 80 %).
- **12 tests, tous verts** : 6 unitaires, 4 d'intégration, 2 E2E.
- **Pipeline CI vert** : job `unit` à chaque push, job `integration` (avec service PostgreSQL) sur les pull requests.

| | Unitaires | Intégration | E2E |
|---|---|---|---|
| Durée mesurée | 3,4 s | 2,8 s | 2,1 s |

## Structure du projet

```
atelier-tests/
├── .github/workflows/ci.yml     # pipeline GitHub Actions
├── e2e/login.spec.js            # scénarios Playwright
├── public/                      # pages login.html et dashboard.html
├── src/
│   ├── panier.js                # calcul du total, refus d'un prix négatif
│   ├── commande.js              # validation de commande
│   ├── mailer.js                # service d'e-mail (mocké dans les tests)
│   ├── db.js                    # connexion PostgreSQL
│   ├── app.js                   # API Express (/users/:id, /login)
│   └── server.js                # démarrage du serveur
├── tests/
│   ├── unit/                    # panier.test.js, commande.test.js
│   └── integration/             # users.test.js
├── docker-compose.yml           # base PostgreSQL de test
└── playwright.config.js
```

## Prérequis

- Node.js 20 ou plus et npm
- Docker avec Docker Compose
- Git

## Installation et lancement

```bash
git clone https://github.com/marwalaamarti2-a11y/atelier-tests.git
cd atelier-tests
npm ci
npx playwright install chromium
```

### Tests unitaires (avec couverture)

```bash
npm run test:unit
```

Le rapport détaillé s'ouvre dans `coverage/lcov-report/index.html`.

### Tests d'intégration

Démarrer d'abord la base PostgreSQL de test :

```bash
docker compose up -d
npm run test:int
```

La base est exposée sur le port **5433** de la machine (pour ne pas entrer en conflit avec un PostgreSQL local déjà installé). L'adresse peut être changée avec la variable d'environnement `DATABASE_URL`. En CI, elle pointe vers le port 5432 du service PostgreSQL du runner.

### Tests E2E

```bash
npm run test:e2e
npx playwright show-report
```

Playwright démarre l'application automatiquement avant les tests.

## Pipeline CI (GitHub Actions)

Le fichier `.github/workflows/ci.yml` définit deux jobs :

- **unit** : s'exécute à chaque push et pull request (installation avec `npm ci`, puis `npm run test:unit`).
- **integration** : s'exécute uniquement sur les pull requests, avec un conteneur PostgreSQL démarré comme service du job et une vérification de disponibilité (`pg_isready`) avant les tests.

## Points techniques notables

- Démarche **rouge → vert** pour la règle du prix négatif.
- **Mock** du mailer avec Jest pour isoler la logique de commande.
- **Requêtes SQL paramétrées** et validation de l'entrée, avec un test qui simule une injection SQL.
- Base de test **recréée avant chaque test** (`beforeEach`) pour garantir des tests indépendants et reproductibles.
- Historique Git lisible, avec un commit par étape (convention `test:`, `feat:`, `ci:`, `chore:`).

## Limites connues

Les identifiants de la route `/login` sont codés en dur uniquement pour l'atelier. La gestion des secrets est l'objet d'un atelier ultérieur.

## Auteure

**Marwa Laamarti** : élève ingénieure en Génie Informatique, ENSA Fès.
[GitHub](https://github.com/marwalaamarti2-a11y) · [LinkedIn](https://www.linkedin.com/in/marwa-laamarti-9b384b318)
