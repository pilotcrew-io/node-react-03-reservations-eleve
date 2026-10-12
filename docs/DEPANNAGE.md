# Dépannage raisonné

Reproduire le symptôme sur un état connu, lire la réponse ou la trace, formuler une hypothèse puis effectuer un contrôle ciblé. Ne pas effacer tout le projet pour faire disparaître l’erreur.

| Symptôme | Hypothèse à vérifier | Contrôle et correction |
| --- | --- | --- |
| Stock retiré deux fois | Nouvelle clé au réessai ou recherche trop tardive du registre | Comparer clé/contenu puis retrouver le résultat avant nouvelle écriture. |
| Conflit pour lignes permutées | Comparaison de JSON brut | Normaliser les couples après validation des doublons. |
| Première ligne retirée malgré un refus | Écritures dans la boucle de contrôle | Préparer et valider l’ensemble avant application. |
| Prix client accepté | Corps trop permissif | Refuser les champs inconnus et relire le produit serveur. |
| Clé consommée lors de stock insuffisant | Enregistrement avant décision finale | N’enregistrer que la création effectivement acceptée. |
| Panier perdu lors d’une réponse perdue | Erreur réseau assimilée à succès ou refus | Conserver intention et clé, exposer une reprise explicite. |
| Promesse de transaction entre serveurs | Garantie du processus unique généralisée | Documenter état mémoire et besoin futur de stockage transactionnel. |

Si le blocage concerne une installation manquante du Day 0, le noter séparément du résultat métier. Demander une aide ciblée après avoir inscrit la commande, le statut et le message utile dans le journal.
