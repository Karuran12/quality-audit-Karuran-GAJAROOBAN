# Audit qualité de QuickOrder

Projet réalisé dans le cadre du cours "Tests et qualité des solutions numériques".

## Équipe

- Karuran GAJAROOBAN

## Le projet

QuickOrder est une application de livraison de repas développée en Flask. On peut y choisir un restaurant, remplir un panier, utiliser un code promo, payer et suivre sa commande. Il existe aussi un espace administrateur et une API REST.

Notre mission était de jouer le rôle d'une équipe QA avant la mise en production: tester l'application sans modifier son code, trouver les anomalies et dire si elle peut être mise en ligne.

## Résultat en bref

| Tests | Pass | Fail | Blocked | Skipped | Bugs trouvés |
|---|---|---|---|---|---|
| 24 | 15 | 9 | 0 | 0 | 8 (dont 2 critiques) |

**Recommandation: No-Go.**

Les parcours simples fonctionnent bien mais la plupart des règles métier ne sont pas respectées. Les problèmes les plus graves sont un paiement PayPal qui crée deux commandes, un stock qui n'est jamais vérifié, un paiement en espèces accepté avec un montant trop faible et des commandes acceptées dans un restaurant fermé.

Le détail est dans le rapport d'exécution.

## Contenu du dossier

| Fichier | Contenu |
|---|---|
| [docs/test-strategy.md](docs/test-strategy.md) | Stratégie de test: périmètre, parcours critiques, risques, types de tests |
| [docs/risk-matrix.md](docs/risk-matrix.md) | Matrice des risques avec probabilité, impact et résultat après la campagne |
| [docs/test-cases.xlsx](docs/Test-Cases.xlsx) | Les 24 cas de test avec leur résultat |
| [docs/execution-report.md](docs/execution-report.md) | Analyse de la campagne et recommandation Go / No-Go |
| [docs/bug-reports/](docs/bug-reports/README.md) | Les 8 bug reports |
| [docs/user-feedback.md](docs/user-feedback.md) | Tests utilisateurs avec 2 participants et 3 scénarios |
| [docs/improvement-plan.md](docs/improvement-plan.md) | Plan d'amélioration avec 7 actions priorisées |
| docs/evidence/ | Captures d'écran utilisées comme preuves |

## Environnement de test

- QuickOrder V3.1 lancé en local avec `python seed.py` puis `python run.py`
- Navigateur: Safari avec l'Inspecteur web
- API: Swagger sur http://127.0.0.1:5000/docs/ avec la clé `quickorder-demo-key`
- Comptes utilisés: alice@example.com, bob@example.com et admin@quickorder.local

## Refaire les tests

1. Lancer QuickOrder avec `python seed.py` puis `python run.py`
2. Ouvrir http://127.0.0.1:5000 dans le navigateur
3. Suivre les étapes de chaque cas dans `docs/test-cases.xlsx`

Il faut relancer `python seed.py` avant chaque série de tests: chaque commande modifie le stock et les résultats ne seraient plus les mêmes.