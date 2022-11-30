# Test Nombre de fois qu'une fonction peut être utilisée Limites

|ID          |
|------------|
|WSTG-BUSL-05|

## Sommaire

De nombreux problèmes que les applications résolvent nécessitent des limites quant au nombre de fois qu'une fonction peut être utilisée ou qu'une action peut être exécutée. Les applications doivent être « suffisamment intelligentes » pour ne pas permettre à l'utilisateur de dépasser sa limite d'utilisation de ces fonctions, car dans de nombreux cas, chaque fois que la fonction est utilisée, l'utilisateur peut obtenir un certain type d'avantage qui doit être pris en compte pour compenser correctement le propriétaire. . Par exemple : un site de commerce électronique peut autoriser les utilisateurs à n'appliquer une remise qu'une seule fois par transaction, ou certaines applications peuvent faire l'objet d'un abonnement et autoriser uniquement les utilisateurs à télécharger trois documents complets par mois.

Les vulnérabilités liées au test des limites de fonction sont spécifiques à l'application et des cas d'utilisation abusive doivent être créés qui s'efforcent d'exercer des parties de l'application/des fonctions/ou des actions plus que le nombre de fois autorisé.

Les attaquants peuvent être en mesure de contourner la logique métier et d'exécuter une fonction plus de fois que « autorisé » en exploitant l'application à des fins personnelles.

### Exemple

Supposons qu'un site de commerce électronique permette aux utilisateurs de profiter de l'une des nombreuses remises sur leur achat total, puis de procéder au paiement et à l'appel d'offres. Que se passe-t-il si l'attaquant revient à la page des remises après avoir pris et appliqué la seule remise "autorisée" ? Peuvent-ils profiter d'une autre réduction ? Peuvent-ils profiter plusieurs fois de la même remise ?

## Objectifs des tests

- Identifier les fonctions qui doivent fixer des limites aux moments où elles peuvent être appelées.
- Évaluer s'il y a une limite logique fixée sur les fonctions et si elle est correctement validée.

## Comment tester

- Examinez la documentation du projet et utilisez des tests exploratoires pour rechercher des fonctions ou des fonctionnalités dans l'application ou le système qui ne doivent pas être exécutées plus d'une seule fois ou un nombre spécifié de fois au cours du flux de travail de la logique métier.
- Pour chacune des fonctions et fonctionnalités trouvées qui ne doivent être exécutées qu'une seule fois ou un nombre de fois spécifié au cours du flux de travail de la logique métier, développez des cas d'abus/de mauvaise utilisation qui peuvent permettre à un utilisateur d'exécuter plus que le nombre de fois autorisé. Par exemple, un utilisateur peut-il parcourir plusieurs fois les pages en exécutant une fonction qui ne devrait s'exécuter qu'une seule fois ? ou un utilisateur peut-il charger et décharger des paniers d'achat permettant des remises supplémentaires.

## Cas de test associés

- [Test d'énumération de compte et de compte d'utilisateur devinable] (../03-Identity_Management_Testing/04-Testing_for_Account_Enumeration_and_Guessable_User_Account.md)
- [Test du mécanisme de verrouillage faible] (../04-Authentication_Testing/03-Testing_for_Weak_Lock_Out_Mechanism.md)

## Correction

L'application doit définir des contrôles stricts pour éviter les abus de limites. Ceci peut être réalisé en définissant un coupon pour qu'il ne soit plus valide au niveau de la base de données, afin de définir une limite de compteur par utilisateur au niveau du back-end ou de la base de données, car tous les utilisateurs doivent être identifiés via une session, selon ce qui convient le mieux aux besoins de l'entreprise. .

## Références

- [La logique métier d'InfoPath Forms Services a dépassé la limite maximale d'opérations] (http://mpwiki.viacode.com/default.aspx?g=posts&t=115678)
- [Le ??commerce de l'or a été temporairement interrompu sur le CME ce matin] (https://www.businessinsider.com/gold-halted-on-cme-for-stop-logic-event-2013-10)
