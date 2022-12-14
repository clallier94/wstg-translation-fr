# Empreintes sur les serveur Web

|ID          |
|------------|
|WSTG-INFO-02|

## Sommaire

L'empreinte digitale du serveur Web consiste à identifier le type et la version du serveur Web sur lequel une cible s'exécute. Bien que les empreintes digitales de serveur Web soient souvent encapsulées dans des outils de test automatisés, il est important que les chercheurs comprennent les principes fondamentaux de la manière dont ces outils tentent d'identifier les logiciels, et pourquoi cela est utile.

La découverte précise du type de serveur Web sur lequel une application s'exécute peut permettre aux testeurs de sécurité de déterminer si l'application est vulnérable aux attaques. En particulier, les serveurs exécutant des versions plus anciennes de logiciels sans correctifs de sécurité à jour peuvent être sensibles aux exploits connus spécifiques à la version.

## Objectifs des tests

- Déterminez la version et le type d'un serveur Web en cours d'exécution pour permettre une découverte plus approfondie de toute vulnérabilité connue.

## Comment tester

Les techniques utilisées pour la prise d'empreintes digitales du serveur Web incluent la [saisie de bannière](https://en.wikipedia.org/wiki/Banner_grabbing), l'obtention de réponses à des requêtes malformées et l'utilisation d'outils automatisés pour effectuer des analyses plus robustes qui utilisent une combinaison de tactiques. Le principe fondamental par lequel toutes ces techniques fonctionnent est le même. Ils s'efforcent tous d'obtenir une réponse du serveur Web qui peut ensuite être comparée à une base de données de réponses et de comportements connus, et donc associée à un type de serveur connu.

### Saisie de bannières

