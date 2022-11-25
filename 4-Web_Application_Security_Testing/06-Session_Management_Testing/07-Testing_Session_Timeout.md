# Test du délai d'expiration de la session

|ID          |
|------------|
|WSTG-SESS-07|

## Sommaire

Dans cette phase, les testeurs vérifient que l'application déconnecte automatiquement un utilisateur lorsque cet utilisateur a été inactif pendant un certain temps, en s'assurant qu'il n'est pas possible de "réutiliser" la même session et qu'aucune donnée sensible ne reste stockée dans le cache du navigateur .

Toutes les applications doivent implémenter un délai d'inactivité ou d'inactivité pour les sessions. Ce délai d'attente définit la durée pendant laquelle une session restera active en cas d'absence d'activité de l'utilisateur, fermant et invalidant la session sur la période d'inactivité définie depuis la dernière requête HTTP reçue par l'application Web pour un ID de session donné. Le délai d'expiration le plus approprié doit être un équilibre entre la sécurité (délai d'expiration plus court) et la convivialité (délai d'expiration plus long) et dépend fortement du niveau de sensibilité des données traitées par l'application. Par exemple, un délai de déconnexion de 60 minutes pour un forum public peut être acceptable, mais un délai aussi long serait trop long dans une application de banque à domicile (où un délai maximum de 15 minutes est recommandé). Dans tous les cas, toute application qui n'applique pas une déconnexion basée sur le délai d'attente doit être considérée comme non sécurisée, à moins qu'un tel comportement ne soit requis par une exigence fonctionnelle spécifique.

Le délai d'inactivité limite les chances qu'un attaquant doive deviner et utiliser un ID de session valide d'un autre utilisateur et, dans certaines circonstances, peut protéger les ordinateurs publics de la réutilisation de session. Cependant, si l'attaquant est capable de détourner une session donnée, le délai d'inactivité ne limite pas les actions de l'attaquant, car il peut générer périodiquement une activité sur la session pour maintenir la session active pendant de plus longues périodes.

La gestion et l'expiration du délai d'expiration de la session doivent être appliquées côté serveur. Si certaines données sous le contrôle du client sont utilisées pour appliquer le délai d'expiration de la session, par exemple en utilisant des valeurs de cookie ou d'autres paramètres du client pour suivre les références temporelles (par exemple, le nombre de minutes depuis l'heure de connexion), un attaquant pourrait les manipuler pour prolonger la session durée. L'application doit donc suivre le temps d'inactivité côté serveur et, une fois le délai expiré, invalider automatiquement la session de l'utilisateur actuel et supprimer toutes les données stockées sur le client.

Ces deux actions doivent être implémentées avec soin, afin d'éviter d'introduire des faiblesses qui pourraient être exploitées par un attaquant pour obtenir un accès non autorisé si l'utilisateur oubliait de se déconnecter de l'application. Plus précisément, en ce qui concerne la fonction de déconnexion, il est important de s'assurer que tous les jetons de session (par exemple, les cookies) sont correctement détruits ou rendus inutilisables, et que des contrôles appropriés sont appliqués côté serveur pour empêcher la réutilisation des jetons de session. Si de telles actions ne sont pas correctement effectuées, un attaquant pourrait rejouer ces jetons de session afin de « ressusciter » la session d'un utilisateur légitime et se faire passer pour lui (cette attaque est généralement connue sous le nom de « cookie replay »). Bien sûr, un facteur atténuant est que l'attaquant doit pouvoir accéder à ces jetons (qui sont stockés sur le PC de la victime), mais, dans une variété de cas, cela peut ne pas être impossible ou particulièrement difficile.

Le scénario le plus courant pour ce type d'attaque est un ordinateur public utilisé pour accéder à certaines informations privées (par exemple, messagerie Web, compte bancaire en ligne). Si l'utilisateur s'éloigne de l'ordinateur sans se déconnecter explicitement et que le délai d'expiration de la session n'est pas implémenté sur l'application, alors un attaquant pourrait accéder au même compte en appuyant simplement sur le bouton "retour" du navigateur.

## Objectifs des tests

- Valider qu'un délai d'expiration de session dur existe.

## Comment tester

### Test de la boîte noire

La même approche vue dans la section [Test de la fonctionnalité de déconnexion](06-Testing_for_Logout_Functionality.md) peut être appliquée lors de la mesure du délai de déconnexion.
La méthodologie de test est très similaire. Tout d'abord, les testeurs doivent vérifier s'il existe un délai d'attente, par exemple en se connectant et en attendant que le délai de déconnexion soit déclenché. Comme dans la fonction de déconnexion, une fois le délai d'expiration écoulé, tous les jetons de session doivent être détruits ou être inutilisables.

Ensuite, si le délai d'attente est configuré, les testeurs doivent comprendre si le délai d'attente est appliqué par le client ou par le serveur (ou les deux). Si le cookie de session n'est pas persistant (ou, plus généralement, le cookie de session ne stocke aucune donnée sur l'heure), les testeurs peuvent supposer que le délai d'attente est appliqué par le serveur. Si le cookie de session contient des données liées au temps (par exemple, l'heure de connexion, l'heure du dernier accès ou la date d'expiration d'un cookie persistant), il est possible que le client soit impliqué dans l'application du délai d'attente. Dans ce cas, les testeurs pourraient essayer de modifier le cookie (s'il n'est pas protégé cryptographiquement) et voir ce qu'il advient de la session. Par exemple, les testeurs peuvent définir la date d'expiration du cookie loin dans le futur et voir si la session peut être prolongée.

En règle générale, tout doit être vérifié côté serveur et il ne doit pas être possible, en réinitialisant les cookies de session aux valeurs précédentes, d'accéder à nouveau à l'application.

### Test de la boîte grise

Le testeur doit vérifier que :

- La fonction de déconnexion détruit effectivement tous les jetons de session, ou du moins les rend inutilisables,
- Le serveur effectue des vérifications appropriées sur l'état de la session, empêchant un attaquant de rejouer les identifiants de session précédemment détruits
- Un délai d'attente est appliqué et il est correctement appliqué par le serveur. Si le serveur utilise un délai d'expiration qui est lu à partir d'un jeton de session envoyé par le client (mais ce n'est pas conseillé), alors le jeton doit être protégé cryptographiquement contre la falsification.

Notez que le plus important est que l'application invalide la session côté serveur. Généralement, cela signifie que le code doit invoquer les méthodes appropriées, par ex. `HttpSession.invalidate()` en Java et `Session.abandon()` en .NET. Effacer les cookies du navigateur est conseillé, mais n'est pas strictement nécessaire, car si la session est correctement invalidée sur le serveur, avoir le cookie dans le navigateur n'aidera pas un attaquant.

## Références

### Ressources OWASP

- [Aide-mémoire de gestion de session](https://cheatsheetseries.owasp.org/cheatsheets/Session_Management_Cheat_Sheet.html)

