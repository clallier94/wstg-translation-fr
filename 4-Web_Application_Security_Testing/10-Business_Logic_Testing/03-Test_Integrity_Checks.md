# Vérifications de l'intégrité des tests

|ID          |
|------------|
|WSTG-BUSL-03|

## Sommaire

De nombreuses applications sont conçues pour afficher différents champs en fonction de l'utilisateur de la situation en laissant certaines entrées masquées. Cependant, dans de nombreux cas, il est possible de soumettre des valeurs de champs masqués au serveur à l'aide d'un proxy. Dans ces cas, les contrôles côté serveur doivent être suffisamment intelligents pour effectuer des modifications relationnelles ou côté serveur afin de garantir que les données appropriées sont autorisées sur le serveur en fonction de la logique métier spécifique à l'utilisateur et à l'application.

De plus, l'application ne doit pas dépendre de contrôles non modifiables, de menus déroulants ou de champs masqués pour le traitement de la logique métier car ces champs restent non modifiables uniquement dans le contexte des navigateurs. Les utilisateurs peuvent modifier leurs valeurs à l'aide d'outils d'édition de proxy et essayer de manipuler la logique métier. Si l'application expose des valeurs liées à des règles métier telles que la quantité, etc. en tant que champs non modifiables, elle doit en conserver une copie côté serveur et l'utiliser pour le traitement de la logique métier. Enfin, outre les données d'application/système, les systèmes de journalisation doivent être sécurisés pour empêcher la lecture, l'écriture et la mise à jour.

Les vulnérabilités de vérification de l'intégrité de la logique métier sont uniques en ce sens que ces cas d'utilisation abusive sont spécifiques à l'application et si les utilisateurs sont en mesure d'apporter des modifications, ils ne devraient pouvoir écrire ou mettre à jour/modifier des artefacts spécifiques qu'à des moments précis selon la logique des processus métier.

L'application doit être suffisamment intelligente pour vérifier les modifications relationnelles et ne pas permettre aux utilisateurs de soumettre directement au serveur des informations qui ne sont pas valides, fiables car elles proviennent de contrôles non modifiables ou que l'utilisateur n'est pas autorisé à soumettre via le frontal. De plus, les artefacts système tels que les journaux doivent être « protégés » contre la lecture, l'écriture et la suppression non autorisées.

### Exemple 1

Imaginez une application GUI d'application ASP.NET qui permet uniquement à l'utilisateur administrateur de modifier le mot de passe des autres utilisateurs du système. L'utilisateur admin verra les champs de nom d'utilisateur et de mot de passe pour entrer un nom d'utilisateur et un mot de passe tandis que les autres utilisateurs ne verront aucun champ. Cependant, si un utilisateur non administrateur soumet des informations dans le champ nom d'utilisateur et mot de passe via un proxy, il peut être en mesure de "tromper" le serveur en lui faisant croire que la demande provient d'un utilisateur administrateur et de changer le mot de passe des autres utilisateurs.

### Exemple 2

La plupart des applications Web ont des listes déroulantes permettant à l'utilisateur de sélectionner rapidement son état, son mois de naissance, etc. Supposons qu'une application de gestion de projet autorise les utilisateurs à se connecter et, en fonction de leurs privilèges, leur présente une liste déroulante des projets auxquels ils ont accès. à. Que se passe-t-il si un attaquant trouve le nom d'un autre projet auquel il ne devrait pas avoir accès et soumet les informations via un proxy. L'application donnera-t-elle accès au projet ? Ils ne devraient pas avoir accès même s'ils ont ignoré une vérification de logique métier d'autorisation.

### Exemple 3

