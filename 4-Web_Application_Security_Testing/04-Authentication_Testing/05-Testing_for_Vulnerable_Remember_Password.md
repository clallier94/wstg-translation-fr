# Test de m�morisation du mot de passe vuln�rable

|ID          |
|------------|
|WSTG-ATHN-05|

## Sommaire

Les informations d'identification sont la technologie d'authentification la plus largement utilis�e. En raison d'une telle utilisation des paires nom d'utilisateur-mot de passe, les utilisateurs ne sont plus en mesure de g�rer correctement leurs informations d'identification dans la multitude d'applications utilis�es.

Afin d'aider les utilisateurs avec leurs informations d'identification, plusieurs technologies ont fait surface�:

- Les applications fournissent une fonctionnalit� * se souvenir de moi * qui permet � l'utilisateur de rester authentifi� pendant de longues p�riodes, sans lui demander � nouveau ses informations d'identification.
- Gestionnaires de mots de passe - y compris les gestionnaires de mots de passe du navigateur - qui permettent � l'utilisateur de stocker ses informations d'identification de mani�re s�curis�e et de les injecter ult�rieurement dans des formulaires utilisateur sans aucune intervention de l'utilisateur.

## Objectifs des tests

- Validez que la session g�n�r�e est g�r�e de mani�re s�curis�e et ne mettez pas en danger les identifiants de l'utilisateur.

## Comment tester

Comme ces m�thodes offrent une meilleure exp�rience utilisateur et permettent � l'utilisateur d'oublier ses informations d'identification, elles augmentent la surface d'attaque. Quelques candidatures�:

- Stockez les informations d'identification de mani�re cod�e dans les m�canismes de stockage du navigateur, ce qui peut �tre v�rifi� en suivant le [sc�nario de test de stockage Web](../11-Client-side_Testing/12-Testing_Browser_Storage.md) et en passant par l'[analyse de session ](../06-Session_Management_Testing/01-Testing_for_Session_Management_Schema.md#session-analysis) sc�narios. Les informations d'identification ne doivent en aucun cas �tre stock�es dans l'application c�t� client et doivent �tre remplac�es par des jetons g�n�r�s c�t� serveur.
- Injectez automatiquement les identifiants de l'utilisateur qui peuvent �tre abus�s par�:
    - [ClickJacking](../11-Client-side_Testing/09-Testing_for_Clickjacking.md) attaques.
    - [CSRF](../06-Session_Management_Testing/05-Testing_for_Cross_Site_Request_Forgery.md).
- Les jetons doivent �tre analys�s en termes de dur�e de vie, o� certains jetons n'expirent jamais et mettent les utilisateurs en danger si ces jetons sont vol�s. Assurez-vous de suivre le sc�nario de test [timeout de session](../06-Session_Management_Testing/07-Testing_Session_Timeout.md).

## Correction

- Suivez les bonnes pratiques [de gestion de session](https://cheatsheetseries.owasp.org/cheatsheets/Session_Management_Cheat_Sheet.html).
- Assurez-vous qu'aucune information d'identification n'est stock�e en texte clair ou n'est facilement r�cup�rable sous des formes cod�es ou crypt�es dans les m�canismes de stockage du navigateur�; ils doivent �tre stock�s c�t� serveur et suivre les bonnes pratiques de [stockage des mots de passe](https://cheatsheetseries.owasp.org/cheatsheets/Password_Storage_Cheat_Sheet.html).
