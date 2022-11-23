# Test de mémorisation du mot de passe vulnérable

|ID          |
|------------|
|WSTG-ATHN-05|

## Sommaire

Les informations d'identification sont la technologie d'authentification la plus largement utilisée. En raison d'une telle utilisation des paires nom d'utilisateur-mot de passe, les utilisateurs ne sont plus en mesure de gérer correctement leurs informations d'identification dans la multitude d'applications utilisées.

Afin d'aider les utilisateurs avec leurs informations d'identification, plusieurs technologies ont fait surface :

- Les applications fournissent une fonctionnalité * se souvenir de moi * qui permet à l'utilisateur de rester authentifié pendant de longues périodes, sans lui demander à nouveau ses informations d'identification.
- Gestionnaires de mots de passe - y compris les gestionnaires de mots de passe du navigateur - qui permettent à l'utilisateur de stocker ses informations d'identification de manière sécurisée et de les injecter ultérieurement dans des formulaires utilisateur sans aucune intervention de l'utilisateur.

## Objectifs des tests

- Validez que la session générée est gérée de manière sécurisée et ne mettez pas en danger les identifiants de l'utilisateur.

## Comment tester

Comme ces méthodes offrent une meilleure expérience utilisateur et permettent à l'utilisateur d'oublier ses informations d'identification, elles augmentent la surface d'attaque. Quelques candidatures :

- Stockez les informations d'identification de manière codée dans les mécanismes de stockage du navigateur, ce qui peut être vérifié en suivant le [scénario de test de stockage Web](../11-Client-side_Testing/12-Testing_Browser_Storage.md) et en passant par l'[analyse de session ](../06-Session_Management_Testing/01-Testing_for_Session_Management_Schema.md#session-analysis) scénarios. Les informations d'identification ne doivent en aucun cas être stockées dans l'application côté client et doivent être remplacées par des jetons générés côté serveur.
- Injectez automatiquement les identifiants de l'utilisateur qui peuvent être abusés par :
    - [ClickJacking](../11-Client-side_Testing/09-Testing_for_Clickjacking.md) attaques.
    - [CSRF](../06-Session_Management_Testing/05-Testing_for_Cross_Site_Request_Forgery.md).
- Les jetons doivent être analysés en termes de durée de vie, où certains jetons n'expirent jamais et mettent les utilisateurs en danger si ces jetons sont volés. Assurez-vous de suivre le scénario de test [timeout de session](../06-Session_Management_Testing/07-Testing_Session_Timeout.md).

## Correction

- Suivez les bonnes pratiques [de gestion de session](https://cheatsheetseries.owasp.org/cheatsheets/Session_Management_Cheat_Sheet.html).
- Assurez-vous qu'aucune information d'identification n'est stockée en texte clair ou n'est facilement récupérable sous des formes codées ou cryptées dans les mécanismes de stockage du navigateur ; ils doivent être stockés côté serveur et suivre les bonnes pratiques de [stockage des mots de passe](https://cheatsheetseries.owasp.org/cheatsheets/Password_Storage_Cheat_Sheet.html).
