# Test de la contrebande de fractionnement HTTP

|ID |
|------------|
|WSTG-INPV-15|

## Sommaire

Cette section illustre des exemples d'attaques qui exploitent des fonctionnalités spécifiques du protocole HTTP, soit en exploitant les faiblesses de l'application Web, soit des particularités dans la manière dont différents agents interprètent les messages HTTP.
Cette section analysera deux attaques différentes qui ciblent des en-têtes HTTP spécifiques :

- Fractionnement HTTP
- Contrebande HTTP

La première attaque exploite un manque de nettoyage des entrées qui permet à un intrus d'insérer des caractères CR et LF dans les en-têtes de la réponse de l'application et de "diviser" cette réponse en deux messages HTTP différents. L'objectif de l'attaque peut varier d'un empoisonnement du cache à un cross site scripting.

Dans la deuxième attaque, l'attaquant exploite le fait que certains messages HTTP spécialement conçus peuvent être analysés et interprétés de différentes manières selon l'agent qui les reçoit. La contrebande HTTP nécessite un certain niveau de connaissances sur les différents agents qui traitent les messages HTTP (serveur Web, proxy, pare-feu) et ne sera donc inclus que dans la section de test de la boîte grise.

## Objectifs des tests

- Évaluer si l'application est vulnérable au fractionnement, en identifiant quelles attaques possibles sont réalisables.
- Évaluer si la chaîne de communication est vulnérable à la contrebande, en identifiant quelles attaques possibles sont réalisables.

## Comment tester

### Test en boîte noire

#### Fractionnement HTTP

Certaines applications Web utilisent une partie de l'entrée utilisateur pour générer les valeurs de certains en-têtes de leurs réponses. L'exemple le plus simple est fourni par les redirections dans lesquelles l'URL cible dépend d'une valeur soumise par l'utilisateur. Disons par exemple que l'utilisateur est invité à choisir s'il préfère une interface Web standard ou avancée. Le choix sera passé en paramètre qui sera utilisé dans l'entête de la réponse pour déclencher la redirection vers la page correspondante.

Plus précisément, si le paramètre 'interface' a la valeur 'advanced', l'application répondra par ceci :

```http
HTTP/1.1 302 Moved Temporarily
Date: Sun, 03 Dec 2005 16:22:19 GMT
Location: http://victim.com/main.jsp?interface=advanced
<snip>
```

Lors de la réception de ce message, le navigateur amènera l'utilisateur à la page indiquée dans l'en-tête Emplacement. Cependant, si l'application ne filtre pas l'entrée utilisateur, il sera possible d'insérer dans le paramètre 'interface' la séquence %0d%0a, qui représente la séquence CRLF utilisée pour séparer les différentes lignes. À ce stade, les testeurs pourront déclencher une réponse qui sera interprétée comme deux réponses différentes par quiconque l'analysera, par exemple un cache Web situé entre nous et l'application. Cela peut être exploité par un attaquant pour empoisonner ce cache Web afin qu'il fournisse un faux contenu dans toutes les requêtes ultérieures.

Disons que dans l'exemple précédent, le testeur passe les données suivantes comme paramètre d'interface :

`Advanced%0d%0aContent-Length:%200%0d%0a%0d%0aHTTP/1.1%20200%20OK%0d%0aContent-Type:%20text/html%0d%0aContent-Length:%2035%0d%0a% 0d%0a<html>Sorry,%20System%20Down</html>`

La réponse résultante de l'application vulnérable sera donc la suivante :

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

Le cache Web verra deux réponses différentes, donc si l'attaquant envoie, immédiatement après la première requête, une seconde demandant `/index.html`, le cache Web fera correspondre cette requête avec la deuxième réponse et mettra son contenu en cache, donc que toutes les requêtes ultérieures dirigées vers `victim.com/index.html` passant par ce cache Web recevront le message "système en panne". De cette manière, un attaquant serait en mesure de défigurer efficacement le site pour tous les utilisateurs utilisant ce cache Web (l'ensemble d'Internet, si le cache Web est un proxy inverse pour l'application Web).

