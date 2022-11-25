# Test de la fonctionnalité de déconnexion

|ID          |
|------------|
|WSTG-SESS-06|

## Sommaire

La fin de session est une partie importante du cycle de vie de la session. Réduire au minimum la durée de vie des jetons de session diminue la probabilité d'une attaque de piratage de session réussie. Cela peut être considéré comme un contrôle contre la prévention d'autres attaques telles que Cross Site Scripting et Cross Site Request Forgery. De telles attaques sont connues pour s'appuyer sur un utilisateur ayant une session authentifiée présente. Ne pas avoir de terminaison de session sécurisée ne fait qu'augmenter la surface d'attaque pour l'une de ces attaques.

Une fin de session sécurisée nécessite au moins les composants suivants :

- Disponibilité des commandes de l'interface utilisateur permettant à l'utilisateur de se déconnecter manuellement.
- Fin de session après un certain temps d'inactivité (session timeout).
- Invalidation correcte de l'état de la session côté serveur.

Plusieurs problèmes peuvent empêcher la fin effective d'une session. Pour l'application Web sécurisée idéale, un utilisateur doit pouvoir mettre fin à tout moment via l'interface utilisateur. Chaque page doit contenir un bouton de déconnexion à un endroit où il est directement visible. Des fonctions de déconnexion peu claires ou ambiguës pourraient amener l'utilisateur à ne pas faire confiance à ces fonctionnalités.

Une autre erreur courante dans la fin de session est que le jeton de session côté client est défini sur une nouvelle valeur alors que l'état côté serveur reste actif et peut être réutilisé en redéfinissant le cookie de session sur la valeur précédente. Parfois, seul un message de confirmation est affiché à l'utilisateur sans effectuer aucune autre action. Cela devrait être évité.

Certains frameworks d'applications Web s'appuient uniquement sur le cookie de session pour identifier l'utilisateur connecté. L'identifiant de l'utilisateur est intégré dans la valeur de cookie (cryptée). Le serveur d'application n'effectue aucun suivi côté serveur de la session. Lors de la déconnexion, le cookie de session est supprimé du navigateur. Cependant, comme l'application n'effectue aucun suivi, elle ne sait pas si une session est déconnectée ou non. Ainsi, en réutilisant un cookie de session, il est possible d'accéder à la session authentifiée. Un exemple bien connu de ceci est la fonctionnalité d'authentification par formulaires dans ASP.NET.

Les utilisateurs de navigateurs Web ne se soucient souvent pas qu'une application soit toujours ouverte et ferment simplement le navigateur ou un onglet. Une application Web doit être consciente de ce comportement et mettre automatiquement fin à la session côté serveur après un laps de temps défini.

L'utilisation d'un système d'authentification unique (SSO) au lieu d'un schéma d'authentification spécifique à l'application entraîne souvent la coexistence de plusieurs sessions qui doivent être terminées séparément. Par exemple, la fin de la session spécifique à l'application ne met pas fin à la session dans le système SSO. Revenir au portail SSO offre à l'utilisateur la possibilité de se reconnecter à l'application où la déconnexion a été effectuée juste avant. D'un autre côté, une fonction de déconnexion dans un système SSO ne provoque pas nécessairement la fin de la session dans les applications connectées.

## Objectifs des tests

- Évaluer l'interface utilisateur de déconnexion.
- Analysez le délai d'expiration de la session et si la session est correctement tuée après la déconnexion.

## Comment tester

### Test de l'interface utilisateur de déconnexion

Vérifiez l'apparence et la visibilité de la fonctionnalité de déconnexion dans l'interface utilisateur. À cette fin, visualisez chaque page du point de vue d'un utilisateur qui a l'intention de se déconnecter de l'application Web.

