# Test pour l'injection XPath

|ID          |
|------------|
|WSTG-INPV-09|

## Sommaire

XPath est un langage qui a été conçu et développé principalement pour traiter des parties d'un document XML. Dans les tests d'injection XPath, nous testons s'il est possible d'injecter la syntaxe XPath dans une requête interprétée par l'application, permettant à un attaquant d'exécuter des requêtes XPath contrôlées par l'utilisateur. Lorsqu'elle est exploitée avec succès, cette vulnérabilité peut permettre à un attaquant de contourner les mécanismes d'authentification ou d'accéder à des informations sans autorisation appropriée.

Les applications Web utilisent fortement les bases de données pour stocker et accéder aux données dont elles ont besoin pour leurs opérations. Historiquement, les bases de données relationnelles ont été de loin la technologie la plus courante pour le stockage de données, mais, ces dernières années, nous assistons à une popularité croissante des bases de données qui organisent les données à l'aide du langage XML. Tout comme les bases de données relationnelles sont accessibles via le langage SQL, les bases de données XML utilisent XPath comme langage de requête standard.

Étant donné que, d'un point de vue conceptuel, XPath est très similaire à SQL dans son objectif et ses applications, un résultat intéressant est que les attaques par injection XPath suivent la même logique que [SQL Injection](https://owasp.org/www-community /attacks/SQL_Injection). Sous certains aspects, XPath est encore plus puissant que le SQL standard, car toute sa puissance est déjà présente dans ses spécifications, alors qu'un grand nombre des techniques pouvant être utilisées dans une attaque par injection SQL dépendent des caractéristiques du dialecte SQL utilisé par la base de données cible. Cela signifie que les attaques par injection XPath peuvent être beaucoup plus adaptables et omniprésentes. Un autre avantage d'une attaque par injection XPath est que, contrairement à SQL, aucune ACL n'est appliquée, car notre requête peut accéder à chaque partie du document XML.

## Objectifs des tests

- Identifier les points d'injection XPATH.

## Comment tester

Le [modèle d'attaque XPath a été publié pour la première fois par Amit Klein](http://dl.packetstormsecurity.net/papers/bypass/Blind_XPath_Injection_20040518.pdf) et est très similaire à l'injection SQL habituelle. Afin d'avoir une première idée du problème, imaginons une page de connexion qui gère l'authentification à une application dans laquelle l'utilisateur doit entrer son nom d'utilisateur et son mot de passe. Supposons que notre base de données est représentée par le fichier XML suivant :

```xml
<?xml version="1.0" encoding="ISO-8859-1"?>
<users>
    <user>
        <username>gandalf</username>
        <password>!c3</password>
        <account>admin</account>
    </user>
    <user>
        <username>Stefan0</username>
        <password>w1s3c</password>
        <account>guest</account>
    </user>
    <user>
        <username>tony</username>
        <password>Un6R34kb!e</password>
        <account>guest</account>
    </user>
</users>
```

Une requête XPath qui renvoie le compte dont le nom d'utilisateur est `gandalf` et le mot de passe est `!c3` serait la suivante :

`string(//user[username/text()='gandalf' and password/text()='!c3']/account/text())`

Si l'application ne filtre pas correctement les entrées de l'utilisateur, le testeur pourra injecter du code XPath et interférer avec le résultat de la requête. Par exemple, le testeur peut saisir les valeurs suivantes :

```text
Username: ' or '1' = '1
Password: ' or '1' = '1
```

Cela semble assez familier, n'est-ce pas? En utilisant ces paramètres, la requête devient :

`string(//user[username/text()='' or '1' = '1' and password/text()='' or '1' = '1']/account/text())`

Comme dans une attaque par injection SQL courante, nous avons créé une requête qui est toujours évaluée à vrai, ce qui signifie que l'application authentifiera l'utilisateur même si un nom d'utilisateur ou un mot de passe n'a pas été fourni. Et comme dans une attaque par injection SQL courante, avec l'injection XPath, la première étape consiste à insérer un guillemet simple (`'`) dans le champ à tester, introduisant une erreur de syntaxe dans la requête, et à vérifier si l'application renvoie un Message d'erreur.

S'il n'y a aucune connaissance des détails internes des données XML et si l'application ne fournit pas de messages d'erreur utiles qui nous aident à reconstruire sa logique interne, il est possible d'effectuer une [Blind XPath Injection](https://owasp.org/www -community/attacks/Blind_XPath_Injection) attaque, dont le but est de reconstruire toute la structure des données. La technique est similaire à l'injection SQL basée sur l'inférence, car l'approche consiste à injecter du code qui crée une requête qui renvoie un bit d'information. [Blind XPath Injection](https://owasp.org/www-community/attacks/Blind_XPath_Injection) est expliqué plus en détail par Amit Klein dans l'article référencé.

## Références

### Papiers blanc

- [Amit Klein : "Blind XPath Injection"](http://dl.packetstormsecurity.net/papers/bypass/Blind_XPath_Injection_20040518.pdf)
- [Spécifications XPath 1.0](https://www.w3.org/TR/1999/REC-xpath-19991116/)
