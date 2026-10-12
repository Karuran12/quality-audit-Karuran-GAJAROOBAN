# Plan d'amélioration

Ce plan reprend les problèmes trouvés pendant la campagne de test (bugs BUG-01 à BUG-08) et pendant les tests utilisateurs (UX-01 à UX-03). Pour chaque amélioration on explique pourquoi elle est utile, ce qu'il faut changer et comment on vérifiera que ça marche.

## Comment on a priorisé

- **P1**: à faire avant la mise en production, parce que le problème fait perdre de l'argent au client ou à l'entreprise
- **P2**: à faire juste après, parce que le problème gêne les clients ou fait perdre confiance dans le site
- **P3**: à planifier, parce que le problème est gênant mais n'empêche pas d'utiliser le site

On a aussi donné un effort estimé pour chaque amélioration: faible (quelques heures), moyen (un ou deux jours) ou important (plus de deux jours).

## Plan

| N° | Amélioration | Problèmes réglés | Priorité | Effort |
|---|---|---|---|---|
| A1 | Rendre le paiement fiable | BUG-01, BUG-03 | P1 | Faible |
| A2 | Contrôler le stock à chaque étape | BUG-02, BUG-07 | P1 | Moyen |
| A3 | Vérifier les règles de commande avant de valider | BUG-04, BUG-05, BUG-06 | P1 | Moyen |
| A4 | Appliquer les promotions telles qu'elles sont annoncées | BUG-08, UX-02 | P2 | Faible |
| A5 | Corriger le filtre "Livraison max" | UX-01 | P2 | Faible |
| A6 | Mettre en place des tests automatiques | Tous | P2 | Moyen |
| A7 | Rendre le suivi de commande plus clair | UX-03, BUG-07 | P3 | Faible |

## Détail des améliorations

### A1. Rendre le paiement fiable (P1)

**Pourquoi**: c'est le problème le plus grave qu'on a trouvé. Avec PayPal le client est débité deux fois pour un seul panier et avec les espèces une commande de 24,29 € est marquée comme payée alors que le client n'a donné que 1 €. Dans les deux cas soit le client perd de l'argent soit c'est l'entreprise qui en perd.

**Ce qu'il faut faire**:

- faire en sorte qu'un paiement crée une seule commande, quel que soit le moyen de paiement choisi
- refuser un paiement en espèces quand le montant remis est plus petit que le total et afficher un message clair au client
- laisser une commande en espèces au statut "en attente de paiement" tant que le livreur n'a pas encaissé l'argent

**Comment on vérifie**: rejouer TC-R05 (une seule commande en plus après un paiement PayPal) et TC-R04 (commande refusée avec 1 €).

### A2. Contrôler le stock à chaque étape (P1)

**Pourquoi**: aujourd'hui un plat marqué "HORS STOCK" peut être commandé et le stock devient négatif. En plus quand une commande est annulée les plats ne sont pas rendus au stock. Au bout de quelques jours le stock affiché ne correspondrait plus du tout à la réalité et les restaurants recevraient des commandes qu'ils ne peuvent pas préparer.

**Ce qu'il faut faire**:

- griser le bouton "Ajouter" sur un plat dont le stock est à 0
- refuser l'ajout au panier quand on demande plus que le stock disponible
- vérifier encore une fois le stock au moment du paiement, au cas où un autre client aurait commandé entre temps
- remettre les plats dans le stock quand une commande est annulée
- appliquer ces règles aussi dans l'API

**Comment on vérifie**: rejouer TC-E02, TC-L03 et TC-R02, puis vérifier qu'aucun stock n'est négatif à la fin de la campagne.

### A3. Vérifier les règles de commande avant de valider (P1)

**Pourquoi**: trois règles affichées sur le site ne sont jamais vérifiées. Un restaurant fermé reçoit des commandes, le minimum de commande est ignoré et un nouveau client peut commander sans adresse. Ces commandes sont payées alors qu'elles ne pourront pas être livrées, ce qui oblige à rembourser et donne une mauvaise image du service.

**Ce qu'il faut faire**:

- bloquer la commande quand le restaurant est fermé et l'indiquer dès la page du menu
- bloquer la validation tant que le minimum n'est pas atteint et afficher combien il manque (par exemple "Il vous manque 7,50 € pour commander")
- ne plus créer d'adresse vide à l'inscription et demander une adresse complète au moment du paiement si le client n'en a pas
- regrouper ces vérifications au même endroit dans le code pour que le site et l'API appliquent exactement les mêmes règles

