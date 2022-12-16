# Tester les méthodes HTTP

|ID          |
|------------|
|WSTG-CONF-06|

## Sommaire

HTTP propose un certain nombre de méthodes (ou verbes) qui peuvent être utilisées pour effectuer des actions sur le serveur Web. Bien que GET et POST soient de loin les méthodes les plus courantes utilisées pour accéder aux informations fournies par un serveur Web, il existe une variété d'autres méthodes qui peuvent également être prises en charge et peuvent parfois être exploitées par des attaquants.

[RFC 7231](https://datatracker.ietf.org/doc/html/rfc7231) définit les principales méthodes de requête HTTP valides (ou verbes), bien que des méthodes supplémentaires aient été ajoutées dans d'autres RFC, telles que [RFC 5789]( https://datatracker.ietf.org/doc/html/rfc5789). Plusieurs de ces verbes ont été réutilisés à différentes fins dans les applications [RESTful](https://en.wikipedia.org/wiki/Representational_state_transfer), répertoriées dans le tableau ci-dessous.

| Méthode | Objectif initial | Objectif RESTful |
|---------|------------------|------------------|
| [`GET`](https://datatracker.ietf.org/doc/html/rfc7231#section-4.3.1) | Demander un dossier. | Demander un objet.|
| [`TÊTE`](https://datatracker.ietf.org/doc/html/rfc7231#section-4.3.2) | Demander un fichier, mais ne renvoyer que les en-têtes HTTP. | |
| [`POST`](https://datatracker.ietf.org/doc/html/rfc7231#section-4.3.3) | Soumettre des données. | |
| [`PUT`](https://datatracker.ietf.org/doc/html/rfc7231#section-4.3.4) | Télécharger un fichier. | Créer un objet. |
| [`SUPPRIMER`](https://datatracker.ietf.org/doc/html/rfc7231#section-4.3.5) | Supprimer un fichier | Supprimer un objet. |
| [`CONNECTER`](https://datatracker.ietf.org/doc/html/rfc7231#section-4.3.6) | Établissez une connexion à un autre système. | |
| [`OPTIONS`](https://datatracker.ietf.org/doc/html/rfc7231#section-4.3.7) | Répertorier les méthodes HTTP prises en charge. | Effectuez une requête [CORS Preflight](https://developer.mozilla.org/en-US/docs/Glossary/Preflight_request).
| [`TRACE`](https://datatracker.ietf.org/doc/html/rfc7231#section-4.3.8) | Faites écho à la requête HTTP à des fins de débogage. | |
| [`PATCH`](https://datatracker.ietf.org/doc/html/rfc5789#section-2) | | Modifier un objet. |

## Objectifs des tests

- Énumérer les méthodes HTTP prises en charge.
- Test de contournement du contrôle d'accès.
- Testez les techniques de remplacement de la méthode HTTP.

## Comment tester

### Découvrez les méthodes prises en charge

Pour effectuer ce test, le testeur a besoin d'un moyen d'identifier les méthodes HTTP prises en charge par le serveur Web examiné. La façon la plus simple de le faire est de faire une requête `OPTIONS` au serveur :

```http
OPTIONS / HTTP/1.1
Host: example.org
```

Le serveur devrait alors répondre avec une liste des méthodes prises en charge :

```http
HTTP/1.1 200 OK
Allow: OPTIONS, GET, HEAD, POST
```

Cependant, certains serveurs peuvent ne pas répondre aux requêtes `OPTIONS` ou renvoyer des informations inexactes. De plus, les serveurs peuvent prendre en charge différentes méthodes pour différents chemins - donc ce n'est pas parce qu'une méthode n'est pas prise en charge pour le répertoire racine `/` que cela signifie nécessairement qu'elle ne sera pas prise en charge ailleurs.

Un moyen plus fiable de tester les méthodes prises en charge consiste simplement à faire une demande avec ce type de méthode et à examiner la réponse du serveur. Si la méthode n'est pas autorisée, le serveur doit renvoyer un état `405 Méthode non autorisée`.

Notez que certains serveurs traitent les méthodes inconnues comme équivalentes à `GET`, ils peuvent donc répondre à des méthodes arbitraires, telles que la requête ci-dessous. Cela peut parfois être utile pour échapper à un pare-feu d'application Web ou à tout autre filtrage qui bloque des méthodes spécifiques.

```http
FOO / HTTP/1.1
Host: example.org
```

Les requêtes avec des méthodes arbitraires peuvent également être effectuées en utilisant curl avec l'option `-X` :

```bash
curl -X FOO https://example.org
```

Il existe également une variété d'outils automatisés qui peuvent tenter de déterminer les méthodes prises en charge, comme le script Nmap [`http-methods`](https://nmap.org/nsedoc/scripts/http-methods.html). Cependant, ces outils peuvent ne pas tester les méthodes dangereuses (c'est-à-dire les méthodes susceptibles de provoquer des modifications telles que `PUT` ou `DELETE`), ou peuvent provoquer involontairement des modifications du serveur Web si ces méthodes sont prises en charge. En tant que tels, ils doivent être utilisés avec précaution.

### METTRE et SUPPRIMER

Les méthodes `PUT` et `DELETE` peuvent avoir des effets différents, selon qu'elles sont interprétées par le serveur Web ou par l'application qui s'y exécute.

#### Serveurs Web hérités

Certains serveurs Web hérités autorisaient l'utilisation de la méthode "PUT" pour créer des fichiers sur le serveur. Par exemple, si le serveur est configuré pour autoriser cela, la requête ci-dessous créera un fichier sur le serveur appelé `test.html` avec le contenu `<script>alert(1)</script>`.

```http
PUT /test.html HTTP/1.1
Host: example.org
Content-Length: 25

<script>alert(1)</script>
```

Des requêtes similaires peuvent également être faites avec cURL :

```bash
curl https://example.org --upload-file test.html
```

Cela permet à un attaquant de télécharger des fichiers arbitraires sur le serveur Web, ce qui pourrait potentiellement entraîner une compromission complète du système s'il est autorisé à télécharger du code exécutable tel que des fichiers PHP. Cependant, cette configuration est extrêmement rare et il est peu probable qu'elle soit vue sur les systèmes modernes.

De même, la méthode `DELETE` peut être utilisée pour supprimer des fichiers du serveur Web. Notez qu'il s'agit d'une **action destructrice**, il convient donc d'être prudent lors du test de cette méthode.

```http
DELETE /test.html HTTP/1.1
Host: example.org
```

Ou avec cURL :

```bash
curl http://example.org/test.html -X DELETE
```

#### API RESTful

En revanche, les méthodes `PUT` et `DELETE` sont couramment utilisées par les applications RESTful modernes pour créer et supprimer des objets. Par exemple, la requête API ci-dessous pourrait être utilisée pour créer un utilisateur appelé "foo" avec un rôle "user":

```http
PUT /api/users/foo HTTP/1.1
Host: example.org
Content-Length: 34

{
    "role": "user"
}
```

Une requête similaire avec la méthode DELETE pourrait être utilisée pour supprimer un objet.

```http
DELETE /api/users/foo HTTP/1.1
Host: example.org
```

Bien qu'elle puisse être signalée par des outils d'analyse automatisés, la présence de ces méthodes sur une API RESTful **n'est pas un problème de sécurité**. Cependant, cette fonctionnalité peut présenter d'autres vulnérabilités (telles qu'un contrôle d'accès faible) et doit être testée de manière approfondie.

### TRACE

La méthode `TRACE` (ou la méthode `TRACK` équivalente de Microsoft) oblige le serveur à renvoyer le contenu de la requête. Cela a conduit à une vulnérabilité appelée Cross-Site Tracing (XST) publiée dans [2003](https://www.cgisecurity.com/whitehat-mirror/WH-WhitePaper_XST_ebook.pdf) (PDF), qui pourrait être utilisée pour accéder les cookies dont l'indicateur `HttpOnly` est défini. La méthode `TRACE` a été bloquée dans tous les navigateurs et plugins pendant de nombreuses années, et en tant que tel, ce problème n'est plus exploitable. Cependant, il peut toujours être signalé par des outils d'analyse automatisés, et la méthode `TRACE` activée sur un serveur Web suggère qu'elle n'a pas été correctement renforcée.

### RELIER

La méthode `CONNECT` oblige le serveur Web à ouvrir une connexion TCP vers un autre système, puis à transmettre le trafic du client à ce système. Cela pourrait permettre à un attaquant de diriger le trafic via le serveur, afin de masquer son adresse source, d'accéder à des systèmes internes ou d'accéder à des services liés à localhost. Un exemple de requête `CONNECT` est illustré ci-dessous :

```http
CONNECT 192.168.0.1:443 HTTP/1.1
Host: example.org
```

### CORRECTIF

La méthode `PATCH` est définie dans [RFC 5789](https://datatracker.ietf.org/doc/html/rfc5789) et est utilisée pour fournir des instructions sur la façon dont un objet doit être modifié. La RFC elle-même ne définit pas le format dans lequel ces instructions doivent être, mais diverses méthodes sont définies dans d'autres normes, telles que le [RFC 6902 - JavaScript Object Notation (JSON) Patch](https://datatracker.ietf.org/doc /html/rfc6902).

Par exemple, si nous avons un utilisateur appelé "foo" avec les propriétés suivantes :

```json
{
    "role": "user",
    "email": "foo@example.org"
}
```

La requête JSON PATCH suivante pourrait être utilisée pour changer le rôle de cet utilisateur "admin", sans modifier l'adresse email :

```http
PATCH /api/users/foo HTTP/1.1
Host: example.org

{ "op": "replace", "path": "/role", "value": "admin" }
```

Bien que la RFC indique qu'elle doit inclure des instructions sur la façon dont l'objet doit être modifié, la méthode `PATCH` est couramment (mal) utilisée pour inclure le contenu modifié à la place, comme indiqué ci-dessous. Tout comme la requête précédente, cela changerait la valeur "role" en "admin" sans modifier le reste de l'objet. Cela contraste avec la méthode `PUT`, qui écraserait l'objet entier (et donnerait ainsi un objet sans attribut "email").

```http
PATCH /api/users/foo HTTP/1.1
Host: example.org

{
    "role": "admin"
}
```

Comme avec la méthode `PUT`, cette fonctionnalité peut présenter des faiblesses de contrôle d'accès ou d'autres vulnérabilités. De plus, les applications peuvent ne pas effectuer le même niveau de validation d'entrée lors de la modification d'un objet que lors de la création d'un objet. Cela pourrait potentiellement permettre l'injection de valeurs malveillantes (comme dans une attaque de script intersite stocké), ou pourrait autoriser des objets cassés ou invalides pouvant entraîner des problèmes liés à la logique métier.

### Test de contournement du contrôle d'accès

Si une page de l'application redirige les utilisateurs vers une page de connexion avec un code `302` lorsqu'ils tentent d'y accéder directement, il peut être possible de contourner cela en faisant une demande avec une méthode HTTP différente, telle que `HEAD`, ` POST` ou même une méthode inventée telle que `FOO`. Si l'application Web répond par un `HTTP/1.1 200 OK` plutôt que par le `HTTP/1.1 302 Found` attendu, il peut être possible de contourner l'authentification ou l'autorisation. L'exemple ci-dessous montre comment une requête `HEAD` peut entraîner une page définissant des cookies administratifs, plutôt que de rediriger l'utilisateur vers une page de connexion :

```http
HEAD /admin/ HTTP/1.1
Host: example.org
```

```http
HTTP/1.1 200 OK
[...]
Set-Cookie: adminSessionCookie=[...];
```

Alternativement, il peut être possible de faire des demandes directes aux pages qui provoquent des actions, telles que :

```http
HEAD /admin/createUser.php?username=foo&password=bar&role=admin HTTP/1.1
Host: example.org
```

Ou :

```http
FOO /admin/createUser.php
Host: example.org
Content-Length: 36

username=foo&password=bar&role=admin
```

### Test du remplacement de la méthode HTTP

Certains frameworks Web fournissent un moyen de remplacer la méthode HTTP réelle dans la requête en émulant les verbes HTTP manquants en transmettant un en-tête personnalisé dans les requêtes. L'objectif principal est de contourner une application middleware (telle qu'un proxy ou un pare-feu d'application Web) qui bloque des méthodes spécifiques. Les en-têtes HTTP alternatifs suivants pourraient éventuellement être utilisés :

- `Méthode X-HTTP`
- `X-HTTP-Method-Override`
- `X-Method-Override`

Afin de tester cela, dans les scénarios où des verbes restreints tels que `PUT` ou `DELETE` renvoient une `405 Méthode non autorisée`, rejouez la même requête avec l'ajout des en-têtes alternatifs pour le remplacement de la méthode HTTP et observez comment le système répond. L'application doit répondre avec un code d'état différent (*par exemple* `200 OK`) dans les cas où le remplacement de méthode est pris en charge.

Le serveur Web de l'exemple suivant n'autorise pas la méthode `DELETE` et la bloque :

```http
DELETE /resource.html HTTP/1.1
Host: example.org
```

```http
HTTP/1.1 405 Method Not Allowed
[...]
```

Après avoir ajouté l'en-tête `X-HTTP-Method`, le serveur répond à la requête par un 200 :

```http
GET /resource.html HTTP/1.1
Host: example.org
X-HTTP-Method: DELETE
```

```http
HTTP/1.1 200 OK
[...]
```

## Correction

- Assurez-vous que seules les méthodes requises sont autorisées et que les méthodes autorisées sont correctement configurées.
- Assurez-vous qu'aucune solution de contournement n'est mise en uvre pour contourner les mesures de sécurité mises en œuvre par les agents utilisateurs, les frameworks ou les serveurs Web.

## Outils

- [Ncat](https://nmap.org/ncat/)
- [cURL](https://curl.haxx.se/)
- [Script NSE des méthodes http-Nmap](https://nmap.org/nsedoc/scripts/http-methods.html)

## Références

- [RFC 7231 - Protocole de transfert hypertexte (HTTP/1.1)](https://datatracker.ietf.org/doc/html/rfc7231)
- [RFC 5789 - Méthode PATCH pour HTTP](https://datatracker.ietf.org/doc/html/rfc5789)
- [HTACCESS : méthode BILBAO exposée](https://web.archive.org/web/20160616172703/http://www.kernelpanik.org/docs/kernelpanik/bme.eng.pdf)
- [Fortify - Remplacement de la méthode HTTP mal utilisé](https://vulncat.fortify.com/en/detail?id=desc.dynamic.xtended_preview.often_misused_http_method_override)
- [Mozilla Developer Network - Safe HTTP Methods](https://developer.mozilla.org/en-US/docs/Glossary/Safe/HTTP)
