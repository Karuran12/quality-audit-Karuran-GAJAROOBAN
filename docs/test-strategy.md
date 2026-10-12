# Stratégie de test

## Contexte

QuickOrder est une application de livraison de repas. Sur le site on peut créer un compte, choisir un restaurant, remplir un panier, appliquer un code promo, payer (carte, PayPal ou espèces) puis suivre ou annuler sa commande. Il existe aussi un espace administrateur et une API REST documentée avec Swagger.

L'application est déjà développée et fonctionnelle. Notre rôle à nous n'est pas de la modifier mais de vérifier si elle peut partir en production. On cherche à savoir si les règles et les normes sont respectées et qu'il n'y a pas d'erreur.


## Environnement de test

- Application lancée en local avec `python seed.py` puis `python run.py`
- Adresse: http://127.0.0.1:5000
- Swagger: http://127.0.0.1:5000/docs/ avec la clé `quickorder-demo-key`
- Comptes fournis :
  - client: alice@example.com / Password123!
  - client: bob@example.com / Password123!
  - admin: admin@quickorder.local / Admin123!
- Navigateur: Safari (navigateur d'Apple)
- Outils API: Swagger

Avant chaque série de tests, on relance `python seed.py`, comme l'indique le README, ainsi on peut repartir des mêmes données de départ afin que les tests soient indépendants les uns des autres.

## 1. Périmètre

Ce qu'on souhaite tester sur le site pour voir sa complexité et ses faiblesses:

- **Compte utilisateur**: inscription, connexion, déconnexion et accès à l'espace admin
- **Restaurants**: liste, recherche, filtres (catégorie, prix de livraison, végétarien), tri et consultation des menus
- **Panier**: ajout de produits, choix de la quantité, mise à jour et panier limité à un seul restaurant
- **Commande**: adresse de livraison, minimum de commande, restaurant ouvert ou fermé, produit en stock ou non
- **Codes promo**: code valide, expiré, montant minimum, plafond de réduction
- **Paiement simulé**: carte, PayPal, espèces avec calcul de la monnaie
- **Suivi**: liste des commandes, détail, annulation, recommander une ancienne commande
- **Administration**: changement du statut d'une commande
- **Avis**: dépôt d'une note et d'un commentaire
- **API REST**: clé API, restaurants, produits, coupons, création et annulation de commande, codes HTTP renvoyés, format JSON

## 2. Hors périmètre

On ne teste pas:

- **La performance et la montée en charge**: on n'a ni l'environnement ni les outils pour ça.
- **Les vrais moyens de paiement**: le paiement est simulé dans l'application, il n'y a pas de banque ni de PayPal derrière.
- **Un test d'intrusion complet**: on fait seulement des vérifications de sécurité de base (accès admin, clé API, pages d'erreur, redirections).
- **La compatibilité navigateurs et mobile**: tout est testé sur le navigateur de l'ordinateur (ici Safari).
- **L'accessibilité**: on notera ce qu'on remarque, mais on ne fait pas d'audit complet.
- **Le code source**: on ne le modifie pas pendant la campagne. On pourra le lire pour comprendre l'origine d'un bug.

## 3. Parcours critiques

Ce sont les parcours qui posent le plus de problèmes s'ils ne marchent pas. On doit les tester en premier car une erreur sur ces parcours a plus conséquences.