Le dernier point est important parce que pendant la campagne on a vu que l'API avait les mêmes défauts que le site. Si on corrige seulement le site, quelqu'un pourrait toujours passer par l'API pour contourner les règles.

**Comment on vérifie**: rejouer TC-E03, TC-R01 et TC-E01 sur le site, puis faire les mêmes essais avec Swagger.

### A4. Appliquer les promotions telles qu'elles sont annoncées (P2)

**Pourquoi**: la page d'accueil annonce -15 % avec le code ATLAS15 mais le site n'applique que 10 %. Pendant les tests utilisateurs un participant ne l'a pas remarqué et a payé plus que prévu, l'autre a fait le calcul de tête et a perdu confiance. Annoncer une réduction et en appliquer une autre peut aussi être considéré comme de la publicité trompeuse.

**Ce qu'il faut faire**:

- corriger le code ATLAS15 pour qu'il donne bien 15 %
- générer le texte des promotions à partir des vrais réglages du code pour que le texte et la réduction ne puissent plus être différents
- afficher le pourcentage à côté du montant de la réduction sur la page de paiement (par exemple "ATLAS15 -15 %: -5,37 €")

**Comment on vérifie**: rejouer TC-R06, la réduction doit être de 5,37 € pour 2 Couscous Royal.

### A5. Corriger le filtre "Livraison max" (P2)

**Pourquoi**: ce problème n'était pas dans nos cas de test, on l'a découvert pendant les tests utilisateurs. Quand on choisit "Livraison max 2,50 €" le site affiche les restaurants les plus chers à livrer et cache ceux qui correspondent. Les deux participants ont arrêté d'utiliser le filtre et Rodney a eu besoin d'aide pour trouver un restaurant.

**Ce qu'il faut faire**:

- corriger le filtre pour qu'il garde les restaurants dont la livraison est inférieure ou égale au montant choisi
- afficher le nombre de restaurants trouvés pour que l'utilisateur voie que le filtre a bien été appliqué

**Comment on vérifie**: avec "Livraison max 2,50 €" seuls Green Kitchen (1,99 €) et Pizza Roma (2,49 €) doivent s'afficher. On peut l'ajouter comme nouveau cas de test limite.

### A6. Mettre en place des tests automatiques (P2)

**Pourquoi**: tous les bugs qu'on a trouvés concernent des règles simples à vérifier par un programme. Sans tests automatiques une correction peut en casser une autre sans que personne ne s'en rende compte, et il faudrait refaire les 24 cas à la main après chaque modification.

**Ce qu'il faut faire**:

- corriger le test automatique déjà présent dans le projet qui échoue aujourd'hui
- écrire un test automatique pour chaque bug corrigé, pour être sûr qu'il ne revienne pas
- lancer ces tests à chaque modification du code avant de la mettre en ligne

**Comment on vérifie**: tous les tests passent au vert après les corrections A1 à A5.

### A7. Rendre le suivi de commande plus clair (P3)

**Pourquoi**: le statut des commandes est affiché en anglais ("confirmed", "cancelled"…) alors que tout le reste du site est en français. Rodney a hésité avant de comprendre que sa commande était validée. Après une annulation la commande reste aussi marquée "paid" et rien n'indique au client qu'il sera remboursé.

**Ce qu'il faut faire**:

- traduire les statuts en français: "Confirmée", "En préparation", "En livraison", "Livrée", "Annulée"
- ajouter une couleur par statut pour qu'on le comprenne sans lire
- afficher "Remboursée" sur une commande annulée qui avait été payée
- cacher le bouton "Annuler la commande" quand la commande ne peut plus être annulée

**Comment on vérifie**: refaire le scénario 3 des tests utilisateurs avec une nouvelle personne et regarder si elle comprend le statut sans hésiter.

## Ordre de réalisation conseillé

1. A1, A2 et A3 avant toute mise en production. Ce sont elles qui règlent les 5 bugs P1.
2. A6 en même temps que A1 à A3, pour vérifier les corrections au fur et à mesure.
3. A4 et A5 juste après, car elles sont rapides à faire et touchent directement la confiance des clients.
4. A7 ensuite, quand le reste est stable.

Une fois A1 à A3 faites et retestées, on pourra refaire la campagne complète et revoir notre recommandation No-Go.