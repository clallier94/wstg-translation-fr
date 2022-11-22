# Identifier les points d'entrée de l'application

|ID          |
|------------|
|WSTG-INFO-06|

## Sommaire

L'énumération de l'application et de sa surface d'attaque est un précurseur clé avant de pouvoir entreprendre tout test approfondi, car cela permet au testeur d'identifier les zones de faiblesse probables. Cette section vise à aider à identifier et à cartographier les zones de l'application qui doivent être étudiées une fois le dénombrement et la cartographie terminés.

## Objectifs des tests

- Identifier les points d'entrée et d'injection possibles grâce à l'analyse des demandes et des réponses.

## Comment tester

Avant le début de tout test, le testeur doit toujours avoir une bonne compréhension de l'application et de la manière dont l'utilisateur et le navigateur communiquent avec elle. Lorsque le testeur parcourt l'application, il doit prêter attention à toutes les requêtes HTTP ainsi qu'à chaque paramètre et champ de formulaire transmis à l'application. Ils doivent porter une attention particulière au moment où les requêtes GET sont utilisées et au moment où les requêtes POST sont utilisées pour transmettre des paramètres à l'application. En outre, ils doivent également faire attention au moment où d'autres méthodes pour les services RESTful sont utilisées.

Notez que pour voir les paramètres envoyés dans le corps des requêtes telles qu'une requête POST, le testeur peut vouloir utiliser un outil tel qu'un proxy d'interception (Voir [tools](#tools)). Dans la requête POST, le testeur doit également prendre note de tous les champs de formulaire masqués qui sont transmis à l'application, car ils contiennent généralement des informations sensibles, telles que des informations sur l'état, la quantité d'articles, le prix des articles, que le développeur n'a jamais destiné à être vu ou modifié par quiconque.

D'après l'expérience de l'auteur, il a été très utile d'utiliser un proxy d'interception et une feuille de calcul pour cette étape de test. Le proxy gardera une trace de chaque demande et réponse entre le testeur et l'application au fur et à mesure qu'ils l'explorent. De plus, à ce stade, les testeurs interceptent généralement chaque demande et réponse afin qu'ils puissent voir exactement chaque en-tête, paramètre, etc. qui est transmis à l'application et ce qui est renvoyé. Cela peut parfois être assez fastidieux, surtout sur les grands sites interactifs (pensez à une application bancaire). Cependant, l'expérience montrera ce qu'il faut rechercher et cette phase peut être considérablement réduite.

