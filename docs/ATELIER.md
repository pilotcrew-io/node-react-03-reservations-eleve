# Tutoriel 03 — Fullstack · Réserver un stock sans doublon

Le temps indiqué suppose [les prérequis](PREREQUIS.md) terminés. Le [contrat commun](PREREQUIS.md) est la référence pour les champs, routes et limites. Consigner les résultats au fil de l’eau dans [le journal](../travail/REPONSES.md).

## 1. Décrire une intention durable pendant la tentative — 25 min

À partir du panier importé, écrivez l’intention envoyée au serveur : un tableau de couples productId/quantity. Retirez mentalement noms, prix, stock et total ; ils ne sont pas nécessaires pour demander une réservation. Faites expliquer pourquoi la confirmation possède davantage de champs que la requête. Le serveur complète ces champs depuis son propre catalogue.

Dessinez trois cas : succès reçu, refus explicite et réponse perdue. Le troisième n’est pas un refus métier. Une requête peut avoir été appliquée avant la rupture du réseau. Formulez donc le résultat attendu d’un réessai : même réservation, même montant, même retrait de stock. Définissez aussi la limite du modèle : une réservation CONFIRMED dure tant que le processus fonctionne, sans expiration ni annulation.

Écrivez cinq invariants dans le journal : aucun stock négatif, tous les produits contrôlés avant écriture, aucun prix fourni par le client, un seul résultat par clé/intention et aucune écriture lors d’un manque de stock. Identifiez pour chacun la preuve qui pourrait révéler une régression. Réserver un stock fictif ne signifie jamais payer ni vendre.

## 2. Construire la réservation côté serveur — 60 min

Ajoutez une opération métier recevant une intention et une clé validées. À la frontière HTTP, limitez le corps à 16 KiB et exigez un type JSON pris en charge. Le corps comporte uniquement items ; chaque ligne uniquement productId et quantity. Refusez lignes dupliquées, tableau vide, plus de dix références, quantités hors 1 à 5, décimales et chaînes numériques. Une propriété totalCents ajoutée n’est pas ignorée comme une instruction possible : le contrat la refuse.

Relisez chaque produit dans le catalogue serveur. Vérifiez son existence puis sa quantité disponible, préparez les montants de lignes et le total en entiers sûrs. Les prix viennent du produit courant. Aucune diminution de stock ne doit se produire pendant cette première phase. Si une ligne manque, retournez l’erreur appropriée pour l’ensemble de la demande.

Après validation complète, créez une réservation avec ID, statut CONFIRMED, lignes triées par productId, totalCents, devise EUR et createdAt en ISO UTC. Enregistrez-la et diminuez chaque stock une seule fois. Ajoutez GET /api/reservations/{id}, qui relit sans aucun effet d’écriture. La réponse initiale est 201 avec Location ; une lecture est 200. Retournez des copies afin que le code appelant ne puisse pas modifier les registres internes.

```bash
curl -i -X POST 'http://127.0.0.1:8080/api/reservations'   -H 'Content-Type: application/json'   -H 'Idempotency-Key: nr03-demo-0001'   --data '{"items":[{"productId":"retail-001","quantity":2}]}'
```

Conservez l’ID réellement reçu. Comparez le prix confirmé avec GET /api/products/retail-001 et calculez le résultat attendu depuis cette lecture ; ne supposez pas qu’un montant d’exemple vaut pour toutes les fixtures.

## 3. Rendre l’opération indivisible et idempotente — 55 min

Normalisez l’intention en triant les couples produit/quantité. Une permutation des lignes ne change pas la demande. Évitez de comparer simplement les chaînes JSON brutes, dont l’ordre peut varier. Une clé valide contient de huit à quatre-vingts caractères alphanumériques ASCII ou tirets. Une clé n’identifie pas un utilisateur et n’accorde aucun droit.

Consultez le registre d’idempotence avant d’appliquer une nouvelle diminution. Si la clé existe et l’intention est identique, rendez exactement la réservation enregistrée avec 201, même si le stock restant serait désormais insuffisant pour une nouvelle opération. Si l’intention diffère, rendez 409 IDEMPOTENCY_CONFLICT. Un manque de stock ne crée ni réservation ni entrée de clé ; une tentative corrigée après ce refus pourra réutiliser cette clé.

