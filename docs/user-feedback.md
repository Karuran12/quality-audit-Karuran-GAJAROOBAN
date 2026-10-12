# Tests utilisateurs

## Objectif

Les tests de la campagne ont été faits par nous, qui connaissons déjà l'application et ses règles. On voulait aussi voir comment QuickOrder est vécu par des personnes qui le découvrent, sans explication, pour repérer ce qui les bloque ou ce qui les surprend. Ce sont souvent des problèmes qu'un testeur ne voit plus parce qu'il sait déjà où cliquer.

## Comment se sont passées les séances

- Nombre de participants: 2
- Durée: entre 10 et 60 minutes par personne
- Matériel: un ordinateur avec QuickOrder lancé en local sur Safari
- La base a été remise à zéro avec `python seed.py` avant chaque participant pour que tout le monde parte des mêmes données
- On lisait le scénario au participant sans lui montrer comment faire
- On lui demandait de dire à voix haute ce qu'il cherchait et ce qu'il pensait pendant qu'il avançait
- Un membre de l'équipe notait les phrases exactes du participant, les hésitations et les erreurs
- On n'aidait le participant que s'il était complètement bloqué et on le notait dans ce cas

Pour chaque scénario on a noté si la personne avait réussi seule, réussi avec de l'aide ou abandonné, ainsi que le temps passé.

## Les scénarios

### Scénario 1: commander un repas végétarien pas cher

Consigne lue au participant:
"Tu as un compte (bob@example.com / Password123!). Tu veux un repas végétarien avec des frais de livraison de 2,50 € maximum. Trouve un restaurant et passe la commande en payant par carte."

Ce qu'on observe:

- est-ce qu'il utilise le filtre "Livraison max" et est-ce que les résultats lui paraissent logiques
- est-ce qu'il essaie d'ajouter un plat marqué "HORS STOCK"
- est-ce qu'il remarque le minimum de commande du restaurant
- est-ce qu'il comprend le statut "confirmed" affiché à la fin

### Scénario 2: profiter d'une promotion

Consigne lue au participant:
"Le restaurant Couscous Atlas propose une promotion. Commande 2 Couscous Royal en profitant de cette promotion et dis-moi si tu penses avoir payé le bon prix."

Ce qu'on observe:

- est-ce qu'il trouve où saisir le code promo
- est-ce qu'il vérifie le montant de la réduction
- est-ce qu'il se rend compte que la réduction fait 10 % au lieu des 15 % annoncés et comment il réagit

### Scénario 3: nouveau client qui paie en espèces puis annule

Consigne lue au participant:
"Crée ton propre compte. Commande au moins 25 € chez Pizza Roma en payant en espèces avec un billet de 50 €. Ensuite tu changes d'avis: annule ta commande."

Ce qu'on observe:

- est-ce qu'il pense à renseigner une adresse avant de commander
- est-ce qu'il comprend le champ "Montant remis" et la monnaie calculée
- est-ce qu'il trouve facilement le bouton pour annuler
- est-ce qu'il a l'impression d'être remboursé après l'annulation

## Profil des participants

| Participant | Prénom | Âge | Métier ou études | Utilise des applis de livraison |
|---|---|---|---|---|
| U1 | Maxime | 21 | Étudiant | Oui, très souvent |
| U2 | Rodney | 72 | Retraité | Pas souvent |

## Résultats par scénario

| Participant | Scénario | Réussite (seul / avec aide / abandon) | Temps | Difficulté (1 = très facile, 5 = très difficile) |
|---|---|---|---|---|
| U1 | S1 | seul | 2min | 1 |
| U1 | S2 | seul | 3min | 2 |
| U1 | S3 | seul | 7min | 3 |
| U2 | S1 | avec aide | 9min | 4 |
| U2 | S2 | seul | 12min | 5 |
| U2 | S3 | seul | 29min | 5 |

## Problèmes relevés

Fréquence: nombre de participants qui ont rencontré le problème (par exemple 2/2).
Impact: Bloquant si la personne n'a pas pu finir, Majeur si elle a fait une erreur ou a été très gênée, Mineur si elle a juste hésité.
Priorité: Haute si le problème est bloquant ou touche tous les participants, Moyenne s'il est majeur pour une seule personne, Basse sinon.

### UX-01 – Le filtre "Livraison max" affiche les mauvais restaurants