Alternativement, l'attaquant pourrait transmettre à ces utilisateurs un extrait de code JavaScript qui monte une attaque de script intersite, par exemple pour voler les cookies. Notez que bien que la vulnérabilité soit dans l'application, la cible ici est ses utilisateurs. Par conséquent, afin de rechercher cette vulnérabilité, le testeur doit identifier toutes les entrées contrôlées par l'utilisateur qui influencent un ou plusieurs en-têtes dans la réponse, et vérifier s'il peut y injecter avec succès une séquence CR+LF.

Les en-têtes qui sont les candidats les plus probables pour cette attaque sont :

- `Location`
- `Set-Cookie`

Il faut noter qu'une exploitation réussie de cette vulnérabilité dans un scénario réel peut être assez complexe, car plusieurs facteurs doivent être pris en compte :

1. Le testeur de stylo doit correctement définir les en-têtes dans la fausse réponse pour qu'elle soit mise en cache avec succès (par exemple, un en-tête Last-Modified avec une date définie dans le futur). Ils peuvent également avoir à détruire les versions précédemment mises en cache des pagers cibles, en émettant une requête préliminaire avec "Pragma : no-cache" dans les en-têtes de requête.
2. L'application, bien qu'elle ne filtre pas la séquence CR+LF, peut filtrer d'autres caractères nécessaires à une attaque réussie (par exemple, `<` et `>`). Dans ce cas, le testeur peut essayer d'utiliser d'autres encodages (par exemple, UTF-7)
3. Certaines cibles (par exemple, ASP) coderont en URL la partie chemin de l'en-tête Location (par exemple, `www.victim.com/redirect.asp`), rendant une séquence CRLF inutile. Cependant, ils ne parviennent pas à encoder la section de requête (par exemple, ?interface=advanced), ce qui signifie qu'un point d'interrogation principal suffit pour contourner ce filtrage.

Pour une discussion plus détaillée sur cette attaque et d'autres informations sur les scénarios et applications possibles, consultez les documents référencés au bas de cette section.

### Test en boîte grise

#### Fractionnement HTTP

Une exploitation réussie de HTTP Splitting est grandement facilitée par la connaissance de certains détails de l'application Web et de la cible de l'attaque. Par exemple, différentes cibles peuvent utiliser différentes méthodes pour décider quand le premier message HTTP se termine et quand le second commence. Certains utiliseront les limites de message, comme dans l'exemple précédent. D'autres cibles supposeront que différents messages seront transportés par différents paquets. D'autres alloueront pour chaque message un nombre de morceaux de longueur prédéterminée : dans ce cas, le deuxième message devra commencer exactement au début d'un morceau et cela obligera le testeur à utiliser un bourrage entre les deux messages. Cela peut causer des problèmes lorsque le paramètre vulnérable doit être envoyé dans l'URL, car une URL très longue est susceptible d'être tronquée ou filtrée. Un scénario de boîte grise peut aider l'attaquant à trouver une solution de contournement : plusieurs serveurs d'application, par exemple, autoriseront l'envoi de la requête en utilisant POST au lieu de GET.

#### Contrebande HTTP

Comme mentionné dans l'introduction, HTTP Smuggling exploite les différentes façons dont un message HTTP particulièrement conçu peut être analysé et interprété par différents agents (navigateurs, caches Web, pare-feu d'application). Ce type d'attaque relativement nouveau a été découvert par Chaim Linhart, Amit Klein, Ronen Heled et Steve Orrin en 2005. Il existe plusieurs applications possibles et nous analyserons l'une des plus spectaculaires : le contournement d'un pare-feu applicatif. Reportez-vous au livre blanc d'origine (lié au bas de cette page) pour des informations plus détaillées et d'autres scénarios.

##### Contournement du pare-feu applicatif

