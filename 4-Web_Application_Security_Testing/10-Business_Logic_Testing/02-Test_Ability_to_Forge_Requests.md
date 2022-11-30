# Tester la capacité à falsifier des requêtes

|ID          |
|------------|
|WSTG-BUSL-02|

## Sommaire

Les requêtes falsifiées sont une méthode utilisée par les attaquants pour contourner l'application GUI frontale afin de soumettre directement des informations pour le traitement back-end. L'objectif de l'attaquant est d'envoyer des requêtes HTTP POST/GET via un proxy d'interception avec des valeurs de données qui ne sont pas prises en charge, protégées ou attendues par la logique métier des applications. Certains exemples de requêtes falsifiées incluent l'exploitation de paramètres devinables ou prévisibles ou l'exposition de fonctionnalités et fonctionnalités "cachées" telles que l'activation du débogage ou la présentation d'écrans ou de fenêtres spéciaux qui sont très utiles pendant le développement mais qui peuvent divulguer des informations ou contourner la logique métier.

Les vulnérabilités liées à la capacité de falsifier des requêtes sont propres à chaque application et différentes de la validation des données de la logique métier en ce sens qu'elles se concentrent sur la rupture du flux de travail de la logique métier.

Les applications doivent avoir mis en place des vérifications logiques pour empêcher le système d'accepter des demandes falsifiées qui pourraient permettre aux attaquants d'exploiter la logique métier, le processus ou le flux de l'application. La contrefaçon de demande n'est pas nouvelle ; l'attaquant utilise un proxy d'interception pour envoyer des requêtes HTTP POST/GET à l'application. Grâce aux falsifications de requêtes, les attaquants peuvent contourner la logique ou le processus métier en trouvant, prédisant et manipulant des paramètres pour faire croire à l'application qu'un processus ou une tâche a ou n'a pas eu lieu.

En outre, les requêtes falsifiées peuvent permettre de subventionner le flux de logique programmatique ou métier en invoquant des fonctionnalités ou fonctionnalités "cachées" telles que le débogage initialement utilisé par les développeurs et les testeurs, parfois appelé ["œuf de Pâques"](http://en.wikipedia. org/wiki/Easter_egg_(media)). "Un œuf de Pâques est une blague intérieure intentionnelle, un message caché ou une fonctionnalité dans une œuvre telle qu'un programme informatique, un film, un livre ou des mots croisés. Selon le concepteur de jeux Warren Robinett, le terme a été inventé chez Atari par le personnel qui a été alerté de la présence d'un message secret qui avait été caché par Robinett dans son jeu déjà largement diffusé, Adventure. On a dit que le nom évoquait l'idée d'une chasse aux œufs de Pâques traditionnelle.

### Exemple 1

Supposons qu'un site de théâtre e-commerce permette aux utilisateurs de sélectionner leur billet, d'appliquer une remise unique de 10 % sur l'ensemble de la vente, d'afficher le sous-total et de proposer la vente. Si un attaquant est capable de voir à travers un proxy que l'application a un champ caché (de 1 ou 0) utilisé par la logique métier pour déterminer si une remise a été prise ou non. L'attaquant est alors en mesure de soumettre plusieurs fois la valeur 1 ou "aucune remise n'a été prise" pour profiter de la même remise plusieurs fois.

### Exemple 2

Supposons qu'un jeu vidéo en ligne paie des jetons pour les points marqués pour trouver des trésors de pirates et des pirates et pour chaque niveau terminé. Ces jetons peuvent ensuite être échangés contre des prix. De plus, les points de chaque niveau ont une valeur multiplicatrice égale au niveau. Si un attaquant pouvait voir à travers un proxy que l'application a un champ caché utilisé pendant le développement et les tests pour accéder rapidement aux niveaux les plus élevés du jeu, il pourrait rapidement atteindre les niveaux les plus élevés et accumuler rapidement des points non gagnés.

De plus, si un attaquant pouvait voir via un proxy que l'application a un champ caché utilisé pendant le développement et les tests pour activer un journal indiquant où d'autres joueurs en ligne ou un trésor caché se trouvaient en relation avec l'attaquant, il pourrait alors pour aller rapidement à ces endroits et marquer des points.

## Objectifs des tests

- Examinez la documentation du projet à la recherche de fonctionnalités devinables, prévisibles ou masquées des champs.
- Insérez des données logiquement valides afin de contourner le flux de travail normal de la logique métier.

## Comment tester

### En identifiant des valeurs devinables

- À l'aide d'un proxy d'interception, observez le HTTP POST/GET à la recherche d'indications indiquant que les valeurs s'incrémentent à intervalles réguliers ou sont facilement devinables.
- S'il s'avère qu'une certaine valeur est devinable, cette valeur peut être modifiée et on peut obtenir une visibilité inattendue.

### En identifiant les options cachées

- À l'aide d'un proxy d'interception, observez le HTTP POST/GET à la recherche d'indications de fonctionnalités cachées telles que le débogage qui peuvent être activées ou activées.
- Si vous en trouvez, essayez de deviner et modifiez ces valeurs pour obtenir une réponse ou un comportement différent de l'application.

## Cas de test associés

- [Test des variables de session exposées](../06-Session_Management_Testing/04-Testing_for_Exposed_Session_Variables.md)
- [Test de falsification de requête intersite (CSRF)](../06-Session_Management_Testing/05-Testing_for_Cross_Site_Request_Forgery.md)
- [Test d'énumération de compte et de compte d'utilisateur devinable] (../03-Identity_Management_Testing/04-Testing_for_Account_Enumeration_and_Guessable_User_Account.md)

## Correction

L'application doit être suffisamment intelligente et conçue avec une logique métier qui empêchera les attaquants de prédire et de manipuler les paramètres pour subvertir le flux de logique programmatique ou métier, ou d'exploiter des fonctionnalités cachées/non documentées telles que le débogage.

## Outils

- [Proxy d'attaque Zed OWASP (ZAP)] (https://www.zaproxy.org)
- [Burp Suite] (https://portswigger.net/burp)

## Références

- [Cross Site Request Forgery - Légitimation des fausses requêtes](http://www.stan.gr/2012/11/cross-site-request-forgery-legitimazing.html)
- [Easter Eggs](https://en.wikipedia.org/wiki/Easter_egg_(media))
- [Top 10 des Easter Eggs logiciels] (https://lifehacker.com/371083/top-10-software-easter-eggs)