| | |
|---|---|
| Profil | U1 étudiant de 21 ans qui commande souvent / U2 72 ans retraité qui commande rarement |
| Contexte | Scénario 1, page d'accueil. Le participant cherche un restaurant avec des frais de livraison de 2,50 € maximum et utilise le filtre "Livraison max ≤ 2,50 €". |
| Problème | Le filtre affiche les restaurants dont la livraison coûte plus de 2,50 € (Burger Factory 2,99 €, Tokyo Bowl 3,49 €) et cache ceux qui correspondent. Les deux participants ont arrêté d'utiliser le filtre et ont cherché à la main. Rodney n'arrivait pas à trouver de restaurant à cause du filtre, on lui a conseillé de l'enlever et de regarder les frais de livraison sur chaque carte. |
| Fréquence | 2/2 |
| Impact | Majeur: les deux ont fini le scénario mais Rodney a eu besoin d'aide et Maxime a perdu du temps |
| Verbatim | U1: "pourquoi il me met des restos plus chers que ce que j'ai demandé ?" / U2: "le filtre il marche pas, je ne comprend pas" |
| Hypothèse d'amélioration | Corriger le filtre pour qu'il garde les restaurants dont la livraison est inférieure ou égale au montant choisi. Afficher aussi le nombre de résultats pour que l'utilisateur voie que le filtre a bien été appliqué. |
| Priorité | Haute (touche tous les participants) |

### UX-02 – La réduction de la promo ATLAS15 ne correspond pas à ce qui est annoncé

| | |
|---|---|
| Profil | U1 étudiant de 21 ans qui commande souvent / U2 72 ans retraité qui commande rarement |
| Contexte | Scénario 2, page de paiement. Le participant applique le code ATLAS15 annoncé à -15 % sur la page d'accueil pour 2 Couscous Royal (35,80 €). |
| Problème | La réduction affichée est de 3,58 € soit 10 % au lieu de 15 %. U1 ne l'a pas remarqué et a validé en pensant avoir eu la bonne promo. U2 a fait le calcul de tête, a vu que ça ne collait pas et a hésité à continuer. |
| Fréquence | 2/2 (un participant ne l'a pas vu, l'autre s'en est rendu compte) |
| Impact | Majeur: un client paie plus que prévu sans le savoir, l'autre perd confiance dans le site |
| Verbatim | U1: "j'ai ma promo" / U2: "attends 15% de 35 ça fait plus que 3 euros non ?" |
| Hypothèse d'amélioration | Appliquer réellement 15 % comme annoncé. Afficher aussi le pourcentage à côté de la réduction ("ATLAS15 -15 %: -5,37 €") pour que le client puisse vérifier facilement. |
| Priorité | Haute |
| Lien avec la campagne | Confirme BUG-08 |

### UX-03 – Le statut de la commande est écrit en anglais

| | |
|---|---|
| Profil | U2 72 ans retraité qui commande rarement et n'est pas à l'aise avec l'anglais |
| Contexte | Scénario 3, page de détail de la commande juste après le paiement. |
| Problème | Le statut s'affiche "confirmed" alors que tout le reste du site est en français. Le participant a hésité quelques secondes avant de comprendre que la commande était bien validée. Les autres statuts (preparing, out_for_delivery, delivered, cancelled) posent le même problème. |
| Fréquence | 1/2 |
| Impact | Mineur: le participant a hésité mais a continué sans aide |
| Verbatim | U2: "confirmed ça veut dire que c'est validé ?" |
| Hypothèse d'amélioration | Traduire les statuts en français: "Confirmée", "En préparation", "En livraison", "Livrée", "Annulée". On peut aussi ajouter une couleur ou une icône par statut pour qu'on comprenne sans lire. |
| Priorité | Basse |
| Lien avec la campagne | Pas de bug lié, c'est un problème d'ergonomie à ajouter au plan d'amélioration |

## Synthèse

- Taux de réussite par scénario:
  - Scénario 1: 2/2 ont fini, dont 1 avec de l'aide (Rodney)
  - Scénario 2: 2/2 ont fini seuls
  - Scénario 3: 2/2 ont fini seuls
- Durée: 12 minutes pour Maxime et 50 minutes pour Rodney. Le scénario 3 est le plus long pour les deux (7 min et 29 min).
- Note moyenne de difficulté (1 = très facile, 5 = très difficile): 2 pour Maxime, 4,7 pour Rodney, 3,3 en moyenne. Rodney a trouvé les scénarios beaucoup plus difficiles, ce qui va avec le temps qu'il a passé.
- Les problèmes les plus importants:
  - UX-01: le filtre "Livraison max" affiche les restaurants les plus chers au lieu des moins chers. Les deux participants ont arrêté de l'utiliser.
  - UX-02: la promo ATLAS15 donne 10 % au lieu des 15 % annoncés. Un participant ne l'a pas remarqué et a payé plus que prévu.
- Problèmes qui confirment des bugs de la campagne: UX-02 confirme BUG-08
- Problèmes nouveaux qu'on n'avait pas vus pendant nos tests: UX-01 (le filtre "Livraison max" qui affiche les mauvais restaurants) et UX-03 (les statuts de commande en anglais). Ce sont des problèmes d'ergonomie qui ne faisaient pas partie de nos 24 cas de test.