Il existe plusieurs produits qui permettent à une administration système de détecter et de bloquer une requête Web hostile en fonction d'un schéma malveillant connu intégré à la requête. Par exemple, considérez l'infâme et ancienne [attaque de traversée de répertoire Unicode contre le serveur IIS](https://www.securityfocus.com/bid/1806), dans laquelle un attaquant pourrait casser la racine www en émettant une requête comme :

`http://target/scripts/..%c1%1c../winnt/system32/cmd.exe?/c+<command_to_execute>`

Bien sûr, il est assez facile de repérer et de filtrer cette attaque par la présence de chaînes comme ".." et "cmd.exe" dans l'URL. Cependant, IIS 5.0 est assez pointilleux sur les requêtes POST dont le corps est jusqu'à 48 Ko et tronque tout le contenu qui dépasse cette limite lorsque l'en-tête Content-Type est différent de application/x-www-form-urlencoded. Le pen-testeur peut en tirer parti en créant une requête très volumineuse, structurée comme suit :

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

Ce qui se passe ici est que la `Request #1` est composée de 49223 octets, ce qui inclut également les lignes de `Request #2`. Par conséquent, un pare-feu (ou tout autre agent à côté d'IIS 5.0) verra la requête n° 1, ne verra pas la requête n° 2 (ses données ne feront partie que de la n° 1), verra la requête n° 3 et ratera ` Requête #4` (parce que le POST ne sera qu'une partie du faux en-tête xxxx).

Maintenant, qu'arrive-t-il à IIS 5.0 ? Il arrêtera d'analyser `Request #1` juste après les 49152 octets de déchets (car il aura atteint la limite de 48K = 49152 octets) et analysera donc `Request #2` comme une nouvelle demande distincte. `Request #2` prétend que son contenu est de 33 octets, ce qui inclut tout jusqu'à "xxxx: ", ce qui fait que IIS manque `Request #3` (interprété comme faisant partie de `Request #2`) mais repère `Request #4`, comme son POST commence juste après le 33ème octet ou `Request #2`. C'est un peu compliqué, mais le fait est que l'URL d'attaque ne sera pas détectée par le pare-feu (elle sera interprétée comme le corps d'une requête précédente) mais sera correctement analysée (et exécutée) par IIS.

Alors que dans le cas susmentionné, la technique exploite un bogue d'un serveur Web, il existe d'autres scénarios dans lesquels nous pouvons tirer parti des différentes façons dont différents appareils compatibles HTTP analysent les messages qui ne sont pas conformes à la RFC 1005. Par exemple, le protocole HTTP n'autorise qu'un seul en-tête Content-Length, mais ne spécifie pas comment gérer un message qui a deux instances de cet en-tête. Certaines implémentations utiliseront la première tandis que d'autres préféreront la seconde, ouvrant la voie aux attaques HTTP Smuggling. Un autre exemple est l'utilisation de l'en-tête Content-Length dans un message GET.

Notez que HTTP Smuggling n'exploite "*pas*" les vulnérabilités de l'application Web cible. Par conséquent, il peut être quelque peu difficile, dans un engagement de test de pénétration, de convaincre le client qu'une contre-mesure doit être recherchée de toute façon.

## Références

### Papiers blanc

- [Amit Klein, "Diviser pour mieux régner : Fractionnement des réponses HTTP, attaques par empoisonnement du cache Web et sujets connexes"](https://packetstormsecurity.com/files/32815/Divide-and-Conquer-HTTP-Response-Splitting-Whitepaper .html)
- [Amit Klein : "Fractionnement de messages HTTP, contrebande et autres animaux"](https://www.slideserve.com/alicia/http-message-splitting-smuggling-and-other-animals-powerpoint-ppt-presentation)
- [Amit Klein : " Smuggling de requêtes HTTP - ERRATA (le phénomène de tampon IIS 48K)"](https://www.securityfocus.com/archive/1/411418)
- [Amit Klein : « Contrebande de réponses HTTP »](https://www.securityfocus.com/archive/1/425593)
- [Chaim Linhart, Amit Klein, Ronen Heled, Steve Orrin : « Contrebande de requêtes HTTP »](https://www.cgisecurity.com/lib/http-request-smuggling.pdf)
