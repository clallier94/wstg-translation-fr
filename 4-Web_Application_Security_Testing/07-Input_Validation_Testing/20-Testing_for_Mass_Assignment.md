# Test pour l'affectation en masse

|ID          |
|------------|
|WSTG-INPV-20|

## Sommaire

Les applications web modernes sont tr�s souvent bas�es sur des frameworks. Bon nombre de ces cadres d'application Web permettent la liaison automatique de l'entr�e de l'utilisateur (sous la forme de param�tres de requ�te HTTP) � des objets internes. Ceci est souvent appel� liaison automatique.
Cette fonctionnalit� peut parfois �tre exploit�e pour acc�der � des champs qui n'ont jamais �t� destin�s � �tre modifi�s de l'ext�rieur, ce qui entra�ne une �l�vation des privil�ges, une falsification des donn�es, un contournement des m�canismes de s�curit�, etc.
Dans ce cas, il existe une vuln�rabilit� d'affectation en masse.

Exemples de propri�t�s sensibles�:

- **Propri�t�s li�es aux autorisations**�: ne doivent �tre d�finies que par des utilisateurs privil�gi�s (par exemple, `is_admin`, `role`, `approved`).
- **Propri�t�s d�pendantes du processus**�: ne doivent �tre d�finies qu'en interne, apr�s la fin d'un processus (par exemple, `balance`, `status`, `email_verified`)
- **Propri�t�s internes**�: ne doivent �tre d�finies qu'en interne par l'application (par exemple, `created_at`, `updated_at`)

## Objectifs des tests

- Identifier les requ�tes qui modifient les objets
- �valuer s'il est possible de modifier des champs jamais destin�s � �tre modifi�s de l'ext�rieur

## Comment tester

Voici un exemple classique qui peut aider � illustrer le probl�me.

Supposons une application Web Java avec un objet `User` semblable � ce qui suit�:

```java
public class User {
   private String username;
   private String password;
   private String email;
   private boolean isAdmin;

   //Getters & Setters
}
```

Pour cr�er un nouvel `User`, l'application Web impl�mente la vue suivante�:

```html
<form action="/createUser" method="POST">
     <input name="username" type="text">
     <input name="password" type="text">
     <input name="email" text="text">
     <input type="submit" value="Create">
</form>
```

Le contr�leur qui g�re la demande de cr�ation (Spring fournit la liaison automatique avec le mod�le `User`):

```java
@RequestMapping(value = "/createUser", method = RequestMethod.POST)
public String createUser(User user) {
   userService.add(user);
   return "successPage";
}
```

Lorsque le formulaire est soumis, la requ�te suivante est g�n�r�e par le navigateur�:

```http
POST /createUser
[...]
username=bob&password=supersecretpassword&email=bob@domain.test
```

Cependant, en raison de la liaison automatique, un attaquant peut ajouter le param�tre `isAdmin` � la requ�te, que le contr�leur liera automatiquement au mod�le.

```http
POST /createUser
[...]
username=bob&password=supersecretpassword&email=bob@domain.test&isAdmin=true
```

L'utilisateur est ensuite cr�� avec la propri�t� `isAdmin` d�finie sur `true`, lui donnant des droits d'administration sur l'application.

### Test de la bo�te noire

#### D�tecter les gestionnaires

Afin de d�terminer quelle partie de l'application est vuln�rable � l'affectation en masse, �num�rez toutes les parties de l'application qui acceptent le contenu de l'utilisateur et peuvent potentiellement �tre mapp�es avec un mod�le. Cela inclut toutes les requ�tes HTTP (le plus souvent GET, POST et PUT) qui semblent autoriser les op�rations de cr�ation ou de mise � jour sur le back-end.
L'un des indicateurs les plus simples des affectations de masse potentielles est la pr�sence d'une syntaxe entre parenth�ses pour les noms de param�tres d'entr�e, comme par exemple�:

```html
<input name="user[name]" type="text">
```

