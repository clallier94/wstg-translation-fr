# Test des informations d'identification par défaut

|ID          |
|------------|
|WSTG-ATHN-02|

## Sommaire

De nombreuses applications Web et périphériques matériels ont des mots de passe par défaut pour le compte administratif intégré. Bien que dans certains cas, ceux-ci puissent être générés de manière aléatoire, ils sont souvent statiques, ce qui signifie qu'ils peuvent être facilement devinés ou obtenus par un attaquant.

De plus, lorsque de nouveaux utilisateurs sont créés sur les applications, ceux-ci peuvent avoir des mots de passe prédéfinis. Ceux-ci peuvent être soit générés automatiquement par l'application, soit créés manuellement par le personnel. Dans les deux cas, s'ils ne sont pas générés de manière sécurisée, les mots de passe peuvent être devinés par un attaquant.

## Objectifs des tests

- Déterminez si l'application possède des comptes d'utilisateurs avec des mots de passe par défaut.
- Vérifiez si de nouveaux comptes d'utilisateurs sont créés avec des mots de passe faibles ou prévisibles.

## Comment tester

### Test des informations d'identification par défaut du fournisseur

La première étape pour identifier les mots de passe par défaut consiste à identifier le logiciel utilisé. Ceci est couvert en détail dans la section [La collecte d'informations](../01-Information_Gathering/README.md) du guide.

Une fois le logiciel identifié, essayez de savoir s'il utilise des mots de passe par défaut, et si oui, quels sont-ils. Cela devrait inclure :

- Recherche du "[LOGICIEL] mot de passe par défaut".
- Examiner le manuel ou la documentation du fournisseur.
- Vérification des bases de données de mots de passe par défaut courantes, telles que [CIRT.net](https://cirt.net/passwords), [SecLists Default Passwords](https://github.com/danielmiessler/SecLists/tree/master/Passwords/ Default-Credentials) ou [DefaultCreds-cheat-sheet](https://github.com/ihebski/DefaultCreds-cheat-sheet/blob/main/DefaultCreds-Cheat-Sheet.csv).
- Inspecter le code source de l'application (si disponible).
- Installer l'application sur une machine virtuelle et l'inspecter.
- Inspecter le matériel physique pour les autocollants (souvent présents sur les périphériques réseau).

Si un mot de passe par défaut est introuvable, essayez les options courantes telles que :

- "admin", "password", "12345", ou d'autres [mots de passe par défaut communs](https://github.com/nixawk/fuzzdb/blob/master/bruteforce/passwds/default_devices_users%2Bpasswords.txt).
- Un mot de passe vide ou vide.
- Le numéro de série ou l'adresse MAC de l'appareil.

Si le nom d'utilisateur est inconnu, il existe différentes options pour énumérer les utilisateurs, décrites dans le guide [Test de l'énumération des comptes](../03-Identity_Management_Testing/04-Testing_for_Account_Enumeration_and_Guessable_User_Account.md). Vous pouvez également essayer des options courantes telles que "admin", "root" ou "system".

### Test des mots de passe par défaut de l'organisation

Lorsque le personnel d'une organisation crée manuellement des mots de passe pour de nouveaux comptes, il peut le faire de manière prévisible. Cela peut souvent être :

- Un seul mot de passe commun tel que "Password1".
- Détails spécifiques à l'organisation, tels que le nom ou l'adresse de l'organisation.
- Les mots de passe qui suivent un modèle simple, comme "Monday123" si le compte est créé un lundi.

Ces types de mots de passe sont souvent difficiles à identifier du point de vue de la boîte noire, à moins qu'ils ne puissent être devinés ou forcés avec succès. Cependant, ils sont faciles à identifier lors de l'exécution de tests en boîte grise ou en boîte blanche.

### Test des mots de passe par défaut générés par l'application

Si l'application génère automatiquement des mots de passe pour les nouveaux comptes d'utilisateurs, ceux-ci peuvent également être prévisibles. Afin de les tester, créez plusieurs comptes sur l'application avec des détails similaires en même temps et comparez les mots de passe qui leur sont attribués.

Les mots de passe peuvent être basés sur :

- Une seule chaîne statique partagée entre les comptes.
- Une partie hachée ou masquée des détails du compte, telle que `md5($username)`.
- Un algorithme basé sur le temps.
- Un générateur de nombres pseudo-aléatoires faibles (PRNG).

Ce type de problème est souvent difficile à identifier du point de vue de la boîte noire.

## Outils

- [Burp Intruder] (https://portswigger.net/burp/documentation/desktop/tools/intrus)
- [THC Hydra](https://github.com/vanhauser-thc/thc-hydra)
- [Nikto2](https://www.cirt.net/nikto2)

## Références

- [CIRT](https://cirt.net/passwords)
- [SecLists Default Passwords](https://github.com/danielmiessler/SecLists/tree/master/Passwords/Default-Credentials)
- [DefaultCreds-cheat-sheet](https://github.com/ihebski/DefaultCreds-cheat-sheet/blob/main/DefaultCreds-Cheat-Sheet.csv)
