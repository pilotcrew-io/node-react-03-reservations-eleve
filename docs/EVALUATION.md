# Évaluation et preuves

Le barème porte sur ce qui est **démontré et expliqué**. Une suite non exécutée ou une capture absente n’est pas un succès implicite. Les choix internes peuvent différer du corrigé si le contrat et les critères sont respectés.

| Critère | Points / 20 | Preuve attendue |
| --- | --- | --- |
| Intention et validation | 3 | Corps minimal et cas invalides refusés sans montant client. |
| Montants et stock atomique | 4 | Toutes les lignes vérifiées, refus sans écriture prouvé. |
| Idempotence et normalisation | 4 | Rejeu, permutation et conflit vérifiés avec stock final. |
| Confirmation et reprise React | 3 | Clé/contenu avant envoi, montant serveur et incertitude explicite. |
| Tests critiques | 3 | Scénarios isolés et portée de la preuve réseau décrite. |
| Trace et limites | 3 | Une histoire corrélée, processus unique et absence de paiement expliqués. |

## Questions d’explication

1. Pourquoi la clé seule ne suffit-elle pas à comparer deux demandes ?
2. Que sait-on réellement après une réponse perdue ?
3. Quand faut-il diminuer les stocks dans une demande à plusieurs lignes ?
4. Pourquoi le rejeu est-il autorisé même si le stock restant est devenu faible ?
5. Quelle garantie disparaît au redémarrage ou avec deux processus ?

## Format de preuve

Pour chaque contrôle : commit, environnement, commande ou action, résultat attendu, résultat observé, interprétation et lien vers le rapport/capture. Séparer backend, frontend, navigateur et éventuelle CI distante. Pour une mesure : conserver aussi protocole et valeurs brutes.

Un invariant critique non respecté reste non acquis même si les points de présentation sont obtenus. L’évaluation est formative ; elle n’atteste ni une mise en production, ni un niveau senior, ni la sécurité d’un service public.
