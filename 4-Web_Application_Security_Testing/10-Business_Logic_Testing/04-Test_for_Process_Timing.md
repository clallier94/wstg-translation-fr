# Test de la synchronisation du processus

|ID          |
|------------|
|WSTG-BUSL-04|

## Sommaire

Il est possible que des attaquants puissent recueillir des informations sur une application en surveillant le temps nécessaire pour accomplir une tâche ou donner une réponse. De plus, les attaquants peuvent être en mesure de manipuler et de casser les flux de processus métier conçus en gardant simplement les sessions actives ouvertes et en ne soumettant pas leurs transactions dans le délai « attendu ».

Les vulnérabilités de la logique de synchronisation des processus sont uniques en ce sens que ces cas d'utilisation abusive manuelle doivent être créés en tenant compte de la synchronisation de l'exécution et des transactions qui sont spécifiques à l'application/au système.

Le temps de traitement peut donner/fuir des informations sur ce qui est fait dans les processus d'arrière-plan de l'application/du système. Si une application permet aux utilisateurs de deviner quel sera le prochain résultat particulaire en traitant les variations de temps, les utilisateurs pourront s'adapter en conséquence et changer de comportement en fonction de l'attente et "jouer avec le système".

### Exemple 1

Les jeux d'argent vidéo/machines à sous peuvent prendre plus de temps pour traiter une transaction juste avant un paiement important. Cela permettrait aux joueurs astucieux de parier des montants minimaux jusqu'à ce qu'ils voient le long temps de traitement qui les inciterait alors à parier le maximum.

### Exemple 2

De nombreux processus de connexion au système demandent le nom d'utilisateur et le mot de passe. Si vous regardez attentivement, vous pourrez peut-être voir que la saisie d'un nom d'utilisateur et d'un mot de passe utilisateur invalides prend plus de temps pour renvoyer une erreur que la saisie d'un nom d'utilisateur valide et d'un mot de passe utilisateur invalide. Cela peut permettre à l'attaquant de savoir s'il a un nom d'utilisateur valide et s'il n'a pas besoin de se fier au message de l'interface graphique.

![Exemple de flux de contrôle du formulaire de connexion](images/Control_Flow_of_Login_Form.jpg)\
*Figure 4.10.4-1 : Exemple de flux de contrôle du formulaire de connexion*

### Exemple 3

La plupart des arénas ou des agences de voyages ont des applications de billetterie qui permettent aux utilisateurs d'acheter des billets et de réserver des places. Lorsque l'utilisateur demande les billets, les sièges sont verrouillés ou réservés en attente de paiement. Que se passe-t-il si un attaquant continue de réserver des sièges mais ne vérifie pas ? Les places seront-elles libérées ou aucun billet ne sera-t-il vendu ? Certains vendeurs de billets n'accordent plus que 5 minutes aux utilisateurs pour effectuer une transaction ou la transaction est invalidée.

### Exemple 4

Supposons qu'un site de commerce électronique de métaux précieux permette aux utilisateurs d'effectuer des achats avec un devis basé sur le prix du marché au moment où ils se connectent. Que se passe-t-il si un attaquant se connecte et passe une commande mais ne termine la transaction que plus tard dans la journée uniquement lorsque le prix des métaux augmente ? L'attaquant obtiendra-t-il le prix inférieur initial ?

## Objectifs des tests

- Examiner la documentation du projet pour les fonctionnalités du système qui peuvent être affectées par le temps.
- Développer et exécuter des cas d'abus.

## Comment tester

Le testeur doit identifier quels processus sont dépendants du temps, s'il s'agissait d'une fenêtre pour qu'une tâche soit accomplie, ou s'il s'agissait d'un temps d'exécution entre deux processus qui pouvait permettre le contournement de certains contrôles.

Ensuite, il est préférable d'automatiser les requêtes qui abuseront des processus découverts ci-dessus, car les outils sont mieux adaptés pour analyser le timing et sont plus précis que les tests manuels. Si cela n'est pas possible, des tests manuels peuvent toujours être utilisés.

Le testeur doit dessiner un schéma du déroulement du processus, des points d'injection, et préparer les requêtes en amont pour les lancer sur les processus vulnérables. Une fois cela fait, une analyse approfondie doit être effectuée pour identifier les différences dans l'exécution du processus et si le processus se comporte mal par rapport à la logique métier convenue.

## Cas de test associés

- [Test des attributs des cookies](../06-Session_Management_Testing/02-Testing_for_Cookies_Attributes.md)
- [Délai d'expiration de la session de test](../06-Session_Management_Testing/07-Testing_Session_Timeout.md)

## Correction

Développez des applications en tenant compte du temps de traitement. Si les attaquants pouvaient éventuellement tirer un certain avantage en connaissant les différents temps de traitement et résultats, ajoutez des étapes ou un traitement supplémentaires afin que, quels que soient les résultats, ils soient fournis dans le même laps de temps.

De plus, l'application/le système doit avoir un mécanisme en place pour empêcher les attaquants d'étendre les transactions sur une durée "acceptable". Cela peut être fait en annulant ou en réinitialisant les transactions après un laps de temps spécifié, comme certains vendeurs de billets l'utilisent actuellement.
