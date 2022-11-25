# Test de la fonctionnalit� de d�connexion

|ID          |
|------------|
|WSTG-SESS-06|

## Sommaire

La fin de session est une partie importante du cycle de vie de la session. R�duire au minimum la dur�e de vie des jetons de session diminue la probabilit� d'une attaque de piratage de session r�ussie. Cela peut �tre consid�r� comme un contr�le contre la pr�vention d'autres attaques telles que Cross Site Scripting et Cross Site Request Forgery. De telles attaques sont connues pour s'appuyer sur un utilisateur ayant une session authentifi�e pr�sente. Ne pas avoir de terminaison de session s�curis�e ne fait qu'augmenter la surface d'attaque pour l'une de ces attaques.

Une fin de session s�curis�e n�cessite au moins les composants suivants�:

- Disponibilit� des commandes de l'interface utilisateur permettant � l'utilisateur de se d�connecter manuellement.
- Fin de session apr�s un certain temps d'inactivit� (session timeout).
- Invalidation correcte de l'�tat de la session c�t� serveur.

Plusieurs probl�mes peuvent emp�cher la fin effective d'une session. Pour l'application Web s�curis�e id�ale, un utilisateur doit pouvoir mettre fin � tout moment via l'interface utilisateur. Chaque page doit contenir un bouton de d�connexion � un endroit o� il est directement visible. Des fonctions de d�connexion peu claires ou ambigu�s pourraient amener l'utilisateur � ne pas faire confiance � ces fonctionnalit�s.

Une autre erreur courante dans la fin de session est que le jeton de session c�t� client est d�fini sur une nouvelle valeur alors que l'�tat c�t� serveur reste actif et peut �tre r�utilis� en red�finissant le cookie de session sur la valeur pr�c�dente. Parfois, seul un message de confirmation est affich� � l'utilisateur sans effectuer aucune autre action. Cela devrait �tre �vit�.

Certains frameworks d'applications Web s'appuient uniquement sur le cookie de session pour identifier l'utilisateur connect�. L'identifiant de l'utilisateur est int�gr� dans la valeur de cookie (crypt�e). Le serveur d'application n'effectue aucun suivi c�t� serveur de la session. Lors de la d�connexion, le cookie de session est supprim� du navigateur. Cependant, comme l'application n'effectue aucun suivi, elle ne sait pas si une session est d�connect�e ou non. Ainsi, en r�utilisant un cookie de session, il est possible d'acc�der � la session authentifi�e. Un exemple bien connu de ceci est la fonctionnalit� d'authentification par formulaires dans ASP.NET.

Les utilisateurs de navigateurs Web ne se soucient souvent pas qu'une application soit toujours ouverte et ferment simplement le navigateur ou un onglet. Une application Web doit �tre consciente de ce comportement et mettre automatiquement fin � la session c�t� serveur apr�s un laps de temps d�fini.

L'utilisation d'un syst�me d'authentification unique (SSO) au lieu d'un sch�ma d'authentification sp�cifique � l'application entra�ne souvent la coexistence de plusieurs sessions qui doivent �tre termin�es s�par�ment. Par exemple, la fin de la session sp�cifique � l'application ne met pas fin � la session dans le syst�me SSO. Revenir au portail SSO offre � l'utilisateur la possibilit� de se reconnecter � l'application o� la d�connexion a �t� effectu�e juste avant. D'un autre c�t�, une fonction de d�connexion dans un syst�me SSO ne provoque pas n�cessairement la fin de la session dans les applications connect�es.

## Objectifs des tests

- �valuer l'interface utilisateur de d�connexion.
- Analysez le d�lai d'expiration de la session et si la session est correctement tu�e apr�s la d�connexion.

## Comment tester

### Test de l'interface utilisateur de d�connexion

V�rifiez l'apparence et la visibilit� de la fonctionnalit� de d�connexion dans l'interface utilisateur. � cette fin, visualisez chaque page du point de vue d'un utilisateur qui a l'intention de se d�connecter de l'application Web.