Lorsque de tels mod�les sont rencontr�s, essayez d'ajouter une entr�e li�e � un attribut non existant (par exemple, `user[nonexistingattribute]`) et analysez la r�ponse/le comportement.
Si l'application n'impl�mente aucun contr�le (par exemple la liste des champs autoris�s), il est probable qu'elle r�ponde par une erreur (par exemple 500) due au fait que l'application ne trouve pas l'attribut associ� � l'objet. Plus int�ressant encore, ces erreurs facilitent parfois la d�couverte des noms d'attributs et des types de donn�es de valeur n�cessaires pour exploiter le probl�me, sans acc�s au code source.

#### Identifier les champs sensibles

Comme dans les tests en bo�te noire, le testeur n'a pas de visibilit� sur le code source, il est n�cessaire de trouver d'autres moyens afin de recueillir des informations sur les attributs associ�s aux objets.
Analysez les r�ponses re�ues par le backend, en particulier faites attention � :

- Code source de la page HTML
- Code JavaScript personnalis�
- R�ponses API

Par exemple, tr�s souvent, il est possible d'exploiter des handlers qui retournent des d�tails sur un objet afin de r�colter des indices sur les champs associ�s.
Supposons par exemple un gestionnaire qui renvoie le profil de l'utilisateur (par exemple `GET /profile`), cela peut inclure d'autres attributs li�s � l'utilisateur (dans cet exemple, l'attribut `isAdmin` semble particuli�rement int�ressant).

```json
{"_id":12345,"username":"bob","age":38,"email":"bob@domain.test","isAdmin":false}
```

Essayez ensuite d'exploiter les gestionnaires qui permettent la modification ou la cr�ation d'utilisateurs, en ajoutant l'attribut `isAdmin` configur� � `true`.

