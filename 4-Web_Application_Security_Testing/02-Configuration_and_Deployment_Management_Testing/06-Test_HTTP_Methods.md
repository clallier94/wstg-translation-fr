# Tester les m�thodes HTTP

|ID          |
|------------|
|WSTG-CONF-06|

## Sommaire

HTTP propose un certain nombre de m�thodes (ou verbes) qui peuvent �tre utilis�es pour effectuer des actions sur le serveur Web. Bien que GET et POST soient de loin les m�thodes les plus courantes utilis�es pour acc�der aux informations fournies par un serveur Web, il existe une vari�t� d'autres m�thodes qui peuvent �galement �tre prises en charge et peuvent parfois �tre exploit�es par des attaquants.

[RFC 7231](https://datatracker.ietf.org/doc/html/rfc7231) d�finit les principales m�thodes de requ�te HTTP valides (ou verbes), bien que des m�thodes suppl�mentaires aient �t� ajout�es dans d'autres RFC, telles que [RFC 5789]( https://datatracker.ietf.org/doc/html/rfc5789). Plusieurs de ces verbes ont �t� r�utilis�s � diff�rentes fins dans les applications [RESTful](https://en.wikipedia.org/wiki/Representational_state_transfer), r�pertori�es dans le tableau ci-dessous.

| M�thode | Objectif initial | Objectif RESTful |
|---------|------------------|------------------|
| [`GET`](https://datatracker.ietf.org/doc/html/rfc7231#section-4.3.1) | Demander un dossier. | Demander un objet.|
| [`T�TE`](https://datatracker.ietf.org/doc/html/rfc7231#section-4.3.2) | Demander un fichier, mais ne renvoyer que les en-t�tes HTTP. | |
| [`POST`](https://datatracker.ietf.org/doc/html/rfc7231#section-4.3.3) | Soumettre des donn�es. | |
| [`PUT`](https://datatracker.ietf.org/doc/html/rfc7231#section-4.3.4) | T�l�charger un fichier. | Cr�er un objet. |
| [`SUPPRIMER`](https://datatracker.ietf.org/doc/html/rfc7231#section-4.3.5) | Supprimer un fichier | Supprimer un objet. |
| [`CONNECTER`](https://datatracker.ietf.org/doc/html/rfc7231#section-4.3.6) | �tablissez une connexion � un autre syst�me. | |
| [`OPTIONS`](https://datatracker.ietf.org/doc/html/rfc7231#section-4.3.7) | R�pertorier les m�thodes HTTP prises en charge. | Effectuez une requ�te [CORS Preflight](https://developer.mozilla.org/en-US/docs/Glossary/Preflight_request).
| [`TRACE`](https://datatracker.ietf.org/doc/html/rfc7231#section-4.3.8) | Faites �cho � la requ�te HTTP � des fins de d�bogage. | |
| [`PATCH`](https://datatracker.ietf.org/doc/html/rfc5789#section-2) | | Modifier un objet. |

## Objectifs des tests

- �num�rer les m�thodes HTTP prises en charge.
- Test de contournement du contr�le d'acc�s.
- Testez les techniques de remplacement de la m�thode HTTP.

## Comment tester

### D�couvrez les m�thodes prises en charge

Pour effectuer ce test, le testeur a besoin d'un moyen d'identifier les m�thodes HTTP prises en charge par le serveur Web examin�. La fa�on la plus simple de le faire est de faire une requ�te `OPTIONS` au serveur�:

```http
OPTIONS / HTTP/1.1
Host: example.org
```

Le serveur devrait alors r�pondre avec une liste des m�thodes prises en charge�:

```http
HTTP/1.1 200 OK
Allow: OPTIONS, GET, HEAD, POST
```

Cependant, certains serveurs peuvent ne pas r�pondre aux requ�tes "OPTIONS" ou renvoyer des informations inexactes. De plus, les serveurs peuvent prendre en charge diff�rentes m�thodes pour diff�rents chemins - donc ce n'est pas parce qu'une m�thode n'est pas prise en charge pour le r�pertoire racine `/` que cela signifie n�cessairement qu'elle ne sera pas prise en charge ailleurs.

Un moyen plus fiable de tester les m�thodes prises en charge consiste simplement � faire une demande avec ce type de m�thode et � examiner la r�ponse du serveur. Si la m�thode n'est pas autoris�e, le serveur doit renvoyer un �tat "405 M�thode non autoris�e".

Notez que certains serveurs traitent les m�thodes inconnues comme �quivalentes � `GET`, ils peuvent donc r�pondre � des m�thodes arbitraires, telles que la requ�te ci-dessous. Cela peut parfois �tre utile pour �chapper � un pare-feu d'application Web ou � tout autre filtrage qui bloque des m�thodes sp�cifiques.

```http
FOO / HTTP/1.1
Host: example.org
```

Les requ�tes avec des m�thodes arbitraires peuvent �galement �tre effectu�es en utilisant curl avec l'option `-X`�:

```bash
curl -X FOO https://example.org
```

Il existe �galement une vari�t� d'outils automatis�s qui peuvent tenter de d�terminer les m�thodes prises en charge, comme le script Nmap [`http-methods`](https://nmap.org/nsedoc/scripts/http-methods.html). Cependant, ces outils peuvent ne pas tester les m�thodes dangereuses (c'est-�-dire les m�thodes susceptibles de provoquer des modifications telles que `PUT` ou `DELETE`), ou peuvent provoquer involontairement des modifications du serveur Web si ces m�thodes sont prises en charge. En tant que tels, ils doivent �tre utilis�s avec pr�caution.

### METTRE et SUPPRIMER

Les m�thodes `PUT` et `DELETE` peuvent avoir des effets diff�rents, selon qu'elles sont interpr�t�es par le serveur Web ou par l'application qui s'y ex�cute.

#### Serveurs Web h�rit�s

Certains serveurs Web h�rit�s autorisaient l'utilisation de la m�thode "PUT" pour cr�er des fichiers sur le serveur. Par exemple, si le serveur est configur� pour autoriser cela, la requ�te ci-dessous cr�era un fichier sur le serveur appel� `test.html` avec le contenu `<script>alert(1)</script>`.

```http
PUT /test.html HTTP/1.1
Host: example.org
Content-Length: 25

<script>alert(1)</script>
```

Des requ�tes similaires peuvent �galement �tre faites avec cURL�:

```bash
curl https://example.org --upload-file test.html
```

Cela permet � un attaquant de t�l�charger des fichiers arbitraires sur le serveur Web, ce qui pourrait potentiellement entra�ner une compromission compl�te du syst�me s'il est autoris� � t�l�charger du code ex�cutable tel que des fichiers PHP. Cependant, cette configuration est extr�mement rare et il est peu probable qu'elle soit vue sur les syst�mes modernes.

De m�me, la m�thode `DELETE` peut �tre utilis�e pour supprimer des fichiers du serveur Web. Notez qu'il s'agit d'une **action destructrice**, il convient donc d'�tre prudent lors du test de cette m�thode.

```http
DELETE /test.html HTTP/1.1
Host: example.org
```

Ou avec cURL�:

```bash
curl http://example.org/test.html -X DELETE
```

#### API RESTful

En revanche, les m�thodes `PUT` et `DELETE` sont couramment utilis�es par les applications RESTful modernes pour cr�er et supprimer des objets. Par exemple, la requ�te API ci-dessous pourrait �tre utilis�e pour cr�er un utilisateur appel� "foo" avec un r�le "user":

```http
PUT /api/users/foo HTTP/1.1
Host: example.org
Content-Length: 34

{
    "role": "user"
}
```

Une requ�te similaire avec la m�thode DELETE pourrait �tre utilis�e pour supprimer un objet.

```http
DELETE /api/users/foo HTTP/1.1
Host: example.org
```

Bien qu'elle puisse �tre signal�e par des outils d'analyse automatis�s, la pr�sence de ces m�thodes sur une API RESTful **n'est pas un probl�me de s�curit�**. Cependant, cette fonctionnalit� peut pr�senter d'autres vuln�rabilit�s (telles qu'un contr�le d'acc�s faible) et doit �tre test�e de mani�re approfondie.

### TRACE

La m�thode `TRACE` (ou la m�thode `TRACK` �quivalente de Microsoft) oblige le serveur � renvoyer le contenu de la requ�te. Cela a conduit � une vuln�rabilit� appel�e Cross-Site Tracing (XST) publi�e dans [2003](https://www.cgisecurity.com/whitehat-mirror/WH-WhitePaper_XST_ebook.pdf) (PDF), qui pourrait �tre utilis�e pour acc�der les cookies dont l'indicateur `HttpOnly` est d�fini. La m�thode `TRACE` a �t� bloqu�e dans tous les navigateurs et plugins pendant de nombreuses ann�es, et en tant que tel, ce probl�me n'est plus exploitable. Cependant, il peut toujours �tre signal� par des outils d'analyse automatis�s, et la m�thode "TRACE" activ�e sur un serveur Web sugg�re qu'elle n'a pas �t� correctement renforc�e.

### RELIER

La m�thode `CONNECT` oblige le serveur Web � ouvrir une connexion TCP vers un autre syst�me, puis � transmettre le trafic du client � ce syst�me. Cela pourrait permettre � un attaquant de diriger le trafic via le serveur, afin de masquer son adresse source, d'acc�der � des syst�mes internes ou d'acc�der � des services li�s � localhost. Un exemple de requ�te "CONNECT" est illustr� ci-dessous�:

```http
CONNECT 192.168.0.1:443 HTTP/1.1
Host: example.org
```

### CORRECTIF

La m�thode `PATCH` est d�finie dans [RFC 5789](https://datatracker.ietf.org/doc/html/rfc5789) et est utilis�e pour fournir des instructions sur la fa�on dont un objet doit �tre modifi�. La RFC elle-m�me ne d�finit pas le format dans lequel ces instructions doivent �tre, mais diverses m�thodes sont d�finies dans d'autres normes, telles que le [RFC 6902 - JavaScript Object Notation (JSON) Patch](https://datatracker.ietf.org/doc /html/rfc6902).

Par exemple, si nous avons un utilisateur appel� "foo" avec les propri�t�s suivantes�:

```json
{
    "role": "user",
    "email": "foo@example.org"
}
```

La requ�te JSON PATCH suivante pourrait �tre utilis�e pour changer le r�le de cet utilisateur "admin", sans modifier l'adresse email�:

```http
PATCH /api/users/foo HTTP/1.1
Host: example.org

{ "op": "replace", "path": "/role", "value": "admin" }
```

Bien que la RFC indique qu'elle doit inclure des instructions sur la fa�on dont l'objet doit �tre modifi�, la m�thode `PATCH` est couramment (mal) utilis�e pour inclure le contenu modifi� � la place, comme indiqu� ci-dessous. Tout comme la requ�te pr�c�dente, cela changerait la valeur "role" en "admin" sans modifier le reste de l'objet. Cela contraste avec la m�thode `PUT`, qui �craserait l'objet entier (et donnerait ainsi un objet sans attribut "email").

```http
PATCH /api/users/foo HTTP/1.1
Host: example.org

{
    "role": "admin"
}
```

Comme avec la m�thode "PUT", cette fonctionnalit� peut pr�senter des faiblesses de contr�le d'acc�s ou d'autres vuln�rabilit�s. De plus, les applications peuvent ne pas effectuer le m�me niveau de validation d'entr�e lors de la modification d'un objet que lors de la cr�ation d'un objet. Cela pourrait potentiellement permettre l'injection de valeurs malveillantes (comme dans une attaque de script intersite stock�), ou pourrait autoriser des objets cass�s ou invalides pouvant entra�ner des probl�mes li�s � la logique m�tier.

### Test de contournement du contr�le d'acc�s

Si une page de l'application redirige les utilisateurs vers une page de connexion avec un code `302` lorsqu'ils tentent d'y acc�der directement, il peut �tre possible de contourner cela en faisant une demande avec une m�thode HTTP diff�rente, telle que `HEAD`, ` POST` ou m�me une m�thode invent�e telle que `FOO`. Si l'application Web r�pond par un "HTTP/1.1 200 OK" plut�t que par le "HTTP/1.1 302 Found" attendu, il peut �tre possible de contourner l'authentification ou l'autorisation. L'exemple ci-dessous montre comment une requ�te `HEAD` peut entra�ner une page d�finissant des cookies administratifs, plut�t que de rediriger l'utilisateur vers une page de connexion�:

```http
HEAD /admin/ HTTP/1.1
Host: example.org
```

```http
HTTP/1.1 200 OK
[...]
Set-Cookie: adminSessionCookie=[...];
```

Alternativement, il peut �tre possible de faire des demandes directes aux pages qui provoquent des actions, telles que�:

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

### Test du remplacement de la m�thode HTTP

Certains frameworks Web fournissent un moyen de remplacer la m�thode HTTP r�elle dans la requ�te en �mulant les verbes HTTP manquants en transmettant un en-t�te personnalis� dans les requ�tes. L'objectif principal est de contourner une application middleware (telle qu'un proxy ou un pare-feu d'application Web) qui bloque des m�thodes sp�cifiques. Les en-t�tes HTTP alternatifs suivants pourraient �ventuellement �tre utilis�s�:

- `M�thode X-HTTP`
- `X-HTTP-Method-Override`
- `X-Method-Override`

Afin de tester cela, dans les sc�narios o� des verbes restreints tels que `PUT` ou `DELETE` renvoient une `405 M�thode non autoris�e`, rejouez la m�me requ�te avec l'ajout des en-t�tes alternatifs pour le remplacement de la m�thode HTTP et observez comment le syst�me r�pond. L'application doit r�pondre avec un code d'�tat diff�rent (*par exemple* `200 OK`) dans les cas o� le remplacement de m�thode est pris en charge.

Le serveur Web de l'exemple suivant n'autorise pas la m�thode "DELETE" et la bloque�:

```http
DELETE /resource.html HTTP/1.1
Host: example.org
```

```http
HTTP/1.1 405 Method Not Allowed
[...]
```

Apr�s avoir ajout� l'en-t�te `X-HTTP-Method`, le serveur r�pond � la requ�te par un 200�:

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

- Assurez-vous que seules les m�thodes requises sont autoris�es et que les m�thodes autoris�es sont correctement configur�es.
- Assurez-vous qu'aucune solution de contournement n'est mise en �uvre pour contourner les mesures de s�curit� mises en �uvre par les agents utilisateurs, les frameworks ou les serveurs Web.

## Outils

- [Ncat](https://nmap.org/ncat/)
- [cURL](https://curl.haxx.se/)
- [Script NSE des m�thodes http-Nmap] (https://nmap.org/nsedoc/scripts/http-methods.html)

## R�f�rences

- [RFC 7231 - Protocole de transfert hypertexte (HTTP/1.1)](https://datatracker.ietf.org/doc/html/rfc7231)
- [RFC 5789 - M�thode PATCH pour HTTP](https://datatracker.ietf.org/doc/html/rfc5789)
- [HTACCESS�: m�thode BILBAO expos�e] (https://web.archive.org/web/20160616172703/http://www.kernelpanik.org/docs/kernelpanik/bme.eng.pdf)
- [Fortify - Remplacement de la m�thode HTTP mal utilis�](https://vulncat.fortify.com/en/detail?id=desc.dynamic.xtended_preview.often_misused_http_method_override)
- [Mozilla Developer Network - Safe HTTP Methods](https://developer.mozilla.org/en-US/docs/Glossary/Safe/HTTP)
