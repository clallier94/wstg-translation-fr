# Test des références d'objets directes non sécurisées

|ID          |
|------------|
|WSTG-ATHZ-04|

## Sommaire

Des références directes d'objets non sécurisées (IDOR) se produisent lorsqu'une application fournit un accès direct à des objets en fonction d'une entrée fournie par l'utilisateur. En raison de cette vulnérabilité, les attaquants peuvent contourner l'autorisation et accéder directement aux ressources du système, par exemple les enregistrements ou les fichiers de la base de données.
Les références d'objet directes non sécurisées permettent aux attaquants de contourner l'autorisation et d'accéder directement aux ressources en modifiant la valeur d'un paramètre utilisé pour pointer directement vers un objet. Ces ressources peuvent être des entrées de base de données appartenant à d'autres utilisateurs, des fichiers du système, etc. Cela est dû au fait que l'application prend l'entrée fournie par l'utilisateur et l'utilise pour récupérer un objet sans effectuer suffisamment de vérifications d'autorisation.

## Objectifs des tests

- Identifiez les points où les références d'objets peuvent se produire.
- Évaluer les mesures de contrôle d'accès et si elles sont vulnérables à IDOR.

## Comment tester

Pour tester cette vulnérabilité, le testeur doit d'abord cartographier tous les emplacements de l'application où l'entrée de l'utilisateur est utilisée pour référencer directement les objets. Par exemple, les emplacements où l'entrée de l'utilisateur est utilisée pour accéder à une ligne de base de données, un fichier, des pages d'application et plus encore. Ensuite, le testeur doit modifier la valeur du paramètre utilisé pour référencer les objets et évaluer s'il est possible de récupérer des objets appartenant à d'autres utilisateurs ou de contourner l'autorisation.

La meilleure façon de tester les références d'objets directes serait d'avoir au moins deux utilisateurs (souvent plus) pour couvrir différents objets et fonctions possédés. Par exemple, deux utilisateurs ayant chacun accès à des objets différents (tels que des informations d'achat, des messages privés, etc.) et (le cas échéant) des utilisateurs avec des privilèges différents (par exemple des utilisateurs administrateurs) pour voir s'il existe des références directes aux fonctionnalités de l'application. En ayant plusieurs utilisateurs, le testeur gagne un temps de test précieux en devinant différents noms d'objets car il peut tenter d'accéder aux objets qui appartiennent à l'autre utilisateur.

Vous trouverez ci-dessous plusieurs scénarios typiques pour cette vulnérabilité et les méthodes à tester pour chacun :

### La valeur d'un paramètre est utilisée directement pour récupérer un enregistrement de base de données

Demande d'échantillon :

```text
http://foo.bar/somepage?invoice=12345
```

Dans ce cas, la valeur du paramètre *invoice* est utilisée comme index dans une table des factures de la base de données. L'application prend la valeur de ce paramètre et l'utilise dans une requête à la base de données. L'application renvoie ensuite les informations de facturation à l'utilisateur.

Comme la valeur de *facture* va directement dans la requête, en modifiant la valeur du paramètre il est possible de récupérer n'importe quel objet facture, quel que soit l'utilisateur auquel appartient la facture. Pour tester ce cas, le testeur doit obtenir l'identifiant d'une facture appartenant à un autre utilisateur de test (en s'assurant qu'il n'est pas censé afficher ces informations par logique métier d'application), puis vérifier s'il est possible d'accéder aux objets sans autorisation.

### La valeur d'un paramètre est utilisée directement pour effectuer une opération dans le système

Demande d'échantillon :

```text
http://foo.bar/changepassword?user=someuser
```

Dans ce cas, la valeur du paramètre `user` est utilisée pour dire à l'application pour quel utilisateur elle doit changer le mot de passe. Dans de nombreux cas, cette étape fera partie d'un assistant ou d'une opération en plusieurs étapes. Dans la première étape, l'application recevra une demande indiquant pour quel utilisateur le mot de passe doit être modifié, et à l'étape suivante, l'utilisateur fournira un nouveau mot de passe (sans demander l'actuel).

Le paramètre `user` est utilisé pour référencer directement l'objet de l'utilisateur pour lequel l'opération de changement de mot de passe sera effectuée. Pour tester ce cas, le testeur doit essayer de fournir un nom d'utilisateur de test différent de celui actuellement connecté et vérifier s'il est possible de modifier le mot de passe d'un autre utilisateur.

### La valeur d'un paramètre est utilisée directement pour récupérer une ressource du système de fichiers

Demande d'échantillon :

```text
http://foo.bar/showImage?img=img00011
```

Dans ce cas, la valeur du paramètre `file` est utilisée pour indiquer à l'application quel fichier l'utilisateur a l'intention de récupérer. En fournissant le nom ou l'identifiant d'un fichier différent (par exemple file=image00012.jpg) l'attaquant pourra récupérer des objets appartenant à d'autres utilisateurs.

Pour tester ce cas, le testeur doit obtenir une référence à laquelle l'utilisateur n'est pas censé pouvoir accéder et tenter d'y accéder en l'utilisant comme valeur du paramètre `file`. Remarque : Cette vulnérabilité est souvent exploitée en conjonction avec une vulnérabilité de traversée de répertoire/chemin (voir [Testing for Path Traversal](01-Testing_Directory_Traversal_File_Include.md))

### La valeur d'un paramètre est utilisée directement pour accéder à la fonctionnalité de l'application

Demande d'échantillon :

```text
http://foo.bar/accessPage?menuitem=12
```

Dans ce cas, la valeur du paramètre `menuitem` est utilisée pour indiquer à l'application à quel élément de menu (et donc à quelle fonctionnalité de l'application) l'utilisateur tente d'accéder. Supposons que l'utilisateur est censé être restreint et dispose donc de liens disponibles uniquement pour accéder aux éléments de menu 1, 2 et 3. En modifiant la valeur du paramètre `menuitem`, il est possible de contourner l'autorisation et d'accéder à des fonctionnalités supplémentaires de l'application. Pour tester ce cas, le testeur identifie un emplacement où la fonctionnalité de l'application est déterminée par référence à un élément de menu, mappe les valeurs des éléments de menu auxquels l'utilisateur de test donné peut accéder, puis tente d'autres éléments de menu.

Dans les exemples ci-dessus la modification d'un seul paramètre est suffisante. Cependant, la référence de l'objet peut parfois être divisée entre plusieurs paramètres et les tests doivent être ajustés en conséquence.

## Références

[Top 10 2013-A4-Insecure Direct Object References](https://owasp.org/www-project-top-ten/2017/Release_Notes)
