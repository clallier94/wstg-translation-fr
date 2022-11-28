# Test de la contrebande de fractionnement HTTP

|ID |
|------------|
|WSTG-INPV-15|

## Sommaire

Cette section illustre des exemples d'attaques qui exploitent des fonctionnalit�s sp�cifiques du protocole HTTP, soit en exploitant les faiblesses de l'application Web, soit des particularit�s dans la mani�re dont diff�rents agents interpr�tent les messages HTTP.
Cette section analysera deux attaques diff�rentes qui ciblent des en-t�tes HTTP sp�cifiques�:

- Fractionnement HTTP
- Contrebande HTTP

La premi�re attaque exploite un manque de nettoyage des entr�es qui permet � un intrus d'ins�rer des caract�res CR et LF dans les en-t�tes de la r�ponse de l'application et de "diviser" cette r�ponse en deux messages HTTP diff�rents. L'objectif de l'attaque peut varier d'un empoisonnement du cache � un cross site scripting.

Dans la deuxi�me attaque, l'attaquant exploite le fait que certains messages HTTP sp�cialement con�us peuvent �tre analys�s et interpr�t�s de diff�rentes mani�res selon l'agent qui les re�oit. La contrebande HTTP n�cessite un certain niveau de connaissances sur les diff�rents agents qui traitent les messages HTTP (serveur Web, proxy, pare-feu) et ne sera donc inclus que dans la section de test de la bo�te grise.

## Objectifs des tests

- �valuer si l'application est vuln�rable au fractionnement, en identifiant quelles attaques possibles sont r�alisables.
- �valuer si la cha�ne de communication est vuln�rable � la contrebande, en identifiant quelles attaques possibles sont r�alisables.

## Comment tester

### Test de la bo�te noire

#### Fractionnement HTTP

Certaines applications Web utilisent une partie de l'entr�e utilisateur pour g�n�rer les valeurs de certains en-t�tes de leurs r�ponses. L'exemple le plus simple est fourni par les redirections dans lesquelles l'URL cible d�pend d'une valeur soumise par l'utilisateur. Disons par exemple que l'utilisateur est invit� � choisir s'il pr�f�re une interface Web standard ou avanc�e. Le choix sera pass� en param�tre qui sera utilis� dans l'ent�te de la r�ponse pour d�clencher la redirection vers la page correspondante.

Plus pr�cis�ment, si le param�tre 'interface' a la valeur 'advanced', l'application r�pondra par ceci :

```http
HTTP/1.1 302 Moved Temporarily
Date: Sun, 03 Dec 2005 16:22:19 GMT
Location: http://victim.com/main.jsp?interface=advanced
<snip>
```

Lors de la r�ception de ce message, le navigateur am�nera l'utilisateur � la page indiqu�e dans l'en-t�te Emplacement. Cependant, si l'application ne filtre pas l'entr�e utilisateur, il sera possible d'ins�rer dans le param�tre 'interface' la s�quence %0d%0a, qui repr�sente la s�quence CRLF utilis�e pour s�parer les diff�rentes lignes. � ce stade, les testeurs pourront d�clencher une r�ponse qui sera interpr�t�e comme deux r�ponses diff�rentes par quiconque l'analysera, par exemple un cache Web situ� entre nous et l'application. Cela peut �tre exploit� par un attaquant pour empoisonner ce cache Web afin qu'il fournisse un faux contenu dans toutes les requ�tes ult�rieures.

Disons que dans l'exemple pr�c�dent, le testeur passe les donn�es suivantes comme param�tre d'interface�:

`Advanced%0d%0aContent-Length:%200%0d%0a%0d%0aHTTP/1.1%20200%20OK%0d%0aContent-Type:%20text/html%0d%0aContent-Length:%2035%0d%0a% 0d%0a<html>Sorry,%20System%20Down</html>`

La r�ponse r�sultante de l'application vuln�rable sera donc la suivante :

```http
HTTP/1.1 302 Moved Temporarily
Date: Sun, 03 Dec 2005 16:22:19 GMT
Location: http://victim.com/main.jsp?interface=advanced
Content-Length: 0

HTTP/1.1 200 OK
Content-Type: text/html
Content-Length: 35

<html>Sorry,%20System%20Down</html>
<other data>
```

Le cache Web verra deux r�ponses diff�rentes, donc si l'attaquant envoie, imm�diatement apr�s la premi�re requ�te, une seconde demandant `/index.html`, le cache Web fera correspondre cette requ�te avec la deuxi�me r�ponse et mettra son contenu en cache, donc que toutes les requ�tes ult�rieures dirig�es vers `victim.com/index.html` passant par ce cache Web recevront le message "syst�me en panne". De cette mani�re, un attaquant serait en mesure de d�figurer efficacement le site pour tous les utilisateurs utilisant ce cache Web (l'ensemble d'Internet, si le cache Web est un proxy inverse pour l'application Web).