> Certaines propriétés indiquent une bonne interface utilisateur de déconnexion :
>
> - Un bouton de déconnexion est présent sur toutes les pages de l'application web.
> - Le bouton de déconnexion doit être identifié rapidement par un utilisateur qui souhaite se déconnecter de l'application Web.
> - Après le chargement d'une page, le bouton de déconnexion doit être visible sans défilement.
> - Idéalement, le bouton de déconnexion est placé dans une zone de la page qui est fixe dans la fenêtre d'affichage du navigateur et non affectée par le défilement du contenu.

### Test de la fin de session côté serveur

Tout d'abord, stockez les valeurs des cookies utilisés pour identifier une session. Invoquez la fonction de déconnexion et observez le comportement de l'application, notamment en ce qui concerne les cookies de session. Essayez d'accéder à une page qui n'est visible que dans une session authentifiée, par ex. en utilisant le bouton de retour du navigateur. Si une version en cache de la page s'affiche, utilisez le bouton de rechargement pour actualiser la page à partir du serveur. Si la fonction de déconnexion entraîne la définition d'une nouvelle valeur pour les cookies de session, restaurez l'ancienne valeur des cookies de session et rechargez une page à partir de la zone authentifiée de l'application. Si ces tests ne montrent aucune vulnérabilité sur une page particulière, essayez au moins d'autres pages de l'application considérées comme critiques pour la sécurité, afin de vous assurer que la fin de session est correctement reconnue par ces zones de l'application.

> Aucune donnée qui ne devrait être visible que par des utilisateurs authentifiés ne doit être visible sur les pages examinées lors de l'exécution des tests. Idéalement, l'application redirige vers une zone publique ou un formulaire de connexion tout en accédant à des zones authentifiées après la fin de la session. Cela ne devrait pas être nécessaire pour la sécurité de l'application, mais la définition de nouvelles valeurs pour les cookies de session après la déconnexion est généralement considérée comme une bonne pratique.

### Test du délai d'expiration de la session

Essayez de déterminer un délai d'expiration de session en effectuant des requêtes vers une page dans la zone authentifiée de l'application Web avec des retards croissants. Si le comportement de déconnexion apparaît, le délai utilisé correspond approximativement à la valeur du délai d'expiration de la session.

> Les mêmes résultats que pour les tests de terminaison de session côté serveur décrits précédemment sont exceptés par une déconnexion causée par un délai d'inactivité.
>
> La valeur appropriée pour le délai d'expiration de la session dépend de l'objectif de l'application et doit être un équilibre entre sécurité et convivialité. Dans une application bancaire, cela n'a aucun sens de garder une session inactive plus de 15 minutes. D'un autre côté, un court délai d'attente dans un wiki ou un forum pourrait ennuyer les utilisateurs qui tapent de longs articles avec des demandes de connexion inutiles. Là, des délais d'attente d'une heure et plus peuvent être acceptables.

### Test de fin de session dans les environnements d'authentification unique (authentification unique)

Effectuez une déconnexion dans l'application testée. Vérifiez s'il existe un portail central ou un répertoire d'applications permettant à l'utilisateur de se reconnecter à l'application sans authentification. Teste si l'application demande à l'utilisateur de s'authentifier, si l'URL d'un point d'entrée à l'application est demandée. Une fois connecté à l'application testée, effectuez une déconnexion dans le système SSO. Essayez ensuite d'accéder à une zone authentifiée de l'application testée.

> Il est prévu que l'invocation d'une fonction de déconnexion dans une application Web connectée à un système SSO ou dans le système SSO lui-même provoque la fermeture globale de toutes les sessions. Une authentification de l'utilisateur doit être requise pour accéder à l'application après la déconnexion du système SSO et de l'application connectée.

## Outils

- [Burp Suite - Répéteur](https://portswigger.net/burp/documentation/desktop/tools/repeater)

## Références

### Papiers blanc

- [Attaques par rejeu de cookies dans ASP.NET lors de l'utilisation de l'authentification par formulaires](https://www.vanstechelman.eu/content/cookie-replay-attacks-in-aspnet-when-using-forms-authentication)