Une capture de bannière est effectuée en envoyant une requête HTTP au serveur Web et en examinant son [en-tête de réponse](https://developer.mozilla.org/en-US/docs/Glossary/Response_header). Cela peut être accompli en utilisant une variété d'outils, y compris `telnet` pour les requêtes HTTP ou `openssl` pour les requêtes via TLS/SSL.

Par exemple, voici la réponse à une requête d'un serveur Apache.

```http
HTTP/1.1 200 OK
Date: Thu, 05 Sep 2019 17:42:39 GMT
Server: Apache/2.4.41 (Unix)
Last-Modified: Thu, 05 Sep 2019 17:40:42 GMT
ETag: "75-591d1d21b6167"
Accept-Ranges: bytes
Content-Length: 117
Connection: close
Content-Type: text/html
...
```

Voici une autre réponse, cette fois de nginx.

```http
HTTP/1.1 200 OK
Server: nginx/1.17.3
Date: Thu, 05 Sep 2019 17:50:24 GMT
Content-Type: text/html
Content-Length: 117
Last-Modified: Thu, 05 Sep 2019 17:40:42 GMT
Connection: close
ETag: "5d71489a-75"
Accept-Ranges: bytes
...
```

Voici à quoi ressemble une réponse de lighttpd.

```sh
HTTP/1.0 200 OK
Content-Type: text/html
Accept-Ranges: bytes
ETag: "4192788355"
Last-Modified: Thu, 05 Sep 2019 17:40:42 GMT
Content-Length: 117
Connection: close
Date: Thu, 05 Sep 2019 17:57:57 GMT
Server: lighttpd/1.4.54
```

Dans ces exemples, le type et la version du serveur sont clairement exposés. Cependant, les applications soucieuses de la sécurité peuvent obscurcir les informations de leur serveur en modifiant l'en-tête. Par exemple, voici un extrait de la réponse à une demande de site avec un en-tête modifié :

```sh
HTTP/1.1 200 OK
Server: Website.com
Date: Thu, 05 Sep 2019 17:57:06 GMT
Content-Type: text/html; charset=utf-8
Status: 200 OK
...
```

Dans les cas où les informations sur le serveur sont masquées, les testeurs peuvent deviner le type de serveur en fonction de l'ordre des champs d'en-tête. Notez que dans l'exemple Apache ci-dessus, les champs suivent cet ordre :

- Date
- Serveur
- Dernière modification
- Etag
- Plages d'acceptation
- Longueur du contenu
- Lien
- Type de contenu

Cependant, dans les exemples de serveur nginx et obscurci, les champs communs suivent cet ordre :

- Serveur
- Date
- Type de contenu

Les testeurs peuvent utiliser ces informations pour deviner que le serveur masqué est nginx. Cependant, étant donné qu'un certain nombre de serveurs Web différents peuvent partager le même ordre de champs et que des champs peuvent être modifiés ou supprimés, cette méthode n'est pas définitive.

### Envoi de requêtes malformées

Les serveurs Web peuvent être identifiés en examinant leurs réponses d'erreur et, dans les cas où ils n'ont pas été personnalisés, leurs pages d'erreur par défaut. Une façon de contraindre un serveur à les présenter consiste à envoyer des requêtes intentionnellement incorrectes ou mal formées.

Par exemple, voici la réponse à une requête pour la méthode inexistante `SANTA CLAUS` d'un serveur Apache.

```sh
GET / SANTA CLAUS/1.1


HTTP/1.1 400 Bad Request
Date: Fri, 06 Sep 2019 19:21:01 GMT
Server: Apache/2.4.41 (Unix)
Content-Length: 226
Connection: close
Content-Type: text/html; charset=iso-8859-1

<!DOCTYPE HTML PUBLIC "-//IETF//DTD HTML 2.0//EN">
<html><head>
<title>400 Bad Request</title>
</head><body>
<h1>Bad Request</h1>
<p>Your browser sent a request that this server could not understand.<br />
</p>
</body></html>
```

Voici la réponse à la même demande de nginx.

```sh
GET / SANTA CLAUS/1.1


<html>
<head><title>404 Not Found</title></head>
<body>
<center><h1>404 Not Found</h1></center>
<hr><center>nginx/1.17.3</center>
</body>
</html>
```

Voici la réponse à la même requête de lighttpd.

```sh
GET / SANTA CLAUS/1.1


HTTP/1.0 400 Bad Request
Content-Type: text/html
Content-Length: 345
Connection: close
Date: Sun, 08 Sep 2019 21:56:17 GMT
Server: lighttpd/1.4.54

<?xml version="1.0" encoding="iso-8859-1"?>
<!DOCTYPE html PUBLIC "-//W3C//DTD XHTML 1.0 Transitional//EN"
         "http://www.w3.org/TR/xhtml1/DTD/xhtml1-transitional.dtd">
<html xmlns="http://www.w3.org/1999/xhtml" xml:lang="en" lang="en">
 <head>
  <title>400 Bad Request</title>
 </head>
 <body>
  <h1>400 Bad Request</h1>
 </body>
</html>
```

Comme les pages d'erreur par défaut offrent de nombreux facteurs de différenciation entre les types de serveurs Web, leur examen peut être une méthode efficace pour la prise d'empreintes digitales même lorsque les champs d'en-tête du serveur sont masqués.

### Utilisation des outils d'analyse automatisés

Comme indiqué précédemment, la prise d'empreintes digitales du serveur Web est souvent incluse en tant que fonctionnalité des outils d'analyse automatisés. Ces outils sont capables de faire des requêtes similaires à celles présentées ci-dessus, ainsi que d'envoyer d'autres sondes plus spécifiques au serveur. Les outils automatisés peuvent comparer les réponses des serveurs Web beaucoup plus rapidement que les tests manuels et utiliser de grandes bases de données de réponses connues pour tenter d'identifier le serveur. Pour ces raisons, les outils automatisés sont plus susceptibles de produire des résultats précis.

Voici quelques outils d'analyse couramment utilisés qui incluent la fonctionnalité d'empreinte digitale du serveur Web.

- [Netcraft](https://toolbar.netcraft.com/site_report), un outil en ligne qui analyse les sites Web à la recherche d'informations, y compris le serveur Web.
- [Nikto](https://github.com/sullo/nikto), un outil d'analyse en ligne de commande Open Source.
- [Nmap](https://nmap.org/), un outil en ligne de commande Open Source qui possède également une interface graphique, [Zenmap](https://nmap.org/zenmap/).

## Correction

Bien que les informations de serveur exposées ne constituent pas nécessairement une vulnérabilité en soi, ce sont des informations qui peuvent aider les attaquants à exploiter d'autres vulnérabilités qui peuvent exister. Les informations de serveur exposées peuvent également conduire les attaquants à trouver des vulnérabilités de serveur spécifiques à la version qui peuvent être utilisées pour exploiter des serveurs non corrigés. Pour cette raison, il est recommandé de prendre certaines précautions. Ces actions comprennent :

- Obscurcissement des informations du serveur Web dans les en-têtes, comme avec le [module mod_headers](https://httpd.apache.org/docs/current/mod/mod_headers.html) d'Apache .
- Utilisation d'un [serveur proxy inverse](https://en.wikipedia.org/wiki/Proxy_server#Reverse_proxies) renforcé  pour créer une couche de sécurité supplémentaire entre le serveur Web et Internet.
- Veiller à ce que les serveurs Web soient mis à jour avec les derniers logiciels et correctifs de sécurité.
