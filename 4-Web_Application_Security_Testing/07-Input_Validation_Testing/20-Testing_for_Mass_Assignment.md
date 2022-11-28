# Test pour l'affectation en masse

|ID          |
|------------|
|WSTG-INPV-20|

## Sommaire

Les applications web modernes sont très souvent basées sur des frameworks. Bon nombre de ces cadres d'application Web permettent la liaison automatique de l'entrée de l'utilisateur (sous la forme de paramètres de requête HTTP) à des objets internes. Ceci est souvent appelé liaison automatique.
Cette fonctionnalité peut parfois être exploitée pour accéder à des champs qui n'ont jamais été destinés à être modifiés de l'extérieur, ce qui entraîne une élévation des privilèges, une falsification des données, un contournement des mécanismes de sécurité, etc.
Dans ce cas, il existe une vulnérabilité d'affectation en masse.

Exemples de propriétés sensibles :

- **Propriétés liées aux autorisations** : ne doivent être définies que par des utilisateurs privilégiés (par exemple, `is_admin`, `role`, `approved`).
- **Propriétés dépendantes du processus** : ne doivent être définies qu'en interne, après la fin d'un processus (par exemple, `balance`, `status`, `email_verified`)
- **Propriétés internes** : ne doivent être définies qu'en interne par l'application (par exemple, `created_at`, `updated_at`)

## Objectifs des tests

- Identifier les requêtes qui modifient les objets
- Évaluer s'il est possible de modifier des champs jamais destinés à être modifiés de l'extérieur

## Comment tester

Voici un exemple classique qui peut aider à illustrer le problème.

Supposons une application Web Java avec un objet `User` semblable à ce qui suit :

```java
public class User {
   private String username;
   private String password;
   private String email;
   private boolean isAdmin;

   //Getters & Setters
}
```

Pour créer un nouvel `User`, l'application Web implémente la vue suivante :

```html
<form action="/createUser" method="POST">
     <input name="username" type="text">
     <input name="password" type="text">
     <input name="email" text="text">
     <input type="submit" value="Create">
</form>
```

Le contrôleur qui gère la demande de création (Spring fournit la liaison automatique avec le modèle `User`):

```java
@RequestMapping(value = "/createUser", method = RequestMethod.POST)
public String createUser(User user) {
   userService.add(user);
   return "successPage";
}
```

Lorsque le formulaire est soumis, la requête suivante est générée par le navigateur :

```http
POST /createUser
[...]
username=bob&password=supersecretpassword&email=bob@domain.test
```

Cependant, en raison de la liaison automatique, un attaquant peut ajouter le paramètre `isAdmin` à la requête, que le contrôleur liera automatiquement au modèle.

```http
POST /createUser
[...]
username=bob&password=supersecretpassword&email=bob@domain.test&isAdmin=true
```

L'utilisateur est ensuite créé avec la propriété `isAdmin` définie sur `true`, lui donnant des droits d'administration sur l'application.

### Test de la boîte noire

#### Détecter les gestionnaires

Afin de déterminer quelle partie de l'application est vulnérable à l'affectation en masse, énumérez toutes les parties de l'application qui acceptent le contenu de l'utilisateur et peuvent potentiellement être mappées avec un modèle. Cela inclut toutes les requêtes HTTP (le plus souvent GET, POST et PUT) qui semblent autoriser les opérations de création ou de mise à jour sur le back-end.
L'un des indicateurs les plus simples des affectations de masse potentielles est la présence d'une syntaxe entre parenthèses pour les noms de paramètres d'entrée, comme par exemple :

```html
<input name="user[name]" type="text">
```

Lorsque de tels modèles sont rencontrés, essayez d'ajouter une entrée liée à un attribut non existant (par exemple, `user[nonexistingattribute]`) et analysez la réponse/le comportement.
Si l'application n'implémente aucun contrôle (par exemple la liste des champs autorisés), il est probable qu'elle réponde par une erreur (par exemple 500) due au fait que l'application ne trouve pas l'attribut associé à l'objet. Plus intéressant encore, ces erreurs facilitent parfois la découverte des noms d'attributs et des types de données de valeur nécessaires pour exploiter le problème, sans accès au code source.

#### Identifier les champs sensibles

Comme dans les tests en boîte noire, le testeur n'a pas de visibilité sur le code source, il est nécessaire de trouver d'autres moyens afin de recueillir des informations sur les attributs associés aux objets.
Analysez les réponses reçues par le backend, en particulier faites attention à :

- Code source de la page HTML
- Code JavaScript personnalisé
- Réponses API

Par exemple, très souvent, il est possible d'exploiter des handlers qui retournent des détails sur un objet afin de récolter des indices sur les champs associés.
Supposons par exemple un gestionnaire qui renvoie le profil de l'utilisateur (par exemple `GET /profile`), cela peut inclure d'autres attributs liés à l'utilisateur (dans cet exemple, l'attribut `isAdmin` semble particulièrement intéressant).

```json
{"_id":12345,"username":"bob","age":38,"email":"bob@domain.test","isAdmin":false}
```

Essayez ensuite d'exploiter les gestionnaires qui permettent la modification ou la création d'utilisateurs, en ajoutant l'attribut `isAdmin` configuré à `true`.

