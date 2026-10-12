# Matrice des risques

Ce document reprend les risques listés dans la stratégie de test et les classe selon leur probabilité et leur impact. La dernière colonne indique ce que la campagne de test et les tests utilisateurs ont montré pour chaque risque.

## Échelles utilisées

**Probabilité** que le problème arrive:

- 1 faible: peu de chances que ça arrive
- 2 moyenne: ça peut arriver dans certains cas
- 3 forte: ça risque d'arriver souvent

**Impact** si le problème arrive:

- 1 gêne: l'utilisateur est embêté mais peut continuer
- 2 perte limitée: une fonction ne marche pas ou une petite perte d'argent
- 3 perte importante: perte d'argent, de données ou de confiance des clients

**Score** = Probabilité × Impact

- de 6 à 9: priorité haute
- de 3 à 4: priorité moyenne
- de 1 à 2: priorité basse

## Matrice détaillée

| ID | Risque | Probabilité | Impact | Score | Priorité | Cas de test liés | Résultat après la campagne |
|---|---|:---:|:---:|:---:|---|---|---|
| R1 | Le paiement crée plusieurs commandes ou accepte un montant insuffisant | 3 | 3 | 9 | Haute | TC-N05, TC-R04, TC-R05 | Confirmé: BUG-01 et BUG-03 |
| R2 | Le stock n'est pas vérifié au moment de la commande | 3 | 3 | 9 | Haute | TC-L03, TC-E02, TC-R02 | Confirmé: BUG-02 et BUG-07 |
| R3 | Les règles de commande ne sont pas appliquées (restaurant fermé, minimum, adresse) | 3 | 2 | 6 | Haute | TC-E01, TC-E03, TC-R01 | Confirmé: BUG-04, BUG-05 et BUG-06 |
| R4 | Un code promo est mal calculé, accepté alors qu'il a expiré ou différent de ce qui est affiché | 3 | 2 | 6 | Haute | TC-N06, TC-L04, TC-L05, TC-E05, TC-R06 | Confirmé en partie: BUG-08 et UX-02. Le calcul de WELCOME10, son seuil et son plafond sont corrects et le code expiré est bien refusé |
| R5 | L'API n'applique pas les mêmes règles que le site | 3 | 2 | 6 | Haute | TC-L01 à TC-L05 | Confirmé: le stock (BUG-02) et le minimum de commande (BUG-05) ne sont pas vérifiés non plus dans l'API |
| R6 | Faille de sécurité simple (accès admin, données d'un autre client) | 2 | 3 | 6 | Haute | TC-E04, TC-E06 | Non confirmé sur ce qu'on a testé: un client ne peut pas ouvrir l'espace admin et un mauvais mot de passe est refusé |
| R7 | Le statut d'une commande peut revenir en arrière ou l'annulation ne rend pas le stock | 2 | 2 | 4 | Moyenne | TC-R02 | Confirmé pour l'annulation: BUG-07. Les changements de statut par l'admin n'ont pas été testés |
| R8 | L'API renvoie des erreurs mal formées (erreur 500, page HTML au lieu de JSON) | 2 | 2 | 4 | Moyenne | TC-L01, TC-L02 | Non confirmé sur ce qu'on a testé: les quantités invalides sont refusées avec un code 400 et un message JSON clair |
| R11 | La connexion ou l'inscription ne fonctionne pas | 1 | 3 | 3 | Moyenne | TC-N01, TC-E04, TC-L06 | Non confirmé: connexion, refus du mauvais mot de passe et inscription fonctionnent |
| R9 | La recherche ou les filtres renvoient de mauvais résultats | 3 | 1 | 3 | Moyenne | Aucun cas, vu en test utilisateur | Confirmé par les tests utilisateurs: UX-01, le filtre "Livraison max" est inversé |
| R10 | Les avis peuvent être déposés par n'importe qui | 2 | 1 | 2 | Basse | Aucun | Non testé |
| R12 | Swagger ou Docker ne se lancent pas | 1 | 1 | 1 | Basse | Utilisé pendant toute la campagne | Swagger fonctionne. Docker n'a pas été testé car on a lancé l'application avec Python |

## Carte 

Chaque case contient les risques qui ont cette probabilité et cet impact. Plus on va en haut à droite, plus le risque est dangereux.

| Probabilité / Impact | 1 Gêne | 2 Perte limitée | 3 Perte importante |
|---|---|---|---|
| **3 Forte** | R9 | R3, R4, R5 | **R1, R2** |
| **2 Moyenne** | R10 | R7, R8 | R6 |
| **1 Faible** | R12 | | R11 |

## Ce qu'on retient

Les deux risques les plus forts (R1 paiement et R2 stock, score de 9) se sont tous les deux confirmés pendant la campagne. C'est sur eux que portent les deux bugs critiques et c'est la principale raison de notre recommandation No-Go.

Sur les 6 risques classés en priorité haute, 5 se sont confirmés au moins en partie. Seul R6 (sécurité) n'a pas posé de problème sur les cas qu'on a testés. Ça montre que notre priorisation était juste: les risques qu'on avait jugés les plus dangereux sont bien ceux où on a trouvé les bugs.

On note aussi que R9 (recherche et filtres) n'était pas couvert par nos cas de test et que c'est grâce aux tests utilisateurs qu'on a découvert le problème du filtre "Livraison max". Si on refaisait la campagne, on ajouterait un cas de test sur les filtres.

Enfin R10 (avis) et le changement de statut par l'admin dans R7 n'ont pas été testés. Ce sont des points à couvrir lors d'une prochaine campagne.