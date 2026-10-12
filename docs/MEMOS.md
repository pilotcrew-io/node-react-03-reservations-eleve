# Mémos — Fullstack · Réserver un stock sans doublon

À relire pour expliquer les décisions ; les mémos ne remplacent pas les preuves de [l’évaluation](EVALUATION.md).

## Idempotence

L’idempotence répond à une question concrète : rejouer cette même intention doit-il appliquer de nouveau ses effets ? La clé permet de retrouver un résultat enregistré ; le contenu normalisé permet de vérifier que la clé n’a pas été réutilisée pour autre chose. Désactiver un bouton réduit les clics accidentels mais ne règle ni réponse perdue, ni rechargement, ni requête manuelle.

## Normalisation

Deux tableaux qui contiennent les mêmes produits et quantités dans un autre ordre expriment la même intention. Normaliser évite de confondre forme de transport et signification métier. L’API refuse les identifiants dupliqués avant cette étape ; elle ne devine pas si deux lignes doivent être fusionnées. Le journal doit rendre explicite cette règle plutôt que parler vaguement de « même JSON ».

## Atomicité locale

Vérifier puis écrire sans suspension forme ici une opération indivisible dans la boucle d’événements d’un seul processus. Cela n’implique aucune durabilité. Deux processus posséderaient deux registres et pourraient accepter deux opérations contradictoires. Une future version distribuée aurait besoin de stockage partagé, contraintes et transactions adaptées. Le test local permet de vérifier le périmètre annoncé, pas d’étendre sa portée.

## Calcul et refus sans écriture

Les montants viennent du catalogue serveur. Les lignes sont préparées, vérifiées, puis appliquées ensemble. Si le stock d’une ligne manque, une réservation partielle contredirait le contrat. Le contrôle doit donc précéder toute diminution. Refuser un champ prix inattendu permet également de détecter une mauvaise compréhension du client au lieu de laisser cette erreur invisible.

## Incertitude réseau

Une réponse perdue ne révèle pas si le serveur a écrit. La clé et le contenu conservés avant l’envoi permettent de poser à nouveau la même question sans créer une nouvelle intention. Une réponse métier 409, au contraire, indique un résultat explicite dans ce contrat. Les actions de reprise ne doivent pas traiter ces deux situations de manière indistincte.

## Stockage et identité

La clé d’idempotence n’est ni un secret, ni une preuve de possession du stock, ni une authentification. Le laboratoire n’a aucun compte utilisateur et n’est pas conçu pour une ouverture publique. sessionStorage aide à reprendre dans une session navigateur ; le registre serveur reste nécessaire. Perdre l’un ou l’autre limite la reprise et doit être expliqué à la démonstration.