| N° | Parcours | Pourquoi c'est critique |
|----|----------|------------------------|
| P1 | Connexion -> choix d'un restaurant -> panier -> paiement par carte -> confirmation | C'est le parcours typique d'un utilisateur lambda. Il est important que tout marche sinon le site n'est pas fonctionnel |
| P2 | Paiement PayPal et paiement en espèces | Une somme fausse peut faire payer un client inutilement, donc plus de réclamations |
| P3 | Commande d'un produit en rupture de stock | Si la commande passe, le restaurant ne pourra pas la préparer donc erreur |
| P4 | Commande dans un restaurant fermé ou sous le minimum de commande | Une commande impossible à livrer chez le client |
| P5 | Application d'un code promo | Une erreur de calcul fait perdre de l'argent |
| P6 | Annulation d'une commande et changement de statut par l'admin | Il faut gérer le stock et le remboursement (éviter des statuts d'erreur) |
| P7 | Création de commande via l'API | L'API doit appliquer les mêmes règles que le site |
| P8 | Accès aux données et à l'administration | Un client ne doit voir que ses commandes et ne doit pas pouvoir accéder au compte de l'admin |

## 4. Risques

On liste les risques à partir de la documentation et des fonctionnalités annoncées, avant de lancer les tests.

| ID | Risque | Conséquence |
|----|--------|-------------|
| R1 | Le paiement crée plusieurs commandes ou accepte un montant insuffisant | Utilisateur débité deux fois, perte de client ou beaucoup de réclamations à traiter |
| R2 | Le stock n'est pas vérifié au moment de la commande | Commandes impossibles à livrer même si elles sont validées sur l'application |
| R3 | Les règles de commande ne sont pas appliquées (restaurant fermé, minimum, adresse) | Commandes bloquées côté restaurant ou livreur |
| R4 | Un code promo est mal calculé, accepté alors qu'il a expiré ou différent de ce qui est affiché | Avantage censé attirer les clients devient donc inutile et les pousse à moins commander |
| R5 | L'API n'applique pas les mêmes règles que le site | Une personne peut contourner les contrôles en passant par l'API |
| R6 | Faille de sécurité simple (accès admin, données d'un autre client) | Fuite de données donc moins de nouveaux clients et inquiétude sur la sécurité des données privées des utilisateurs |
| R7 | Le statut d'une commande peut revenir en arrière ou l'annulation ne rend pas le stock | Incohérences dans le suivi et la comptabilité |
| R8 | L'API renvoie des erreurs mal formées (erreur 500, page HTML au lieu de JSON) | Les applications qui utilisent l'API ne savent pas gérer l'erreur |
| R9 | La recherche ou les filtres renvoient de mauvais résultats | L'utilisateur ne trouve pas ce qu'il cherche, donc des clients en moins |
| R10 | Les avis peuvent être déposés par n'importe qui | Notes peu fiables, donc les clients ont moins confiance pour commander |
| R11 | La connexion ou l'inscription ne fonctionne pas | Plus aucun client ne peut commander |
| R12 | Swagger ou Docker ne se lancent pas | Gêne pour les développeurs |

## 5. Types de tests

| Type de test | Ce qu'on fait concrètement | Outil |
|--------------|----------------------------|-------|
| Tests fonctionnels (interface) | On suit les parcours comme un client souhaitant commander | Navigateur |
| Tests d'API | On appelle chaque endpoint avec des données correctes puis incorrectes et on vérifie le code HTTP et le JSON renvoyé par celui-ci | Swagger |
| Valeurs limites | On teste juste avant et juste après chaque seuil: quantité 0/1 et 20/21, montant 19,99 € / 20,00 €, stock disponible | Swagger, Navigateur |
| Cas d'erreur | On envoie des données invalides et on vérifie que l'application refuse avec un message clair | Navigateur, Swagger |
| Règles métier | On vérifie les contraintes de l'entreprise: minimum de commande, panier mono-restaurant, paiement, annulation | Navigateur, Swagger |
| Transitions d'état | On fait changer une commande de statut et on regarde si les retours en arrière sont bloqués | Espace admin |
| Sécurité de base | Accès admin avec un compte client, appel API sans clé, accès à la commande d'un autre client, contenu des pages d'erreur | Navigateur, Inspecteur web (DevTools de Safari), Swagger |
| Tests exploratoires | Courtes sessions libres pour trouver ce que nos cas n'avaient pas prévu | Navigateur |
| Tests utilisateurs | Trois scénarios réalisés par au moins deux personnes extérieures au projet | Observation + prise de notes |

## 6. Priorisation risque × impact

On note chaque risque selon deux critères:

- **Probabilité** que le problème arrive: 1 faible, 2 moyenne, 3 forte
- **Impact** s'il arrive: 1 simple gêne, 2 perte limitée, 3 perte d'argent, de données ou de confiance

Le score est le produit des deux: **Probabilité × Impact**.

| Risque | Probabilité | Impact | Score | Priorité |
|--------|:-----------:|:------:|:-----:|----------|
| R1 Paiement | 3 | 3 | 9 | Haute |
| R2 Stock | 3 | 3 | 9 | Haute |
| R3 Règles de commande | 3 | 2 | 6 | Haute |
| R4 Codes promo | 3 | 2 | 6 | Haute |
| R5 Écart site / API | 3 | 2 | 6 | Haute |
| R6 Sécurité | 2 | 3 | 6 | Haute |
| R7 Statuts et annulation | 2 | 2 | 4 | Moyenne |
| R8 Erreurs de l'API | 2 | 2 | 4 | Moyenne |
| R11 Connexion | 1 | 3 | 3 | Moyenne |
| R9 Recherche et filtres | 3 | 1 | 3 | Moyenne |
| R10 Avis | 2 | 1 | 2 | Basse |
| R12 Swagger / Docker | 1 | 1 | 1 | Basse |

La classification des notes:

- **Score de 6 à 9 (haute)**: on écrit des cas nominaux, limites, d'erreur et de règle métier. On teste à la fois sur le site et sur l'API.
- **Score de 3 à 4 (moyenne)**: on teste le cas normal et les erreurs principales.
- **Score de 1 à 2 (basse)**: une vérification rapide ou un test exploratoire suffit.

## Critères de fin de campagne

On considère que l'application peut partir en production si:

- tous les cas prévus ont été exécutés, ou qu'on sait expliquer pourquoi certains ne l'ont pas été 
- il ne reste aucun bug critique ouvert 
- les parcours P1 à P4 passent tous 
- les corrections ont été retestées