Alternativement, l'attaquant pourrait transmettre � ces utilisateurs un extrait de code JavaScript qui monte une attaque de script intersite, par exemple pour voler les cookies. Notez que bien que la vuln�rabilit� soit dans l'application, la cible ici est ses utilisateurs. Par cons�quent, afin de rechercher cette vuln�rabilit�, le testeur doit identifier toutes les entr�es contr�l�es par l'utilisateur qui influencent un ou plusieurs en-t�tes dans la r�ponse, et v�rifier s'il peut y injecter avec succ�s une s�quence CR+LF.

Les en-t�tes qui sont les candidats les plus probables pour cette attaque sont�:

- `Location`
- `Set-Cookie`

Il faut noter qu'une exploitation r�ussie de cette vuln�rabilit� dans un sc�nario r�el peut �tre assez complexe, car plusieurs facteurs doivent �tre pris en compte�:

1. Le testeur de stylo doit correctement d�finir les en-t�tes dans la fausse r�ponse pour qu'elle soit mise en cache avec succ�s (par exemple, un en-t�te Last-Modified avec une date d�finie dans le futur). Ils peuvent �galement avoir � d�truire les versions pr�c�demment mises en cache des pagers cibles, en �mettant une requ�te pr�liminaire avec "Pragma�: no-cache" dans les en-t�tes de requ�te.
2. L'application, bien qu'elle ne filtre pas la s�quence CR+LF, peut filtrer d'autres caract�res n�cessaires � une attaque r�ussie (par exemple, `<` et `>`). Dans ce cas, le testeur peut essayer d'utiliser d'autres encodages (par exemple, UTF-7)
3. Certaines cibles (par exemple, ASP) coderont en URL la partie chemin de l'en-t�te Location (par exemple, `www.victim.com/redirect.asp`), rendant une s�quence CRLF inutile. Cependant, ils ne parviennent pas � encoder la section de requ�te (par exemple, ?interface=advanced), ce qui signifie qu'un point d'interrogation principal suffit pour contourner ce filtrage.

Pour une discussion plus d�taill�e sur cette attaque et d'autres informations sur les sc�narios et applications possibles, consultez les documents r�f�renc�s au bas de cette section.

###�Test de la bo�te grise

#### Fractionnement HTTP

Une exploitation r�ussie de HTTP Splitting est grandement facilit�e par la connaissance de certains d�tails de l'application Web et de la cible de l'attaque. Par exemple, diff�rentes cibles peuvent utiliser diff�rentes m�thodes pour d�cider quand le premier message HTTP se termine et quand le second commence. Certains utiliseront les limites de message, comme dans l'exemple pr�c�dent. D'autres cibles supposeront que diff�rents messages seront transport�s par diff�rents paquets. D'autres alloueront pour chaque message un nombre de morceaux de longueur pr�d�termin�e : dans ce cas, le deuxi�me message devra commencer exactement au d�but d'un morceau et cela obligera le testeur � utiliser un bourrage entre les deux messages. Cela peut causer des probl�mes lorsque le param�tre vuln�rable doit �tre envoy� dans l'URL, car une URL tr�s longue est susceptible d'�tre tronqu�e ou filtr�e. Un sc�nario de bo�te grise peut aider l'attaquant � trouver une solution de contournement�: plusieurs serveurs d'application, par exemple, autoriseront l'envoi de la requ�te en utilisant POST au lieu de GET.

#### Contrebande HTTP

Comme mentionn� dans l'introduction, HTTP Smuggling exploite les diff�rentes fa�ons dont un message HTTP particuli�rement con�u peut �tre analys� et interpr�t� par diff�rents agents (navigateurs, caches Web, pare-feu d'application). Ce type d'attaque relativement nouveau a �t� d�couvert par Chaim Linhart, Amit Klein, Ronen Heled et Steve Orrin en 2005. Il existe plusieurs applications possibles et nous analyserons l'une des plus spectaculaires : le contournement d'un pare-feu applicatif. Reportez-vous au livre blanc d'origine (li� au bas de cette page) pour des informations plus d�taill�es et d'autres sc�narios.

##### Contournement du pare-feu applicatif

