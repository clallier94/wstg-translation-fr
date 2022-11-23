# Test des informations d'identification par d�faut

|ID          |
|------------|
|WSTG-ATHN-02|

## Sommaire

De nombreuses applications Web et p�riph�riques mat�riels ont des mots de passe par d�faut pour le compte administratif int�gr�. Bien que dans certains cas, ceux-ci puissent �tre g�n�r�s de mani�re al�atoire, ils sont souvent statiques, ce qui signifie qu'ils peuvent �tre facilement devin�s ou obtenus par un attaquant.

De plus, lorsque de nouveaux utilisateurs sont cr��s sur les applications, ceux-ci peuvent avoir des mots de passe pr�d�finis. Ceux-ci peuvent �tre soit g�n�r�s automatiquement par l'application, soit cr��s manuellement par le personnel. Dans les deux cas, s'ils ne sont pas g�n�r�s de mani�re s�curis�e, les mots de passe peuvent �tre devin�s par un attaquant.

## Objectifs des tests

- D�terminez si l'application poss�de des comptes d'utilisateurs avec des mots de passe par d�faut.
- V�rifiez si de nouveaux comptes d'utilisateurs sont cr��s avec des mots de passe faibles ou pr�visibles.

## Comment tester

### Test des informations d'identification par d�faut du fournisseur

La premi�re �tape pour identifier les mots de passe par d�faut consiste � identifier le logiciel utilis�. Ceci est couvert en d�tail dans la section [La collecte d'informations](../01-Information_Gathering/README.md) du guide.

Une fois le logiciel identifi�, essayez de savoir s'il utilise des mots de passe par d�faut, et si oui, quels sont-ils. Cela devrait inclure :

- Recherche du "[LOGICIEL] mot de passe par d�faut".
- Examiner le manuel ou la documentation du fournisseur.
- V�rification des bases de donn�es de mots de passe par d�faut courantes, telles que [CIRT.net](https://cirt.net/passwords), [SecLists Default Passwords](https://github.com/danielmiessler/SecLists/tree/master/Passwords/ Default-Credentials) ou [DefaultCreds-cheat-sheet](https://github.com/ihebski/DefaultCreds-cheat-sheet/blob/main/DefaultCreds-Cheat-Sheet.csv).
- Inspecter le code source de l'application (si disponible).
- Installer l'application sur une machine virtuelle et l'inspecter.
- Inspecter le mat�riel physique pour les autocollants (souvent pr�sents sur les p�riph�riques r�seau).

Si un mot de passe par d�faut est introuvable, essayez les options courantes telles que�:

- "admin", "password", "12345", ou d'autres [mots de passe par d�faut communs](https://github.com/nixawk/fuzzdb/blob/master/bruteforce/passwds/default_devices_users%2Bpasswords.txt).
- Un mot de passe vide ou vide.
- Le num�ro de s�rie ou l'adresse MAC de l'appareil.

Si le nom d'utilisateur est inconnu, il existe diff�rentes options pour �num�rer les utilisateurs, d�crites dans le guide [Test de l'�num�ration des comptes](../03-Identity_Management_Testing/04-Testing_for_Account_Enumeration_and_Guessable_User_Account.md). Vous pouvez �galement essayer des options courantes telles que "admin", "root" ou "system".

### Test des mots de passe par d�faut de l'organisation

Lorsque le personnel d'une organisation cr�e manuellement des mots de passe pour de nouveaux comptes, il peut le faire de mani�re pr�visible. Cela peut souvent �tre :

- Un seul mot de passe commun tel que "Password1".
- D�tails sp�cifiques � l'organisation, tels que le nom ou l'adresse de l'organisation.
- Les mots de passe qui suivent un mod�le simple, comme "Monday123" si le compte est cr�� un lundi.

Ces types de mots de passe sont souvent difficiles � identifier du point de vue de la bo�te noire, � moins qu'ils ne puissent �tre devin�s ou forc�s avec succ�s. Cependant, ils sont faciles � identifier lors de l'ex�cution de tests en bo�te grise ou en bo�te blanche.

### Test des mots de passe par d�faut g�n�r�s par l'application

Si l'application g�n�re automatiquement des mots de passe pour les nouveaux comptes d'utilisateurs, ceux-ci peuvent �galement �tre pr�visibles. Afin de les tester, cr�ez plusieurs comptes sur l'application avec des d�tails similaires en m�me temps et comparez les mots de passe qui leur sont attribu�s.

Les mots de passe peuvent �tre bas�s sur :

- Une seule cha�ne statique partag�e entre les comptes.
- Une partie hach�e ou masqu�e des d�tails du compte, telle que `md5($username)`.
- Un algorithme bas� sur le temps.
- Un g�n�rateur de nombres pseudo-al�atoires faibles (PRNG).

Ce type de probl�me est souvent difficile � identifier du point de vue de la bo�te noire.

## Outils

- [Burp Intruder] (https://portswigger.net/burp/documentation/desktop/tools/intrus)
- [THC Hydra](https://github.com/vanhauser-thc/thc-hydra)
- [Nikto2](https://www.cirt.net/nikto2)

## R�f�rences

- [CIRT](https://cirt.net/passwords)
- [SecLists Default Passwords](https://github.com/danielmiessler/SecLists/tree/master/Passwords/Default-Credentials)
- [DefaultCreds-cheat-sheet](https://github.com/ihebski/DefaultCreds-cheat-sheet/blob/main/DefaultCreds-Cheat-Sheet.csv)
