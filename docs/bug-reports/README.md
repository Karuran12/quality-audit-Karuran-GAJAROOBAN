# Bug reports – QuickOrder

8 bugs trouvés pendant la campagne de test. Ils sont tous reproductibles à partir d'une base remise à zéro (`python seed.py`). Les captures sont dans `docs/evidence/`.

## Échelles utilisées

**Sévérité**: la gravité du problème pour l'utilisateur ou l'entreprise.

- **Critique**: perte d'argent ou de marchandise, sans moyen de l'éviter
- **Majeur**: une règle importante n'est pas respectée
- **Mineur**: gêne sans perte directe

**Priorité**: dans quel ordre il faut le corriger.

- **P1**: à corriger avant la mise en production
- **P2**: à corriger juste après

## Liste des bugs

| ID | Titre | Sévérité | Priorité | Cas de test |
|----|-------|----------|----------|-------------|
| [BUG-01](BUG-01.md) | Un paiement PayPal crée deux commandes | Critique | P1 | TC-R05 |
| [BUG-02](BUG-02.md) | Le stock n'est pas vérifié à la commande | Critique | P1 | TC-L03, TC-E02 |
| [BUG-03](BUG-03.md) | Paiement en espèces accepté avec un montant insuffisant | Majeur | P1 | TC-R04 |
| [BUG-04](BUG-04.md) | Un restaurant fermé accepte les commandes | Majeur | P1 | TC-E03 |
| [BUG-05](BUG-05.md) | Le minimum de commande n'est pas appliqué | Majeur | P1 | TC-R01 |
| [BUG-06](BUG-06.md) | Un nouveau client peut commander sans adresse | Majeur | P2 | TC-E01 |
| [BUG-07](BUG-07.md) | Annuler une commande ne remet pas le stock | Majeur | P2 | TC-R02 |
| [BUG-08](BUG-08.md) | La promo ATLAS15 applique 10 % au lieu de 15 % | Majeur | P2 | TC-R06 |