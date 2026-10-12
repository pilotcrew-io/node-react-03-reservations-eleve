# Fullstack · Réserver un stock sans doublon

**Atelier 03/06 · 4 h 30 · nr-03-reservations**

Deux utilisateurs peuvent regarder le même stock. Une connexion peut perdre la réponse alors que le serveur a réservé. Le magasin doit éviter de retirer deux fois le stock pour la même intention, et ne doit jamais confirmer une ligne partielle si une autre échoue.

Ce dépôt contient les instructions, les mémos et les espaces de travail. **Le code est à construire ; aucune solution terminée n’est fournie dans le dépôt élève.** Les acquisitions précédentes sont reprises avant la séance selon [Day 0](DAY0.md).

## À la fin de la séance

- Valider une intention sans accepter un montant du navigateur.
- Vérifier tous les stocks avant toute écriture.
- Associer une clé à une intention normalisée et vérifier les rejeux.
- Construire une reprise frontend qui conserve clé et contenu.

## Déroulé de 270 minutes

| Séquence | Minutes |
| --- | --- |
| Intention et invariants | 25 |
| Validation et réservation serveur | 60 |
| Atomicité et idempotence | 55 |
| Confirmation et reprise React | 55 |
| Tests des scénarios critiques | 50 |
| Revue et remise | 25 |

Répartition cible de cette séance : **120 min frontend / 150 min backend**, tests et revue de chaque couche inclus. Sur le cursus : **18 h frontend et 9 h backend**. Cette dominante React est un choix pédagogique, pas un ratio chiffré attribué à l’annonce.

## Documents de travail

- [Prérequis et reprise](docs/PREREQUIS.md)
- [Tutoriel guidé](docs/ATELIER.md)
- [Mémos de compréhension](docs/MEMOS.md)
- [Dépannage](docs/DEPANNAGE.md)
- [Grille et preuves](docs/EVALUATION.md)
- [Backlog](travail/BACKLOG.md)
- [Journal à compléter](travail/REPONSES.md)
- [Sources officielles](docs/SOURCES.md)
- [Travailler et remettre](CONTRIBUTING.md)