Placez validation métier, décision, vérification et écritures dans une section synchrone du processus Node. N’ajoutez aucun await au milieu. L’absence d’interruption dans cette section explique la garantie locale ; « JavaScript est asynchrone » n’est pas une preuve suffisante. Décrivez ce qui se casserait avec deux processus indépendants et ce qu’une base transactionnelle devrait apporter dans une évolution réelle.

Bornez les nouvelles réservations à cinq cents pour éviter une croissance mémoire sans limite dans ce laboratoire. Le plafond répond 503 CAPACITY_REACHED, mais une ancienne clé/intention doit encore pouvoir être relue par rejeu. Ne supprimez pas arbitrairement les clés les plus anciennes pour rester sous le plafond : cela autoriserait un double traitement d’une ancienne intention.

## 4. Brancher la confirmation React et la reprise — 55 min

Ajoutez une action « Réserver le stock fictif » dans le panier. Au clic, construisez une intention normalisée et figée, générez une clé, puis conservez **clé et intention avant le POST** sous `pilotcrew.retail.reservation.v1` dans sessionStorage. Le nom exact de la structure interne vous appartient, mais sa version et ses invariants doivent être validés à la restauration. Ne transmettez aucun montant local.

Désactivez les modifications qui changeraient l’intention pendant la tentative, ou imposez une séparation explicite entre tentative incertaine et futur panier. L’utilisateur doit comprendre ce qui est en cours. Un double clic rapide ne doit pas générer deux clés. Si la réponse arrive, validez son contrat puis affichez ID, lignes et montant de la réservation serveur. Le total estimé ne doit pas être réutilisé comme confirmation.

Si la réponse est perdue, conservez le contexte et proposez de reprendre avec **la même clé et exactement les mêmes quantités**. Une nouvelle clé transformera le réessai en nouvelle opération, ce qui est le bug à éviter. Après un refus métier explicite, le panier peut être corrigé ; expliquez la différence avec une tentative dont le résultat demeure inconnu. Une absence de réponse ne justifie pas de vider le panier.

Si l’écriture de la tentative dans sessionStorage échoue, signalez l’impossibilité de garantir la reprise après fermeture/rechargement. Gardez au minimum l’intention en mémoire et ne promettez pas une récupération qui ne serait pas possible. Vous pouvez choisir de bloquer l’envoi dans ce cas, à condition de fournir une action utile et de tester ce choix. L’atelier 06 approfondira la relecture par ID après un succès.

## 5. Prouver les cas qui retireraient trop de stock — 50 min

Sur un serveur neuf ou une instance métier isolée, relevez le stock d’un produit. Envoyez la requête de démonstration puis rejouez exactement la même. L’ID, createdAt, lignes et total doivent rester identiques ; le stock ne baisse qu’une fois. Modifiez une quantité en gardant la clé et prouvez le conflit sans nouvelle écriture. Rejouez une demande de deux produits dans l’ordre inverse : elle doit rester identique.

Écrivez un test de manque de stock portant sur deux lignes. Préparez une première ligne disponible et une seconde insuffisante, puis vérifiez que la première n’a pas été retirée. Pour éviter vingt appels manuels, utilisez une instance de domaine de test avec une fixture de stock contrôlée. Un autre test envoie deux demandes concurrentes sur le vrai serveur local ; il vérifie la conservation du stock et les réponses, sans généraliser le résultat à plusieurs processus.

Dans React, simulez une réponse perdue après traitement, puis rechargez et reprenez. Comparez les deux corps et les deux clés capturés. Le test doit vraiment transmettre la première requête au serveur si vous présentez cela comme une preuve de bout en bout ; un mock qui refuse avant tout traitement ne couvre que l’interface. Notez précisément le niveau de preuve.

```bash
npm test
npm run build
```

Exécutez dans les deux projets. Si des scripts de smoke existent déjà dans votre travail, utilisez-les en complément ; leur code doit provenir de votre réalisation ou être identifié comme aide du formateur.

## 6. Rendre une trace corrélée — 25 min

Présentez une seule histoire complète : panier choisi, clé créée, première requête, résultat incertain, reprise, réservation confirmée et stock final. Reliez chaque étape à une observation. Ne montrez aucune donnée de personne réelle et ne qualifiez jamais la confirmation de paiement.

Faites expliquer à un binôme pourquoi GET n’écrit rien, pourquoi trier les lignes change la comparaison, et pourquoi un redémarrage détruit la garantie mémoire. Remettez les tests, les traces et une courte décision sur le comportement d’un stockage navigateur indisponible. Une version qui avoue cette limite est plus facile à exploiter qu’une promesse de reprise sans état conservé.