Au fur et à mesure que le testeur parcourt l'application, il doit prendre note de tous les paramètres intéressants dans l'URL, les en-têtes personnalisés ou le corps des demandes/réponses, et les enregistrer dans une feuille de calcul. La feuille de calcul doit inclure la page demandée (il peut être bon d'ajouter également le numéro de requête du proxy, pour référence future), les paramètres intéressants, le type de requête (GET, POST, etc.), si l'accès est authentifié/non authentifié, si TLS est utilisé, s'il fait partie d'un processus en plusieurs étapes, si WebSockets est utilisé et toute autre note pertinente. Une fois qu'ils ont cartographié tous les domaines de l'application, ils peuvent parcourir l'application et tester chacun des domaines qu'ils ont identifiés et prendre des notes sur ce qui a fonctionné et ce qui n'a pas fonctionné. Le reste de ce guide identifiera comment tester chacun de ces domaines d'intérêt, mais cette section doit être effectuée avant que l'un des tests réels puisse commencer.

Vous trouverez ci-dessous quelques points d'intérêt pour toutes les demandes et réponses. Dans la section des requêtes, concentrez-vous sur les méthodes GET et POST, car elles apparaissent comme la majorité des requêtes. Notez que d'autres méthodes, telles que PUT et DELETE, peuvent être utilisées. Souvent, ces demandes plus rares, si elles sont autorisées, peuvent exposer des vulnérabilités. Il y a une section spéciale dans ce guide dédiée au test de ces méthodes HTTP.

### Demandes

- Identifiez où les GET sont utilisés et où les POST sont utilisés.
- Identifiez tous les paramètres utilisés dans une requête POST (ceux-ci sont dans le corps de la requête).
- Dans la requête POST, faites particulièrement attention aux paramètres cachés. Lorsqu'un POST est envoyé, tous les champs du formulaire (y compris les paramètres masqués) seront envoyés dans le corps du message HTTP à l'application. Ceux-ci ne sont généralement pas visibles à moins qu'un proxy ou une vue du code source HTML ne soit utilisé. De plus, la page suivante affichée, ses données et le niveau d'accès peuvent tous être différents en fonction de la valeur du ou des paramètres masqués.
- Identifiez tous les paramètres utilisés dans une requête GET (c'est-à-dire l'URL), en particulier la chaîne de requête (généralement après un signe ?).
- Identifiez tous les paramètres de la chaîne de requête. Ceux-ci sont généralement dans un format de paire, tel que `foo=bar`. Notez également que de nombreux paramètres peuvent être dans une chaîne de requête tels que séparés par un `&`, `\~`, `:`, ou tout autre caractère spécial ou encodage.
- Une remarque spéciale lorsqu'il s'agit d'identifier plusieurs paramètres dans une chaîne ou dans une requête POST est que certains ou tous les paramètres seront nécessaires pour exécuter les attaques. Le testeur doit identifier tous les paramètres (même codés ou cryptés) et identifier ceux qui sont traités par l'application. Les sections ultérieures du guide identifieront comment tester ces paramètres. À ce stade, assurez-vous simplement que chacun d'eux est identifié.
- Faites également attention à tous les en-têtes de type supplémentaires ou personnalisés qui ne sont généralement pas visibles (tels que `debug: false`).

### Réponses

- Identifiez où les nouveaux cookies sont définis (en-tête `Set-Cookie`), modifiés ou ajoutés.
- Identifiez où se trouvent des redirections (code d'état HTTP 3xx), 400 codes d'état, en particulier 403 Interdit, et 500 erreurs de serveur internes lors de réponses normales (c'est-à-dire des demandes non modifiées).
- Notez également où les en-têtes intéressants sont utilisés. Par exemple, `Server : BIG-IP` indique que le site est à charge équilibrée. Ainsi, si un site est à équilibrage de charge et qu'un serveur est mal configuré, le testeur peut être amené à effectuer plusieurs requêtes pour accéder au serveur vulnérable, selon le type d'équilibrage de charge utilisé.

### Détecteur de surface d'attaque OWASP

L'outil Attack Surface Detector (ASD) examine le code source et découvre les points de terminaison d'une application Web, les paramètres que ces points de terminaison acceptent et le type de données de ces paramètres. Cela inclut les points de terminaison non liés qu'un spider ne pourra pas trouver, ou des paramètres facultatifs totalement inutilisés dans le code côté client. Il a également la capacité de calculer les changements de surface d'attaque entre deux versions d'une application.

Le détecteur de surface d'attaque est disponible en tant que plug-in pour ZAP et Burp Suite, et un outil de ligne de commande est également disponible. L'outil de ligne de commande exporte la surface d'attaque sous forme de sortie JSON, qui peut ensuite être utilisée par le plug-in ZAP et Burp Suite. Ceci est utile dans les cas où le code source n'est pas fourni directement au testeur d'intrusion. Par exemple, le testeur d'intrusion peut obtenir le fichier de sortie json d'un client qui ne souhaite pas lui-même fournir le code source.

#### Comment utiliser

Le fichier jar CLI est disponible en téléchargement sur [https://github.com/secdec/attack-surface-detector-cli/releases](https://github.com/secdec/attack-surface-detector-cli/releases).

Vous pouvez exécuter la commande suivante pour ASD afin d'identifier les points de terminaison à partir du code source de l'application Web cible.

`java -jar attack-surface-detector-cli-1.3.5.jar <source-code-path> [flags]`

Voici un exemple d'exécution de la commande sur [OWASP RailsGoat](https://github.com/OWASP/railsgoat).

```text
$ java -jar attack-surface-detector-cli-1.3.5.jar railsgoat/
Beginning endpoint detection for '<...>/railsgoat' with 1 framework types
Using framework=RAILS
[0] GET: /login (0 variants): PARAMETERS={url=name=url, paramType=QUERY_STRING, dataType=STRING}; FILE=/app/controllers/sessions_contro
ller.rb (lines '6'-'9')
[1] GET: /logout (0 variants): PARAMETERS={}; FILE=/app/controllers/sessions_controller.rb (lines '33'-'37')
[2] POST: /forgot_password (0 variants): PARAMETERS={email=name=email, paramType=QUERY_STRING, dataType=STRING}; FILE=/app/controllers/
password_resets_controller.rb (lines '29'-'38')
[3] GET: /password_resets (0 variants): PARAMETERS={token=name=token, paramType=QUERY_STRING, dataType=STRING}; FILE=/app/controllers/p
assword_resets_controller.rb (lines '19'-'27')
[4] POST: /password_resets (0 variants): PARAMETERS={password=name=password, paramType=QUERY_STRING, dataType=STRING, user=name=user, paramType=QUERY_STRING, dataType=STRING, confirm_password=name=confirm_password, paramType=QUERY_STRING, dataType=STRING}; FILE=/app/controllers/password_resets_controller.rb (lines '5'-'17')
[5] GET: /sessions/new (0 variants): PARAMETERS={url=name=url, paramType=QUERY_STRING, dataType=STRING}; FILE=/app/controllers/sessions_controller.rb (lines '6'-'9')
[6] POST: /sessions (0 variants): PARAMETERS={password=name=password, paramType=QUERY_STRING, dataType=STRING, user_id=name=user_id, paramType=SESSION, dataType=STRING, remember_me=name=remember_me, paramType=QUERY_STRING, dataType=STRING, url=name=url, paramType=QUERY_STRING, dataType=STRING, email=name=email, paramType=QUERY_STRING, dataType=STRING}; FILE=/app/controllers/sessions_controller.rb (lines '11'-'31')
[7] DELETE: /sessions/{id} (0 variants): PARAMETERS={}; FILE=/app/controllers/sessions_controller.rb (lines '33'-'37')
[8] GET: /users (0 variants): PARAMETERS={}; FILE=/app/controllers/api/v1/users_controller.rb (lines '9'-'11')
[9] GET: /users/{id} (0 variants): PARAMETERS={}; FILE=/app/controllers/api/v1/users_controller.rb (lines '13'-'15')
... snipped ...
[38] GET: /api/v1/mobile/{id} (0 variants): PARAMETERS={id=name=id, paramType=QUERY_STRING, dataType=STRING, class=name=class, paramType=QUERY_STRING, dataType=STRING}; FILE=/app/controllers/api/v1/mobile_controller.rb (lines '8'-'13')
[39] GET: / (0 variants): PARAMETERS={url=name=url, paramType=QUERY_STRING, dataType=STRING}; FILE=/app/controllers/sessions_controller.rb (lines '6'-'9')
Generated 40 distinct endpoints with 0 variants for a total of 40 endpoints
Successfully validated serialization for these endpoints
0 endpoints were missing code start line
0 endpoints were missing code end line
0 endpoints had the same code start and end line
Generated 36 distinct parameters
Generated 36 total parameters
- 36/36 have their data type
- 0/36 have a list of accepted values
- 36/36 have their parameter type
--- QUERY_STRING: 35
--- SESSION: 1
Finished endpoint detection for '<...>/railsgoat'
----------
-- DONE --
0 projects had duplicate endpoints
Generated 40 distinct endpoints
Generated 40 total endpoints
Generated 36 distinct parameters
Generated 36 total parameters
1/1 projects had endpoints generated
To enable logging include the -debug argument
```

Vous pouvez également générer un fichier de sortie JSON en utilisant le drapeau `-json`, qui peut être utilisé par le plugin à la fois pour ZAP et Burp Suite. Voir les liens suivants pour plus de détails.

- [Accueil du plugin ASD pour OWASP ZAP] (https://github.com/secdec/attack-surface-detector-zap/wiki)
- [Accueil du plugin ASD pour PortSwigger Burp](https://github.com/secdec/attack-surface-detector-burp/wiki)

### Test des points d'entrée de l'application

Voici deux exemples de vérification des points d'entrée de l'application.

#### Exemple 1

Cet exemple montre une requête GET qui achèterait un article à partir d'une application d'achat en ligne.

```http
GET /shoppingApp/buyme.asp?CUSTOMERID=100&ITEM=z101a&PRICE=62.50&IP=x.x.x.x HTTP/1.1
Host: x.x.x.x
Cookie: SESSIONID=Z29vZCBqb2IgcGFkYXdhIG15IHVzZXJuYW1lIGlzIGZvbyBhbmQgcGFzc3dvcmQgaXMgYmFy
```

> Tous les paramètres de la requête tels que CUSTOMERID, ITEM, PRICE, IP et le Cookie, qui peuvent être simplement des paramètres encodés ou des paramètres utilisés pour l'état de la session.

#### Exemple 2

Cet exemple montre une requête POST qui vous connecterait à une application.

```http
POST /example/authenticate.asp?service=login HTTP/1.1
Host: x.x.x.x
Cookie: SESSIONID=dGhpcyBpcyBhIGJhZCBhcHAgdGhhdCBzZXRzIHByZWRpY3RhYmxlIGNvb2tpZXMgYW5kIG1pbmUgaXMgMTIzNA==;CustomCookie=00my00trusted00ip00is00x.x.x.x00

user=admin&pass=pass123&debug=true&fromtrustIP=true
```

On peut noter que les paramètres sont envoyés à plusieurs endroits :

1. Dans la chaîne de requête : `service`
2. Dans l'en-tête Cookie : `SESSIONID`, `CustomCookie`
3. Dans le corps de la requête : `user`, `pass`, `debug`, `fromtrustIP`

Le fait d'avoir une variété d'emplacements d'injection offre à l'attaquant des possibilités de chaînage qui pourraient améliorer les chances de trouver un bogue dans le code de gestion.

## Outils

- [OWASP Zed Attack Proxy (ZAP)](https://www.zaproxy.org/)
- [Burp Suite](https://www.portswigger.net/burp/)
- [Fiddler](https://www.telerik.com/fiddler)

## References

- [RFC 2616 – Protocole de transfert hypertexte – HTTP 1.1](https://tools.ietf.org/html/rfc2616)
- [Détecteur de surface d'attaque OWASP](https://owasp.org/www-project-attack-surface-detector/)
