# Test des r�f�rences d'objets directes non s�curis�es

|ID          |
|------------|
|WSTG-ATHZ-04|

## Sommaire

Des r�f�rences directes d'objets non s�curis�es (IDOR) se produisent lorsqu'une application fournit un acc�s direct � des objets en fonction d'une entr�e fournie par l'utilisateur. En raison de cette vuln�rabilit�, les attaquants peuvent contourner l'autorisation et acc�der directement aux ressources du syst�me, par exemple les enregistrements ou les fichiers de la base de donn�es.
Les r�f�rences d'objet directes non s�curis�es permettent aux attaquants de contourner l'autorisation et d'acc�der directement aux ressources en modifiant la valeur d'un param�tre utilis� pour pointer directement vers un objet. Ces ressources peuvent �tre des entr�es de base de donn�es appartenant � d'autres utilisateurs, des fichiers du syst�me, etc. Cela est d� au fait que l'application prend l'entr�e fournie par l'utilisateur et l'utilise pour r�cup�rer un objet sans effectuer suffisamment de v�rifications d'autorisation.

## Objectifs des tests

- Identifiez les points o� les r�f�rences d'objets peuvent se produire.
- �valuer les mesures de contr�le d'acc�s et si elles sont vuln�rables � IDOR.

## Comment tester

Pour tester cette vuln�rabilit�, le testeur doit d'abord cartographier tous les emplacements de l'application o� l'entr�e de l'utilisateur est utilis�e pour r�f�rencer directement les objets. Par exemple, les emplacements o� l'entr�e de l'utilisateur est utilis�e pour acc�der � une ligne de base de donn�es, un fichier, des pages d'application et plus encore. Ensuite, le testeur doit modifier la valeur du param�tre utilis� pour r�f�rencer les objets et �valuer s'il est possible de r�cup�rer des objets appartenant � d'autres utilisateurs ou de contourner l'autorisation.

La meilleure fa�on de tester les r�f�rences d'objets directes serait d'avoir au moins deux utilisateurs (souvent plus) pour couvrir diff�rents objets et fonctions poss�d�s. Par exemple, deux utilisateurs ayant chacun acc�s � des objets diff�rents (tels que des informations d'achat, des messages priv�s, etc.) et (le cas �ch�ant) des utilisateurs avec des privil�ges diff�rents (par exemple des utilisateurs administrateurs) pour voir s'il existe des r�f�rences directes aux fonctionnalit�s de l'application. En ayant plusieurs utilisateurs, le testeur gagne un temps de test pr�cieux en devinant diff�rents noms d'objets car il peut tenter d'acc�der aux objets qui appartiennent � l'autre utilisateur.

Vous trouverez ci-dessous plusieurs sc�narios typiques pour cette vuln�rabilit� et les m�thodes � tester pour chacun�:

### La valeur d'un param�tre est utilis�e directement pour r�cup�rer un enregistrement de base de donn�es

Demande d'�chantillon :

```text
http://foo.bar/somepage?invoice=12345
```

Dans ce cas, la valeur du param�tre *invoice* est utilis�e comme index dans une table des factures de la base de donn�es. L'application prend la valeur de ce param�tre et l'utilise dans une requ�te � la base de donn�es. L'application renvoie ensuite les informations de facturation � l'utilisateur.

Comme la valeur de *facture* va directement dans la requ�te, en modifiant la valeur du param�tre il est possible de r�cup�rer n'importe quel objet facture, quel que soit l'utilisateur auquel appartient la facture. Pour tester ce cas, le testeur doit obtenir l'identifiant d'une facture appartenant � un autre utilisateur de test (en s'assurant qu'il n'est pas cens� afficher ces informations par logique m�tier d'application), puis v�rifier s'il est possible d'acc�der aux objets sans autorisation.

### La valeur d'un param�tre est utilis�e directement pour effectuer une op�ration dans le syst�me

Demande d'�chantillon :

```text
http://foo.bar/changepassword?user=someuser
```

Dans ce cas, la valeur du param�tre `user` est utilis�e pour dire � l'application pour quel utilisateur elle doit changer le mot de passe. Dans de nombreux cas, cette �tape fera partie d'un assistant ou d'une op�ration en plusieurs �tapes. Dans la premi�re �tape, l'application recevra une demande indiquant pour quel utilisateur le mot de passe doit �tre modifi�, et � l'�tape suivante, l'utilisateur fournira un nouveau mot de passe (sans demander l'actuel).

Le param�tre `user` est utilis� pour r�f�rencer directement l'objet de l'utilisateur pour lequel l'op�ration de changement de mot de passe sera effectu�e. Pour tester ce cas, le testeur doit essayer de fournir un nom d'utilisateur de test diff�rent de celui actuellement connect� et v�rifier s'il est possible de modifier le mot de passe d'un autre utilisateur.

### La valeur d'un param�tre est utilis�e directement pour r�cup�rer une ressource du syst�me de fichiers

Demande d'�chantillon :

```text
http://foo.bar/showImage?img=img00011
```

Dans ce cas, la valeur du param�tre `file` est utilis�e pour indiquer � l'application quel fichier l'utilisateur a l'intention de r�cup�rer. En fournissant le nom ou l'identifiant d'un fichier diff�rent (par exemple file=image00012.jpg) l'attaquant pourra r�cup�rer des objets appartenant � d'autres utilisateurs.

Pour tester ce cas, le testeur doit obtenir une r�f�rence � laquelle l'utilisateur n'est pas cens� pouvoir acc�der et tenter d'y acc�der en l'utilisant comme valeur du param�tre `file`. Remarque : Cette vuln�rabilit� est souvent exploit�e en conjonction avec une vuln�rabilit� de travers�e de r�pertoire/chemin (voir [Testing for Path Traversal](01-Testing_Directory_Traversal_File_Include.md))

### La valeur d'un param�tre est utilis�e directement pour acc�der � la fonctionnalit� de l'application

Demande d'�chantillon :

```text
http://foo.bar/accessPage?menuitem=12
```

Dans ce cas, la valeur du param�tre `menuitem` est utilis�e pour indiquer � l'application � quel �l�ment de menu (et donc � quelle fonctionnalit� de l'application) l'utilisateur tente d'acc�der. Supposons que l'utilisateur est cens� �tre restreint et dispose donc de liens disponibles uniquement pour acc�der aux �l�ments de menu 1, 2 et 3. En modifiant la valeur du param�tre `menuitem`, il est possible de contourner l'autorisation et d'acc�der � des fonctionnalit�s suppl�mentaires de l'application. Pour tester ce cas, le testeur identifie un emplacement o� la fonctionnalit� de l'application est d�termin�e par r�f�rence � un �l�ment de menu, mappe les valeurs des �l�ments de menu auxquels l'utilisateur de test donn� peut acc�der, puis tente d'autres �l�ments de menu.

Dans les exemples ci-dessus la modification d'un seul param�tre est suffisante. Cependant, la r�f�rence de l'objet peut parfois �tre divis�e entre plusieurs param�tres et les tests doivent �tre ajust�s en cons�quence.

## R�f�rences

[Top 10 2013-A4-Insecure Direct Object References](https://owasp.org/www-project-top-ten/2017/Release_Notes)