> Certaines propri�t�s indiquent une bonne interface utilisateur de d�connexion�:
>
> - Un bouton de d�connexion est pr�sent sur toutes les pages de l'application web.
> - Le bouton de d�connexion doit �tre identifi� rapidement par un utilisateur qui souhaite se d�connecter de l'application Web.
> - Apr�s le chargement d'une page, le bouton de d�connexion doit �tre visible sans d�filement.
> - Id�alement, le bouton de d�connexion est plac� dans une zone de la page qui est fixe dans la fen�tre d'affichage du navigateur et non affect�e par le d�filement du contenu.

### Test de la fin de session c�t� serveur

Tout d'abord, stockez les valeurs des cookies utilis�s pour identifier une session. Invoquez la fonction de d�connexion et observez le comportement de l'application, notamment en ce qui concerne les cookies de session. Essayez d'acc�der � une page qui n'est visible que dans une session authentifi�e, par ex. en utilisant le bouton de retour du navigateur. Si une version en cache de la page s'affiche, utilisez le bouton de rechargement pour actualiser la page � partir du serveur. Si la fonction de d�connexion entra�ne la d�finition d'une nouvelle valeur pour les cookies de session, restaurez l'ancienne valeur des cookies de session et rechargez une page � partir de la zone authentifi�e de l'application. Si ces tests ne montrent aucune vuln�rabilit� sur une page particuli�re, essayez au moins d'autres pages de l'application consid�r�es comme critiques pour la s�curit�, afin de vous assurer que la fin de session est correctement reconnue par ces zones de l'application.

> Aucune donn�e qui ne devrait �tre visible que par des utilisateurs authentifi�s ne doit �tre visible sur les pages examin�es lors de l'ex�cution des tests. Id�alement, l'application redirige vers une zone publique ou un formulaire de connexion tout en acc�dant � des zones authentifi�es apr�s la fin de la session. Cela ne devrait pas �tre n�cessaire pour la s�curit� de l'application, mais la d�finition de nouvelles valeurs pour les cookies de session apr�s la d�connexion est g�n�ralement consid�r�e comme une bonne pratique.

### Test du d�lai d'expiration de la session

Essayez de d�terminer un d�lai d'expiration de session en effectuant des requ�tes vers une page dans la zone authentifi�e de l'application Web avec des retards croissants. Si le comportement de d�connexion appara�t, le d�lai utilis� correspond approximativement � la valeur du d�lai d'expiration de la session.

> Les m�mes r�sultats que pour les tests de terminaison de session c�t� serveur d�crits pr�c�demment sont except�s par une d�connexion caus�e par un d�lai d'inactivit�.
>
> La valeur appropri�e pour le d�lai d'expiration de la session d�pend de l'objectif de l'application et doit �tre un �quilibre entre s�curit� et convivialit�. Dans une application bancaire, cela n'a aucun sens de garder une session inactive plus de 15 minutes. D'un autre c�t�, un court d�lai d'attente dans un wiki ou un forum pourrait ennuyer les utilisateurs qui tapent de longs articles avec des demandes de connexion inutiles. L�, des d�lais d'attente d'une heure et plus peuvent �tre acceptables.

###�Test de fin de session dans les environnements d'authentification unique (authentification unique)

Effectuez une d�connexion dans l'application test�e. V�rifiez s'il existe un portail central ou un r�pertoire d'applications permettant � l'utilisateur de se reconnecter � l'application sans authentification. Teste si l'application demande � l'utilisateur de s'authentifier, si l'URL d'un point d'entr�e � l'application est demand�e. Une fois connect� � l'application test�e, effectuez une d�connexion dans le syst�me SSO. Essayez ensuite d'acc�der � une zone authentifi�e de l'application test�e.

> Il est pr�vu que l'invocation d'une fonction de d�connexion dans une application Web connect�e � un syst�me SSO ou dans le syst�me SSO lui-m�me provoque la fermeture globale de toutes les sessions. Une authentification de l'utilisateur doit �tre requise pour acc�der � l'application apr�s la d�connexion du syst�me SSO et de l'application connect�e.

## Outils

- [Burp Suite - R�p�teur](https://portswigger.net/burp/documentation/desktop/tools/repeater)

## R�f�rences

### Papiers blanc

- [Attaques par rejeu de cookies dans ASP.NET lors de l'utilisation de l'authentification par formulaires](https://www.vanstechelman.eu/content/cookie-replay-attacks-in-aspnet-when-using-forms-authentication)