Une autre approche consiste � utiliser des listes de mots afin d'essayer d'�num�rer tous les attributs potentiels. L'�num�ration peut ensuite �tre automatis�e (par exemple via wfuzz, Burp Intruder, ZAP fuzzer, etc.). L'outil sqlmap inclut une liste de mots [common-columns.txt](https://github.com/sqlmapproject/sqlmap/blob/master/data/txt/common-columns.txt) qui peut �tre utile pour identifier les attributs potentiellement sensibles.
Voici un petit exemple de noms d'attributs int�ressants courants�:

- `is_admin`
- `is_administrator`
- `isAdmin`
- `isAdministrator`
- `admin`
- `administrator`
- `role`

Lorsque plusieurs r�les sont disponibles, essayez de comparer les demandes faites par diff�rents niveaux d'utilisateurs (faites particuli�rement attention aux r�les privil�gi�s). Par exemple, si des param�tres suppl�mentaires sont inclus dans les requ�tes effectu�es par un utilisateur administratif, essayez-les en tant qu'utilisateur faiblement privil�gi�/anonyme.

#### V�rifier l'impact

L'impact d'une affectation de masse peut varier en fonction du contexte. Par cons�quent, pour chaque entr�e de test tent�e dans la phase pr�c�dente, analysez le r�sultat et d�terminez s'il repr�sente une vuln�rabilit� ayant un impact r�aliste sur la s�curit� de l'application Web.
Par exemple, la modification de l'identifiant d'un objet peut entra�ner un d�ni de service applicatif ou une �l�vation de privil�ges. Un autre exemple est li� � la possibilit� de modifier le r�le/statut de l'utilisateur (par exemple, `role` ou `isAdmin`) entra�nant une �l�vation verticale des privil�ges.

###�Test de la bo�te grise

Lorsque l'analyse est effectu�e avec une approche de test en bo�te grise, il est possible de suivre la m�me m�thodologie pour v�rifier le probl�me. Cependant, la plus grande connaissance de l'application permet d'identifier plus facilement les frameworks et les gestionnaires sujets � une vuln�rabilit� d'affectation de masse.
En particulier, lorsque le code source est disponible, il est possible de rechercher plus facilement et plus pr�cis�ment les vecteurs d'entr�e. Lors d'une r�vision du code source, utilisez des outils simples (tels que la commande grep) pour rechercher un ou plusieurs mod�les courants dans le code de l'application.
L'acc�s au sch�ma de la BD ou au code source permet �galement d'identifier facilement les champs sensibles.

####Java

Spring MVC permet de lier automatiquement l'entr�e de l'utilisateur � l'objet. Identifiez les contr�leurs qui g�rent les demandes de changement d'�tat (par exemple, trouvez les occurrences de `@RequestMapping`), puis v�rifiez si des contr�les sont en place (� la fois sur le contr�leur ou sur les mod�les concern�s). Les limitations � l'exploitation de l'assignation de masse peuvent �tre, par exemple, sous la forme de :

- liste des champs pouvant �tre li�s via la m�thode `setAllowedFields` de la classe `DataBinder` (par exemple `binder.setAllowedFields(["username","password","email"])`)
- liste des champs non contraignants via la m�thode `setDisallowedFields` de la classe `DataBinder` (par exemple `binder.setDisallowedFields(["isAdmin"])`)

Il est �galement conseill� de faire attention � l'utilisation de l'annotation `@ModelAttribute` qui permet de sp�cifier un nom/cl� diff�rent.

####PHP

Laravel Eloquent ORM fournit une m�thode "create" qui permet l'attribution automatique d'attributs. Cependant, les derni�res versions d'Eloquent ORM fournissent une protection par d�faut contre les vuln�rabilit�s d'attribution de masse n�cessitant de sp�cifier explicitement les attributs autoris�s qui peuvent �tre attribu�s automatiquement, via le tableau `$fillable`, ou les attributs qui doivent �tre prot�g�s (non-bindable), via le Tableau `$gard�`. Par cons�quent, en analysant les mod�les (classes qui �tendent la classe `Model`), il est possible d'identifier les attributs autoris�s ou refus�s et donc de signaler les vuln�rabilit�s potentielles.

#### .RAPPORTER

La liaison de mod�le dans ASP.NET lie automatiquement les entr�es utilisateur aux propri�t�s de l'objet. Cela fonctionne �galement avec les types complexes et convertit automatiquement les donn�es d'entr�e en propri�t�s si les noms des propri�t�s correspondent � l'entr�e.
Identifiez les contr�leurs, puis v�rifiez si des contr�les sont en place (� la fois � l'int�rieur du contr�leur ou dans les mod�les concern�s). Les limitations � l'exploitation de l'assignation de masse peuvent �tre, par exemple, sous la forme de :

- champs d�clar�s en `ReadOnly`
- liste des champs pouvant �tre li�s via l'attribut `Bind` (par exemple `[Bind(Include = "FirstName, LastName")] Student std`), via `includeProperties` (par exemple `includeProperties: new[] { "FirstName, LastName" }` ) ou via `TryUpdateModel`
- liste des champs non contraignants via l'attribut `Bind` (par exemple `[Bind(Exclude = "Status")] Student std`) ou via `excludeProperties` (par exemple `excludeProperties: new[] { "Status" }`)

## Correction

Utilisez les fonctionnalit�s int�gr�es, fournies par les frameworks, pour d�finir des champs pouvant �tre li�s et non li�s. Une approche bas�e sur les champs autoris�s (bindable), dans laquelle seules les propri�t�s qui doivent �tre mises � jour par l'utilisateur sont explicitement d�finies, est pr�f�rable.
Une approche architecturale pour �viter le probl�me consiste � utiliser le mod�le *Data Transfer Object* (DTO) afin d'�viter la liaison directe. Le DTO ne doit inclure que les champs cens�s �tre modifiables par l'utilisateur.

## R�f�rences

- [OWASP�: S�curit� des API](https://github.com/OWASP/API-Security/blob/master/2019/en/src/0xa6-mass-assignment.md)
- [OWASP�: s�rie de feuilles de triche] (https://cheatsheetseries.owasp.org/cheatsheets/Mass_Assignment_Cheat_Sheet.html)
- [CWE-915�: Modification mal contr�l�e d'attributs d'objet d�termin�s dynamiquement] (https://cwe.mitre.org/data/definitions/915.html)