Une autre approche consiste à utiliser des listes de mots afin d'essayer d'énumérer tous les attributs potentiels. L'énumération peut ensuite être automatisée (par exemple via wfuzz, Burp Intruder, ZAP fuzzer, etc.). L'outil sqlmap inclut une liste de mots [common-columns.txt](https://github.com/sqlmapproject/sqlmap/blob/master/data/txt/common-columns.txt) qui peut être utile pour identifier les attributs potentiellement sensibles.
Voici un petit exemple de noms d'attributs intéressants courants :

- `is_admin`
- `is_administrator`
- `isAdmin`
- `isAdministrator`
- `admin`
- `administrator`
- `role`

Lorsque plusieurs rôles sont disponibles, essayez de comparer les demandes faites par différents niveaux d'utilisateurs (faites particulièrement attention aux rôles privilégiés). Par exemple, si des paramètres supplémentaires sont inclus dans les requêtes effectuées par un utilisateur administratif, essayez-les en tant qu'utilisateur faiblement privilégié/anonyme.

#### Vérifier l'impact

L'impact d'une affectation de masse peut varier en fonction du contexte. Par conséquent, pour chaque entrée de test tentée dans la phase précédente, analysez le résultat et déterminez s'il représente une vulnérabilité ayant un impact réaliste sur la sécurité de l'application Web.
Par exemple, la modification de l'identifiant d'un objet peut entraîner un déni de service applicatif ou une élévation de privilèges. Un autre exemple est lié à la possibilité de modifier le rôle/statut de l'utilisateur (par exemple, `role` ou `isAdmin`) entraînant une élévation verticale des privilèges.

### Test de la boîte grise

Lorsque l'analyse est effectuée avec une approche de test en boîte grise, il est possible de suivre la même méthodologie pour vérifier le problème. Cependant, la plus grande connaissance de l'application permet d'identifier plus facilement les frameworks et les gestionnaires sujets à une vulnérabilité d'affectation de masse.
En particulier, lorsque le code source est disponible, il est possible de rechercher plus facilement et plus précisément les vecteurs d'entrée. Lors d'une révision du code source, utilisez des outils simples (tels que la commande grep) pour rechercher un ou plusieurs modèles courants dans le code de l'application.
L'accès au schéma de la BD ou au code source permet également d'identifier facilement les champs sensibles.

####Java

Spring MVC permet de lier automatiquement l'entrée de l'utilisateur à l'objet. Identifiez les contrôleurs qui gèrent les demandes de changement d'état (par exemple, trouvez les occurrences de `@RequestMapping`), puis vérifiez si des contrôles sont en place (à la fois sur le contrôleur ou sur les modèles concernés). Les limitations à l'exploitation de l'assignation de masse peuvent être, par exemple, sous la forme de :

- liste des champs pouvant être liés via la méthode `setAllowedFields` de la classe `DataBinder` (par exemple `binder.setAllowedFields(["username","password","email"])`)
- liste des champs non contraignants via la méthode `setDisallowedFields` de la classe `DataBinder` (par exemple `binder.setDisallowedFields(["isAdmin"])`)

Il est également conseillé de faire attention à l'utilisation de l'annotation `@ModelAttribute` qui permet de spécifier un nom/clé différent.

####PHP

Laravel Eloquent ORM fournit une méthode "create" qui permet l'attribution automatique d'attributs. Cependant, les dernières versions d'Eloquent ORM fournissent une protection par défaut contre les vulnérabilités d'attribution de masse nécessitant de spécifier explicitement les attributs autorisés qui peuvent être attribués automatiquement, via le tableau `$fillable`, ou les attributs qui doivent être protégés (non-bindable), via le Tableau `$gardé`. Par conséquent, en analysant les modèles (classes qui étendent la classe `Model`), il est possible d'identifier les attributs autorisés ou refusés et donc de signaler les vulnérabilités potentielles.

#### .RAPPORTER

La liaison de modèle dans ASP.NET lie automatiquement les entrées utilisateur aux propriétés de l'objet. Cela fonctionne également avec les types complexes et convertit automatiquement les données d'entrée en propriétés si les noms des propriétés correspondent à l'entrée.
Identifiez les contrôleurs, puis vérifiez si des contrôles sont en place (à la fois à l'intérieur du contrôleur ou dans les modèles concernés). Les limitations à l'exploitation de l'assignation de masse peuvent être, par exemple, sous la forme de :

- champs déclarés en `ReadOnly`
- liste des champs pouvant être liés via l'attribut `Bind` (par exemple `[Bind(Include = "FirstName, LastName")] Student std`), via `includeProperties` (par exemple `includeProperties: new[] { "FirstName, LastName" }` ) ou via `TryUpdateModel`
- liste des champs non contraignants via l'attribut `Bind` (par exemple `[Bind(Exclude = "Status")] Student std`) ou via `excludeProperties` (par exemple `excludeProperties: new[] { "Status" }`)

## Correction

Utilisez les fonctionnalités intégrées, fournies par les frameworks, pour définir des champs pouvant être liés et non liés. Une approche basée sur les champs autorisés (bindable), dans laquelle seules les propriétés qui doivent être mises à jour par l'utilisateur sont explicitement définies, est préférable.
Une approche architecturale pour éviter le problème consiste à utiliser le modèle *Data Transfer Object* (DTO) afin d'éviter la liaison directe. Le DTO ne doit inclure que les champs censés être modifiables par l'utilisateur.

## Références

- [OWASP : Sécurité des API](https://github.com/OWASP/API-Security/blob/master/2019/en/src/0xa6-mass-assignment.md)
- [OWASP : série de feuilles de triche] (https://cheatsheetseries.owasp.org/cheatsheets/Mass_Assignment_Cheat_Sheet.html)
- [CWE-915 : Modification mal contrôlée d'attributs d'objet déterminés dynamiquement] (https://cwe.mitre.org/data/definitions/915.html)
