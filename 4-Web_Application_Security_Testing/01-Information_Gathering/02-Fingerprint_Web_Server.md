# Serveur Web d'empreintes digitales

|ID          |
|------------|
|WSTG-INFO-02|

## Sommaire

L'empreinte digitale du serveur Web consiste � identifier le type et la version du serveur Web sur lequel une cible s'ex�cute. Bien que les empreintes digitales de serveur Web soient souvent encapsul�es dans des outils de test automatis�s, il est important que les chercheurs comprennent les principes fondamentaux de la mani�re dont ces outils tentent d'identifier les logiciels, et pourquoi cela est utile.

La d�couverte pr�cise du type de serveur Web sur lequel une application s'ex�cute peut permettre aux testeurs de s�curit� de d�terminer si l'application est vuln�rable aux attaques. En particulier, les serveurs ex�cutant des versions plus anciennes de logiciels sans correctifs de s�curit� � jour peuvent �tre sensibles aux exploits connus sp�cifiques � la version.

## Objectifs des tests

- D�terminez la version et le type d'un serveur Web en cours d'ex�cution pour permettre une d�couverte plus approfondie de toute vuln�rabilit� connue.

## Comment tester

Les techniques utilis�es pour la prise d'empreintes digitales du serveur Web incluent la [saisie de banni�re](https://en.wikipedia.org/wiki/Banner_grabbing), l'obtention de r�ponses � des requ�tes malform�es et l'utilisation d'outils automatis�s pour effectuer des analyses plus robustes qui utilisent une combinaison de tactiques. Le principe fondamental par lequel toutes ces techniques fonctionnent est le m�me. Ils s'efforcent tous d'obtenir une r�ponse du serveur Web qui peut ensuite �tre compar�e � une base de donn�es de r�ponses et de comportements connus, et donc associ�e � un type de serveur connu.

### Saisie de banni�res

Une capture de banni�re est effectu�e en envoyant une requ�te HTTP au serveur Web et en examinant son [en-t�te de r�ponse] (https://developer.mozilla.org/en-US/docs/Glossary/Response_header). Cela peut �tre accompli en utilisant une vari�t� d'outils, y compris `telnet` pour les requ�tes HTTP ou `openssl` pour les requ�tes via TLS/SSL.

Par exemple, voici la r�ponse � une requ�te d'un serveur Apache.

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

Voici une autre r�ponse, cette fois de nginx.

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

Voici � quoi ressemble une r�ponse de lighttpd.

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

Dans ces exemples, le type et la version du serveur sont clairement expos�s. Cependant, les applications soucieuses de la s�curit� peuvent obscurcir les informations de leur serveur en modifiant l'en-t�te. Par exemple, voici un extrait de la r�ponse � une demande de site avec un en-t�te modifi�:

```sh
HTTP/1.1 200 OK
Server: Website.com
Date: Thu, 05 Sep 2019 17:57:06 GMT
Content-Type: text/html; charset=utf-8
Status: 200 OK
...
```

Dans les cas o� les informations sur le serveur sont masqu�es, les testeurs peuvent deviner le type de serveur en fonction de l'ordre des champs d'en-t�te. Notez que dans l'exemple Apache ci-dessus, les champs suivent cet ordre�:

- Date
- Serveur
- Derni�re modification
- Etag
- Plages d'acceptation
- Longueur du contenu
- Lien
- Type de contenu

Cependant, dans les exemples de serveur nginx et obscurci, les champs communs suivent cet ordre�:

- Serveur
- Date
- Type de contenu

Les testeurs peuvent utiliser ces informations pour deviner que le serveur masqu� est nginx. Cependant, �tant donn� qu'un certain nombre de serveurs Web diff�rents peuvent partager le m�me ordre de champs et que des champs peuvent �tre modifi�s ou supprim�s, cette m�thode n'est pas d�finitive.

### Envoi de requ�tes malform�es

Les serveurs Web peuvent �tre identifi�s en examinant leurs r�ponses d'erreur et, dans les cas o� ils n'ont pas �t� personnalis�s, leurs pages d'erreur par d�faut. Une fa�on de contraindre un serveur � les pr�senter consiste � envoyer des requ�tes intentionnellement incorrectes ou mal form�es.

Par exemple, voici la r�ponse � une requ�te pour la m�thode inexistante `SANTA CLAUS` d'un serveur Apache.

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

Voici la r�ponse � la m�me demande de nginx.

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

Voici la r�ponse � la m�me requ�te de lighttpd.

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

Comme les pages d'erreur par d�faut offrent de nombreux facteurs de diff�renciation entre les types de serveurs Web, leur examen peut �tre une m�thode efficace pour la prise d'empreintes digitales m�me lorsque les champs d'en-t�te du serveur sont masqu�s.

### Utilisation des outils d'analyse automatis�s

Comme indiqu� pr�c�demment, la prise d'empreintes digitales du serveur Web est souvent incluse en tant que fonctionnalit� des outils d'analyse automatis�s. Ces outils sont capables de faire des requ�tes similaires � celles pr�sent�es ci-dessus, ainsi que d'envoyer d'autres sondes plus sp�cifiques au serveur. Les outils automatis�s peuvent comparer les r�ponses des serveurs Web beaucoup plus rapidement que les tests manuels et utiliser de grandes bases de donn�es de r�ponses connues pour tenter d'identifier le serveur. Pour ces raisons, les outils automatis�s sont plus susceptibles de produire des r�sultats pr�cis.

Voici quelques outils d'analyse couramment utilis�s qui incluent la fonctionnalit� d'empreinte digitale du serveur Web.

- [Netcraft](https://toolbar.netcraft.com/site_report), un outil en ligne qui analyse les sites Web � la recherche d'informations, y compris le serveur Web.
- [Nikto](https://github.com/sullo/nikto), un outil d'analyse en ligne de commande Open Source.
- [Nmap](https://nmap.org/), un outil en ligne de commande Open Source qui poss�de �galement une interface graphique, [Zenmap](https://nmap.org/zenmap/).

## Correction

Bien que les informations de serveur expos�es ne constituent pas n�cessairement une vuln�rabilit� en soi, ce sont des informations qui peuvent aider les attaquants � exploiter d'autres vuln�rabilit�s qui peuvent exister. Les informations de serveur expos�es peuvent �galement conduire les attaquants � trouver des vuln�rabilit�s de serveur sp�cifiques � la version qui peuvent �tre utilis�es pour exploiter des serveurs non corrig�s. Pour cette raison, il est recommand� de prendre certaines pr�cautions. Ces actions comprennent :

- Obscurcissement des informations du serveur Web dans les en-t�tes, comme avec le [module mod_headers] d'Apache (https://httpd.apache.org/docs/current/mod/mod_headers.html).
- Utilisation d'un [serveur proxy inverse] renforc� (https://en.wikipedia.org/wiki/Proxy_server#Reverse_proxies) pour cr�er une couche de s�curit� suppl�mentaire entre le serveur Web et Internet.
- Veiller � ce que les serveurs Web soient mis � jour avec les derniers logiciels et correctifs de s�curit�.
