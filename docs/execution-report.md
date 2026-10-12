# Rapport d'exécution de la campagne de test

## Contexte de la campagne

La campagne a été faite sur QuickOrder V3.1 lancé en local avec `python seed.py` puis `python run.py`. On n'a pas modifié le code de l'application pendant les tests.

Outils utilisés:

- Safari pour tester le site comme un vrai client
- l'Inspecteur web de Safari (DevTools) pour regarder les requêtes envoyées au serveur
- Swagger avec la clé `quickorder-demo-key` pour tester l'API

Avant chaque série de tests on a relancé `python seed.py` pour repartir des mêmes données. C'est important parce que chaque commande fait baisser le stock et sans ça les résultats d'un test pouvaient dépendre de ce qu'on avait fait juste avant.

Le détail de chaque cas est dans le fichier `test-cases.xlsx` et les captures sont dans le dossier `docs/evidence/`.

## Résultats

| | Nombre |
|---|---|
| Tests prévus | 24 |
| Tests exécutés | 24 |
| Pass | 15 |
| Fail | 9 |
| Blocked | 0 |
| Skipped | 0 |

Taux de réussite: 15 sur 24 soit 62,5 %.

### Résultats par catégorie

| Catégorie | Total | Pass | Fail |
|---|---|---|---|
| Nominaux | 6 | 6 | 0 |
| Limites | 6 | 5 | 1 |
| Erreurs | 6 | 3 | 3 |
| Règles métier | 6 | 1 | 5 |
| **Total** | **24** | **15** | **9** |

### Pourquoi il n'y a aucun Blocked ni Skipped

Tous les cas prévus ont pu être exécutés jusqu'au bout. L'application s'est lancée sans problème, Swagger fonctionnait avec la clé fournie et les comptes de démonstration marchaient. On n'a donc eu aucun test bloqué par l'environnement et on n'a pas eu besoin d'en mettre de côté.

## Ce que montrent les résultats

Ce qui ressort le plus c'est la différence entre les cas nominaux et les règles métier. Tous les cas nominaux passent: on peut se connecter, voir les menus, remplir un panier, modifier les quantités, payer par carte et utiliser un code promo valide sans aucun souci. Par contre dès qu'on teste une règle que l'application est censée faire respecter, presque tout échoue puisque 5 règles métier sur 6 sont en Fail.

En clair l'application fonctionne bien tant que l'utilisateur fait exactement ce qui est prévu mais elle ne se protège pas du tout quand il fait quelque chose d'inhabituel, comme commander un plat épuisé ou payer avec un montant trop faible.

Les cas limites sur l'API passent presque tous: les quantités 0 et 21 sont bien refusées, le seuil du code promo à 20 € et son plafond de 8 € sont respectés. Le seul échec dans les limites concerne le stock, qui n'est jamais contrôlé.

## Bugs trouvés

Les 9 cas en Fail correspondent à 8 bugs différents. TC-L03 et TC-E02 montrent le même problème de stock, une fois avec l'API et une fois sur le site.

| Bug | Titre | Sévérité | Priorité |
|---|---|---|---|
| BUG-01 | Un paiement PayPal crée deux commandes | Critique | P1 |
| BUG-02 | Le stock n'est pas vérifié à la commande | Critique | P1 |
| BUG-03 | Paiement en espèces accepté avec un montant insuffisant | Majeur | P1 |
| BUG-04 | Un restaurant fermé accepte les commandes | Majeur | P1 |
| BUG-05 | Le minimum de commande n'est pas appliqué | Majeur | P1 |
| BUG-06 | Un nouveau client peut commander sans adresse | Majeur | P2 |
| BUG-07 | Annuler une commande ne remet pas le stock | Majeur | P2 |
| BUG-08 | La promo ATLAS15 applique 10 % au lieu de 15 % | Majeur | P2 |

Soit 2 bugs critiques et 6 bugs majeurs. 5 bugs sont en priorité P1 et doivent être corrigés avant une mise en production.

## Principaux risques

Si on compare avec les risques listés dans la stratégie de test, la plupart de ceux qu'on avait classés en priorité haute se sont confirmés.

**Risque financier.** C'est le plus grave. Un client qui paie avec PayPal est débité deux fois (BUG-01). Une commande en espèces peut être marquée comme payée alors que le client ne donne que 1 € pour 24,29 € (BUG-03). Enfin la promotion ATLAS15 ne donne pas la réduction annoncée (BUG-08), ce qui peut poser un problème de publicité trompeuse en plus de mécontenter les clients.

**Risque sur le stock.** Les plats marqués "HORS STOCK" peuvent quand même être commandés et le stock devient négatif (BUG-02). Quand une commande est annulée le stock n'est pas rendu non plus (BUG-07). Au bout de quelques jours le stock affiché ne voudrait plus rien dire et les restaurants recevraient des commandes qu'ils ne peuvent pas préparer.

**Risque sur les commandes impossibles à livrer.** Un restaurant fermé reçoit des commandes (BUG-04), le minimum de commande n'est jamais vérifié (BUG-05) et un nouveau client peut commander sans avoir donné d'adresse (BUG-06). Dans les trois cas la commande est validée et payée alors qu'elle ne pourra pas être honorée, ce qui veut dire des remboursements à faire et des clients déçus.

**Risque d'écart entre le site et l'API.** Les problèmes de stock et de minimum existent aussi dans l'API. Une application mobile ou un partenaire qui passerait par l'API aurait donc les mêmes défauts.

Ce qui fonctionne bien et n'est pas un risque aujourd'hui: la connexion, le refus d'un mauvais mot de passe, le blocage de l'espace admin pour un client, le refus d'un code promo expiré sur le site et le panier limité à un seul restaurant.

## Recommandation

### No-Go

Dans la stratégie de test on avait fixé des critères pour autoriser la mise en production. Ils ne sont pas remplis:

| Critère | Résultat |
|---|---|
| Tous les cas prévus ont été exécutés | Oui, 24 sur 24 |
| Aucun bug critique ouvert | Non, 2 bugs critiques |
| Les parcours P1 à P4 passent tous | Non, seul P1 (commande simple par carte) passe. P2 (paiement), P3 (stock) et P4 (restaurant fermé et minimum) échouent |
| Les corrections ont été retestées | Non, aucune correction n'est encore disponible |

On recommande donc de ne pas mettre QuickOrder en production dans son état actuel.

### Conditions pour passer en Go

1. Corriger les 5 bugs P1 (BUG-01 à BUG-05).
2. Rejouer les cas qui étaient en Fail pour vérifier que les corrections marchent: TC-R05, TC-L03, TC-E02, TC-R04, TC-E03 et TC-R01.
3. Rejouer ensuite les 24 cas pour vérifier que les corrections n'ont rien cassé ailleurs.
4. Corriger les bugs P2 (BUG-06 à BUG-08) rapidement après, ou au moins avant d'ouvrir le service à un grand nombre de clients.