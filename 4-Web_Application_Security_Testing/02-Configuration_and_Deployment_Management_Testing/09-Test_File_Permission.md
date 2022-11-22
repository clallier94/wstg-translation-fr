# Autorisation de fichier de test

|ID          |
|------------|
|WSTG-CONF-09|

## Sommaire

Lorsqu'une ressource se voit attribuer un paramètre d'autorisations qui donne accès à un éventail d'acteurs plus large que nécessaire, cela peut entraîner l'exposition d'informations sensibles ou la modification de cette ressource par des parties non intentionnelles. Ceci est particulièrement dangereux lorsque la ressource est liée à la configuration du programme, à l'exécution ou à des données utilisateur sensibles.

Un exemple clair est un fichier d'exécution qui est exécutable par des utilisateurs non autorisés. Pour un autre exemple, les informations de compte ou une valeur de jeton pour accéder à une API - de plus en plus présentes dans les services Web modernes ou les microservices - peuvent être stockées dans un fichier de configuration dont les autorisations sont définies sur lisibles par le monde depuis l'installation par défaut. Ces données sensibles peuvent être exposées par des acteurs internes malveillants de l'hôte ou par un attaquant distant qui a compromis le service avec d'autres vulnérabilités mais n'a obtenu qu'un privilège d'utilisateur normal.

## Objectifs des tests

- Examinez et identifiez toutes les autorisations de fichiers malveillants.

## Comment tester

Sous Linux, utilisez la commande `ls` pour vérifier les autorisations de fichier. Alternativement, `namei` peut également être utilisé pour répertorier récursivement les autorisations de fichiers.

`$ nomi -l /CheminÀVérifier/`

Les fichiers et répertoires qui nécessitent un test d'autorisation de fichier incluent, sans s'y limiter :

- Fichiers/répertoire Web
- Fichiers/répertoire de configuration
- Fichiers sensibles (données cryptées, mot de passe, clé)/répertoire
- Fichiers journaux (journaux de sécurité, journaux d'opération, journaux d'administration)/répertoire
- Exécutables (scripts, EXE, JAR, classe, PHP, ASP)/répertoire
- Fichiers/répertoire de la base de données
- Fichiers/répertoire temporaires
- Télécharger des fichiers/répertoire

## Correction

Définissez correctement les autorisations des fichiers et des répertoires afin que les utilisateurs non autorisés ne puissent pas accéder inutilement aux ressources critiques.

## Outils

- [Windows AccessEnum](https://technet.microsoft.com/en-us/sysinternals/accessenum)
- [Windows AccessChk](https://technet.microsoft.com/en-us/sysinternals/accesschk)
- [nom Linuxi](https://linux.die.net/man/1/namei)

## Références

- [CWE-732 : Attribution d'autorisation incorrecte pour une ressource critique] (https://cwe.mitre.org/data/definitions/732.html)