Supposons que le système d'administration des véhicules à moteur exige qu'un employé vérifie initialement la documentation et les informations de chaque citoyen lorsqu'il délivre une pièce d'identité ou un permis de conduire. À ce stade, le processus métier a créé des données avec un haut niveau d'intégrité car l'intégrité des données soumises est vérifiée par l'application. Supposons maintenant que l'application soit déplacée vers Internet afin que les employés puissent se connecter pour un service complet ou que les citoyens puissent se connecter pour une application en libre-service réduite afin de mettre à jour certaines informations. À ce stade, un attaquant peut être en mesure d'utiliser un proxy d'interception pour ajouter ou mettre à jour des données auxquelles il ne devrait pas avoir accès et il pourrait détruire l'intégrité des données en déclarant que le citoyen n'était pas marié mais en fournissant des données pour le nom d'un conjoint. Ce type d'insertion ou de mise à jour de données non vérifiées détruit l'intégrité des données et aurait pu être empêché si la logique du processus métier avait été suivie.

### Exemple 4

De nombreux systèmes incluent la journalisation à des fins d'audit et de dépannage. Mais quelle est la qualité/la validité des informations contenues dans ces journaux ? Peuvent-ils être manipulés par des attaquants intentionnellement ou accidentellement dont l'intégrité est détruite ?

## Objectifs des tests

- Examinez la documentation du projet pour les composants du système qui déplacent, stockent ou gèrent les données.
- Déterminer quel type de données est logiquement acceptable par le composant et contre quels types le système doit se protéger.
- Déterminez qui devrait être autorisé à modifier ou à lire ces données dans chaque composant.
- Essayez d'insérer, de mettre à jour ou de supprimer des valeurs de données utilisées par chaque composant qui ne devraient pas être autorisées par le workflow de logique métier.

## Comment tester

### Méthode de test spécifique 1

- À l'aide d'un proxy, capturez le trafic HTTP à la recherche de champs cachés.
- Si un champ masqué est trouvé, comparez ces champs avec l'application GUI et commencez à interroger cette valeur via le proxy en soumettant différentes valeurs de données en essayant de contourner le processus métier et de manipuler des valeurs auxquelles vous n'étiez pas censé avoir accès.

### Méthode de test spécifique 2

- L'utilisation d'un proxy capture le trafic HTTP à la recherche d'un emplacement pour insérer des informations dans des zones de l'application qui ne sont pas modifiables.
- S'il est trouvé, comparez ces champs avec l'application GUI et commencez à interroger cette valeur via le proxy en soumettant différentes valeurs de données en essayant de contourner le processus métier et de manipuler des valeurs auxquelles vous n'étiez pas censé avoir accès.

### Méthode de test spécifique 3

- Répertoriez les composants de l'application ou du système qui pourraient être impactés, par exemple les journaux ou les bases de données.
- Pour chaque composant identifié, essayez de lire, modifier ou supprimer ses informations. Par exemple, les fichiers journaux doivent être identifiés et les testeurs doivent essayer de manipuler les données/informations collectées.

## Cas de test associés

Tous les cas de test [Input Validation](../07-Input_Validation_Testing/README.md).

## Correction

L'application doit suivre des contrôles d'accès stricts sur la façon dont les données et les artefacts peuvent être modifiés et lus, et via des canaux de confiance qui garantissent l'intégrité des données. Une journalisation appropriée doit être mise en place pour examiner et s'assurer qu'aucun accès ou modification non autorisé ne se produit.

## Outils

- Divers outils système/applicatifs tels que des éditeurs et des outils de manipulation de fichiers.
- [Proxy d'attaque Zed OWASP (ZAP)](https://www.zaproxy.org)
- [Burp Suite](https://portswigger.net/burp)

## Références

- [Mise en œuvre de l'intégrité référentielle et de la logique métier partagée dans une RDB](http://www.agiledata.org/essays/referentialIntegrity.html)
- [Sur les règles et les contraintes d'intégrité dans les systèmes de bases de données](https://www.comp.nus.edu.sg/~lingtw/papers/IST92.teopk.pdf)
- [Utiliser l'intégrité référentielle pour appliquer les règles métier de base dans Oracle](https://www.techrepublic.com/article/use-referential-integrity-to-enforce-basic-business-rules-in-oracle/)
- [Optimiser la réutilisation de la logique métier avec la logique réactive](https://dzone.com/articles/maximizing-business-logic)