Il existe plusieurs produits qui permettent � une administration syst�me de d�tecter et de bloquer une requ�te Web hostile en fonction d'un sch�ma malveillant connu int�gr� � la requ�te. Par exemple, consid�rez l'inf�me et ancienne [attaque de travers�e de r�pertoire Unicode contre le serveur IIS] (https://www.securityfocus.com/bid/1806), dans laquelle un attaquant pourrait casser la racine www en �mettant une requ�te comme�:

`http://target/scripts/..%c1%1c../winnt/system32/cmd.exe?/c+<command_to_execute>`

Bien s�r, il est assez facile de rep�rer et de filtrer cette attaque par la pr�sence de cha�nes comme ".." et "cmd.exe" dans l'URL. Cependant, IIS 5.0 est assez pointilleux sur les requ�tes POST dont le corps est jusqu'� 48 Ko et tronque tout le contenu qui d�passe cette limite lorsque l'en-t�te Content-Type est diff�rent de application/x-www-form-urlencoded. Le pen-testeur peut en tirer parti en cr�ant une requ�te tr�s volumineuse, structur�e comme suit�:

```html
POST /target.asp HTTP/1.1        <-- Request #1
Host: target
Connection: Keep-Alive
Content-Length: 49225
<CRLF>
<49152 bytes of garbage>
```

```html
POST /target.asp HTTP/1.0        <-- Request #2
Connection: Keep-Alive
Content-Length: 33
<CRLF>
```

```html
POST /target.asp HTTP/1.0        <-- Request #3
xxxx: POST /scripts/..%c1%1c../winnt/system32/cmd.exe?/c+dir HTTP/1.0   <-- Request #4
Connection: Keep-Alive
<CRLF>
```

Ce qui se passe ici est que la `Request #1` est compos�e de 49223 octets, ce qui inclut �galement les lignes de `Request #2`. Par cons�quent, un pare-feu (ou tout autre agent � c�t� d'IIS 5.0) verra la requ�te n��1, ne verra pas la requ�te n��2 (ses donn�es ne feront partie que de la n��1), verra la requ�te n��3 et ratera ` Requ�te #4` (parce que le POST ne sera qu'une partie du faux en-t�te xxxx).

Maintenant, qu'arrive-t-il � IIS 5.0 ? Il arr�tera d'analyser `Request #1` juste apr�s les 49152 octets de d�chets (car il aura atteint la limite de 48K = 49152 octets) et analysera donc `Request #2` comme une nouvelle demande distincte. `Request #2` pr�tend que son contenu est de 33 octets, ce qui inclut tout jusqu'� "xxxx: ", ce qui fait que IIS manque `Request #3` (interpr�t� comme faisant partie de `Request #2`) mais rep�re `Request #4`, comme son POST commence juste apr�s le 33�me octet ou `Request #2`. C'est un peu compliqu�, mais le fait est que l'URL d'attaque ne sera pas d�tect�e par le pare-feu (elle sera interpr�t�e comme le corps d'une requ�te pr�c�dente) mais sera correctement analys�e (et ex�cut�e) par IIS.

Alors que dans le cas susmentionn�, la technique exploite un bogue d'un serveur Web, il existe d'autres sc�narios dans lesquels nous pouvons tirer parti des diff�rentes fa�ons dont diff�rents appareils compatibles HTTP analysent les messages qui ne sont pas conformes � la RFC 1005. Par exemple, le protocole HTTP n'autorise qu'un seul en-t�te Content-Length, mais ne sp�cifie pas comment g�rer un message qui a deux instances de cet en-t�te. Certaines impl�mentations utiliseront la premi�re tandis que d'autres pr�f�reront la seconde, ouvrant la voie aux attaques HTTP Smuggling. Un autre exemple est l'utilisation de l'en-t�te Content-Length dans un message GET.

Notez que HTTP Smuggling n'exploite "*pas*" les vuln�rabilit�s de l'application Web cible. Par cons�quent, il peut �tre quelque peu difficile, dans un engagement de test de p�n�tration, de convaincre le client qu'une contre-mesure doit �tre recherch�e de toute fa�on.

## R�f�rences

### Papiers blanc

- [Amit�Klein, "Diviser pour mieux r�gner�: Fractionnement des r�ponses HTTP, attaques par empoisonnement du cache Web et sujets connexes"](https://packetstormsecurity.com/files/32815/Divide-and-Conquer-HTTP-Response-Splitting-Whitepaper .html)
- [Amit Klein�: "Fractionnement de messages HTTP, contrebande et autres animaux"](https://www.slideserve.com/alicia/http-message-splitting-smuggling-and-other-animals-powerpoint-ppt-presentation)
- [Amit�Klein�: "�Smuggling de requ�tes HTTP - ERRATA (le ph�nom�ne de tampon IIS 48K)"](https://www.securityfocus.com/archive/1/411418)
- [Amit�Klein�: ��Contrebande de r�ponses HTTP��](https://www.securityfocus.com/archive/1/425593)
- [Chaim Linhart, Amit Klein, Ronen Heled, Steve Orrin : � Contrebande de requ�tes HTTP �](https://www.cgisecurity.com/lib/http-request-smuggling.pdf)
