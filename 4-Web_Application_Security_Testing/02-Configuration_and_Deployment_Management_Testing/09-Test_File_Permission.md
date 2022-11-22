# Autorisation de fichier de test

|ID          |
|------------|
|WSTG-CONF-09|

## Sommaire

Lorsqu'une ressource se voit attribuer un param�tre d'autorisations qui donne acc�s � un �ventail d'acteurs plus large que n�cessaire, cela peut entra�ner l'exposition d'informations sensibles ou la modification de cette ressource par des parties non intentionnelles. Ceci est particuli�rement dangereux lorsque la ressource est li�e � la configuration du programme, � l'ex�cution ou � des donn�es utilisateur sensibles.

Un exemple clair est un fichier d'ex�cution qui est ex�cutable par des utilisateurs non autoris�s. Pour un autre exemple, les informations de compte ou une valeur de jeton pour acc�der � une API - de plus en plus pr�sentes dans les services Web modernes ou les microservices - peuvent �tre stock�es dans un fichier de configuration dont les autorisations sont d�finies sur lisibles par le monde depuis l'installation par d�faut. Ces donn�es sensibles peuvent �tre expos�es par des acteurs internes malveillants de l'h�te ou par un attaquant distant qui a compromis le service avec d'autres vuln�rabilit�s mais n'a obtenu qu'un privil�ge d'utilisateur normal.

## Objectifs des tests

- Examinez et identifiez toutes les autorisations de fichiers malveillants.

## Comment tester

Sous Linux, utilisez la commande `ls` pour v�rifier les autorisations de fichier. Alternativement, `namei` peut �galement �tre utilis� pour r�pertorier r�cursivement les autorisations de fichiers.

`$ nomi -l /Chemin�V�rifier/`

Les fichiers et r�pertoires qui n�cessitent un test d'autorisation de fichier incluent, sans s'y limiter�:

- Fichiers/r�pertoire Web
- Fichiers/r�pertoire de configuration
- Fichiers sensibles (donn�es crypt�es, mot de passe, cl�)/r�pertoire
- Fichiers journaux (journaux de s�curit�, journaux d'op�ration, journaux d'administration)/r�pertoire
- Ex�cutables (scripts, EXE, JAR, classe, PHP, ASP)/r�pertoire
- Fichiers/r�pertoire de la base de donn�es
- Fichiers/r�pertoire temporaires
- T�l�charger des fichiers/r�pertoire

## Correction

D�finissez correctement les autorisations des fichiers et des r�pertoires afin que les utilisateurs non autoris�s ne puissent pas acc�der inutilement aux ressources critiques.

## Outils

- [Windows AccessEnum](https://technet.microsoft.com/en-us/sysinternals/accessenum)
- [Windows AccessChk](https://technet.microsoft.com/en-us/sysinternals/accesschk)
- [nom Linuxi](https://linux.die.net/man/1/namei)

## R�f�rences

- [CWE-732�: Attribution d'autorisation incorrecte pour une ressource critique] (https://cwe.mitre.org/data/definitions/732.html)