# Énumérer les applications sur le serveur Web

|ID          |
|------------|
|WSTG-INFO-04|

## Sommaire

Une étape primordiale dans le test des vulnérabilités des applications Web consiste à déterminer quelles applications particulières sont hébergées sur un serveur Web. De nombreuses applications ont des vulnérabilités connues et des stratégies d'attaque connues qui peuvent être exploitées afin de prendre le contrôle à distance ou d'exploiter des données. De plus, de nombreuses applications sont souvent mal configurées ou non mises à jour, en raison de la perception qu'elles ne sont utilisées qu'en "interne" et qu'il n'existe donc aucune menace.
Avec la prolifération des serveurs Web virtuels, la relation traditionnelle de type 1:1 entre une adresse IP et un serveur Web perd une grande partie de sa signification d'origine. Il n'est pas rare d'avoir plusieurs sites Web ou applications dont les noms symboliques correspondent à la même adresse IP. Ce scénario ne se limite pas aux environnements d'hébergement, mais s'applique également aux environnements d'entreprise ordinaires.

Les professionnels de la sécurité reçoivent parfois un ensemble d'adresses IP comme cible à tester. On peut soutenir que ce scénario s'apparente davantage à un engagement de type test d'intrusion, mais dans tous les cas, on s'attend à ce qu'une telle mission teste toutes les applications Web accessibles via cette cible. Le problème est que l'adresse IP donnée héberge un service HTTP sur le port 80, mais si un testeur doit y accéder en spécifiant l'adresse IP (qui est tout ce qu'il sait), il signale "Aucun serveur Web configuré à cette adresse" ou un message similaire . Mais ce système pourrait "cacher" un certain nombre d'applications Web, associées à des noms symboliques (DNS) sans rapport. De toute évidence, l'étendue de l'analyse est profondément affectée par le fait que le testeur teste toutes les applications ou ne teste que les applications dont il a connaissance.

Parfois, la spécification cible est plus riche. Le testeur peut recevoir une liste d'adresses IP et leurs noms symboliques correspondants. Néanmoins, cette liste peut transmettre des informations partielles, c'est-à-dire qu'elle peut omettre certains noms symboliques et que le client peut même ne pas en être conscient (cela est plus susceptible de se produire dans les grandes organisations).

D'autres problèmes affectant la portée de l'évaluation sont représentés par des applications Web publiées sur des URL non évidentes (par exemple, `http://www.exemple.com/some-strange-URL`), qui ne sont référencées nulle part ailleurs. Cela peut se produire soit par erreur (en raison de mauvaises configurations), soit intentionnellement (par exemple, des interfaces administratives non annoncées).

Pour résoudre ces problèmes, il est nécessaire d'effectuer une découverte d'applications Web.

## Objectifs des tests

- Énumérer les applications dans la portée qui existent sur un serveur Web.

## Comment tester

La découverte d'applications Web est un processus visant à identifier les applications Web sur une infrastructure donnée. Ce dernier est généralement spécifié comme un ensemble d'adresses IP (peut-être un bloc réseau), mais peut consister en un ensemble de noms symboliques DNS ou un mélange des deux. Ces informations sont distribuées avant l'exécution d'une évaluation, qu'il s'agisse d'un test d'intrusion de style classique ou d'une évaluation axée sur l'application. Dans les deux cas, sauf indication contraire dans les règles d'engagement (par exemple, tester uniquement l'application située à l'URL `http://www.exemple.com/`), l'évaluation doit s'efforcer d'être la plus complète possible, c'est-à-dire qu'elle doit identifier toutes les applications accessibles via la cible donnée. Les exemples suivants examinent quelques techniques qui peuvent être employées pour atteindre cet objectif.

> Certaines des techniques suivantes s'appliquent aux serveurs Web accessibles sur Internet, à savoir les services de recherche Web DNS et IP inversé et l'utilisation de moteurs de recherche. Les exemples utilisent des adresses IP privées (telles que `192.168.1.100`), qui, sauf indication contraire, représentent des adresses IP *génériques* et sont utilisées uniquement à des fins d'anonymat.

Trois facteurs influencent le nombre d'applications liées à un nom DNS donné (ou à une adresse IP) :

1. **URL de base différente**

    Le point d'entrée évident pour une application Web est `www.exemple.com`, c'est-à-dire qu'avec cette notation abrégée, nous pensons à l'application Web provenant de `http://www.exemple.com/` (la même chose s'applique pour HTTPS) . Cependant, même s'il s'agit de la situation la plus courante, rien n'oblige l'application à démarrer à `/`.

    Par exemple, le même nom symbolique peut être associé à trois applications Web telles que : `http://www.exemple.com/url1` `http://www.exemple.com/url2` `http://www.exemple.com/url3`

    Dans ce cas, l'URL `http://www.exemple.com/` ne serait pas associée à une page significative, et les trois applications seraient **masquées**, à moins que le testeur ne sache explicitement comment les atteindre, c'est-à-dire , le testeur connaît *url1*, *url2* ou *url3*. Il n'est généralement pas nécessaire de publier des applications Web de cette manière, à moins que le propriétaire ne souhaite pas qu'elles soient accessibles de manière standard et soit prêt à informer les utilisateurs de leur emplacement exact. Cela ne signifie pas que ces applications sont secrètes, simplement que leur existence et leur emplacement ne sont pas explicitement annoncés.

2. **Ports non standard**

    Alors que les applications Web vivent généralement sur les ports 80 (HTTP) et 443 (HTTPS), ces numéros de port n'ont rien de magique. En fait, les applications Web peuvent être associées à des ports TCP arbitraires et peuvent être référencées en spécifiant le numéro de port comme suit : `http[s]://www.exemple.com:port/`. Par exemple, `http://www.exemple.com:20000/`.

3. **Hôtes virtuels**

    Le DNS permet d'associer une seule adresse IP à un ou plusieurs noms symboliques. Par exemple, l'adresse IP `192.168.1.100` peut être associée aux noms DNS `www.exemple.com`, `helpdesk.exemple.com`, `webmail.exemple.com`. Il n'est pas nécessaire que tous les noms appartiennent au même domaine DNS. Cette relation 1 à N peut être reflétée pour servir un contenu différent en utilisant ce que l'on appelle des hôtes virtuels. Les informations spécifiant l'hôte virtuel auquel nous faisons référence sont intégrées dans l'en-tête [Host header] HTTP 1.1 (https://tools.ietf.org/html/rfc2616#section-14.23).

    On ne soupçonnerait pas l'existence d'autres applications Web en plus de l'évident `www.exemple.com`, à moins de connaître `helpdesk.exemple.com` et `webmail.exemple.com`.

### Approches pour résoudre le problème 1 : URL non standard

Il n'existe aucun moyen de s'assurer pleinement de l'existence d'applications Web dont le nom n'est pas standard. Étant non standard, il n'y a pas de critères fixes régissant la convention de dénomination, mais il existe un certain nombre de techniques que le testeur peut utiliser pour obtenir des informations supplémentaires.

Tout d'abord, si le serveur Web est mal configuré et autorise la navigation dans les répertoires, il peut être possible de repérer ces applications. Les scanners de vulnérabilité peuvent aider à cet égard.

Deuxièmement, ces applications peuvent être référencées par d'autres pages Web et il est possible qu'elles aient été analysées et indexées par les moteurs de recherche Web. Si les testeurs soupçonnent l'existence de telles applications **cachées** sur `www.exemple.com`, ils peuvent effectuer une recherche à l'aide de l'opérateur *site* et examiner le résultat d'une requête pour `site: www.exemple.com`. Parmi les URL renvoyées, il pourrait y en avoir une pointant vers une telle application non évidente.

Une autre option consiste à rechercher des URL qui pourraient être des candidats probables pour des applications non publiées. Par exemple, un frontal de messagerie Web peut être accessible à partir d'URL telles que `https://www.exemple.com/webmail`, `https://webmail.exemple.com/` ou `https://mail.exemple.com/`. Il en va de même pour les interfaces d'administration, qui peuvent être publiées sur des URL masquées (par exemple, une interface d'administration Tomcat), et pourtant référencées nulle part. Donc, faire un peu de recherche de style dictionnaire (ou "devinette intelligente") pourrait donner des résultats. Les scanners de vulnérabilité peuvent aider à cet égard.

### Approches pour résoudre le problème 2 - Ports non standard

Il est facile de vérifier l'existence d'applications Web sur des ports non standard. Un analyseur de port tel que Nmap est capable d'effectuer une reconnaissance de service au moyen de l'option `-sV` et identifiera les services http[s] sur des ports arbitraires. Ce qui est requis est une analyse complète de l'ensemble de l'espace d'adressage du port TCP 64k.

Par exemple, la commande suivante recherchera, avec une analyse de connexion TCP, tous les ports ouverts sur l'IP `192.168.1.100` et essaiera de déterminer quels services leur sont liés (seuls les commutateurs *essentiels* sont affichés - Nmap dispose d'un large ensemble d'options, dont la discussion est hors de propos) :

`nmap –Pn –sT –sV –p0-65535 192.168.1.100`

Il suffit d'examiner la sortie et de rechercher HTTP ou l'indication de services encapsulés dans TLS (qui doivent être sondés pour confirmer qu'ils sont HTTPS). Par exemple, la sortie de la commande précédente pourrait ressembler à :

```bash
Interesting ports on 192.168.1.100:
(The 65527 ports scanned but not shown below are in state: closed)
PORT      STATE SERVICE     VERSION
22/tcp    open  ssh         OpenSSH 3.5p1 (protocol 1.99)
80/tcp    open  http        Apache httpd 2.0.40 ((Red Hat Linux))
443/tcp   open  ssl         OpenSSL
901/tcp   open  http        Samba SWAT administration server
1241/tcp  open  ssl         Nessus security scanner
3690/tcp  open  unknown
8000/tcp  open  http-alt?
8080/tcp  open  http        Apache Tomcat/Coyote JSP engine 1.1
```

A partir de cet exemple, on voit que :

- Il existe un serveur HTTP Apache en cours d'exécution sur le port 80.
- Il semble qu'il y ait un serveur HTTPS sur le port 443 (mais cela doit être confirmé, par exemple, en visitant `https://192.168.1.100` avec un navigateur).
- Sur le port 901, il y a une interface Web Samba SWAT.
- Le service sur le port 1241 n'est pas HTTPS, mais est le démon Nessus enveloppé dans TLS.
- Le port 3690 propose un service non spécifié (Nmap renvoie son *empreinte digitale* - ici omise pour plus de clarté - ainsi que des instructions pour la soumettre pour incorporation dans la base de données d'empreintes digitales Nmap, à condition que vous sachiez quel service il représente).
- Un autre service non spécifié sur le port 8000 ; il peut s'agir de HTTP, car il n'est pas rare de trouver des serveurs HTTP sur ce port. Examinons ce problème :

```bash
$ telnet 192.168.10.100 8000
Trying 192.168.1.100...
Connected to 192.168.1.100.
Escape character is '^]'.
GET / HTTP/1.0

HTTP/1.0 200 OK
pragma: no-cache
Content-Type: text/html
Server: MX4J-HTTPD/1.0
expires: now
Cache-Control: no-cache

<html>
...
```

Cela confirme qu'il s'agit bien d'un serveur HTTP. Alternativement, les testeurs auraient pu visiter l'URL avec un navigateur Web ; ou utilisé les commandes GET ou HEAD Perl, qui imitent les interactions HTTP telles que celle donnée ci-dessus (cependant, les requêtes HEAD peuvent ne pas être honorées par tous les serveurs).

- Apache Tomcat fonctionnant sur le port 8080.

La même tâche peut être effectuée par des scanners de vulnérabilité, mais vérifiez d'abord que le scanner de votre choix est capable d'identifier les services HTTP[S] s'exécutant sur des ports non standard. Par exemple, Nessus est capable de les identifier sur des ports arbitraires (à condition qu'il soit chargé de scanner tous les ports), et fournira, vis-à-vis de Nmap, un certain nombre de tests sur les vulnérabilités connues des serveurs Web, ainsi que sur le TLS/ Configuration SSL des services HTTPS. Comme indiqué précédemment, Nessus est également capable de repérer des applications ou des interfaces Web populaires qui pourraient autrement passer inaperçues (par exemple, une interface administrative Tomcat).

### Approches pour résoudre le problème 3 - Hôtes virtuels

Il existe un certain nombre de techniques qui peuvent être utilisées pour identifier les noms DNS associés à une adresse IP donnée "x.y.z.t".

#### Transferts de zone DNS

Cette technique a une utilisation limitée de nos jours, compte tenu du fait que les transferts de zone ne sont en grande partie pas honorés par les serveurs DNS. Cependant, cela peut valoir la peine d'essayer. Tout d'abord, les testeurs doivent déterminer les serveurs de noms servant `x.y.z.t`. Si un nom symbolique est connu pour `x.y.z.t` (soit `www.exemple.com`), ses serveurs de noms peuvent être déterminés au moyen d'outils tels que `nslookup`, `host` ou `dig`, en demandant Enregistrements DNS NS.

Si aucun nom symbolique n'est connu pour `x.y.z.t`, mais que la définition cible contient au moins un nom symbolique, les testeurs peuvent essayer d'appliquer le même processus et interroger le serveur de noms de ce nom (en espérant que `x.y.z.t` sera également servi par ce serveur de noms). Par exemple, si la cible est constituée de l'adresse IP `x.y.z.t` et du nom `mail.exemple.com`, déterminez les serveurs de noms pour le domaine `exemple.com`.

L'exemple suivant montre comment identifier les serveurs de noms pour `www.owasp.org` en utilisant la commande `host` :

```bash
$ host -t ns www.owasp.org
www.owasp.org is an alias for owasp.org.
owasp.org name server ns1.secure.net.
owasp.org name server ns2.secure.net.
```

Un transfert de zone peut maintenant être demandé aux serveurs de noms pour le domaine `exemple.com`. Si le testeur a de la chance, il récupérera une liste des entrées DNS pour ce domaine. Cela inclura l'évident `www.exemple.com` et les moins évidents `helpdesk.exemple.com` et `webmail.exemple.com` (et éventuellement d'autres). Vérifiez tous les noms renvoyés par le transfert de zone et considérez tous ceux qui sont liés à la cible en cours d'évaluation.

Essayer de demander un transfert de zone pour `owasp.org` depuis l'un de ses serveurs de noms :

```bash
$ host -l www.owasp.org ns1.secure.net
Using domain server:
Name: ns1.secure.net
Address: 192.220.124.10#53
Aliases:

Host www.owasp.org not found: 5(REFUSED)
; Transfer failed.
```

#### Requêtes DNS inverses

Ce processus est similaire au précédent, mais repose sur des enregistrements DNS inverses (PTR). Plutôt que de demander un transfert de zone, essayez de définir le type d'enregistrement sur PTR et lancez une requête sur l'adresse IP donnée. Si les testeurs ont de la chance, ils peuvent récupérer une entrée de nom DNS. Cette technique repose sur l'existence de mappages IP-nom symbolique, ce qui n'est pas garanti.

#### Recherches DNS sur le Web

Ce type de recherche s'apparente au transfert de zone DNS, mais s'appuie sur des services Web qui permettent des recherches DNS basées sur le nom. L'un de ces services est le service [Netcraft Search DNS](https://searchdns.netcraft.com/?host). Le testeur peut interroger une liste de noms appartenant au domaine de votre choix, comme "exemple.com". Ensuite, ils vérifieront si les noms qu'ils ont obtenus sont pertinents pour la cible qu'ils examinent.

#### Services IP inversés

Les services d'IP inversé sont similaires aux requêtes inverses DNS, à la différence que les testeurs interrogent une application Web au lieu d'un serveur de noms. Il existe un certain nombre de ces services disponibles. Puisqu'ils ont tendance à renvoyer des résultats partiels (et souvent différents), il est préférable d'utiliser plusieurs services pour obtenir une analyse plus complète.

- [IP inversée MxToolbox] (https://mxtoolbox.com/ReverseLookup.aspx)
- [DNSstuff](https://www.dnsstuff.com/) (plusieurs services disponibles)
- [Net Square](https://web.archive.org/web/20190515092354/http://www.net-square.com/mspawn.html) (plusieurs requêtes sur les domaines et adresses IP, nécessite une installation)

#### Googler

Suite à la collecte d'informations à partir des techniques précédentes, les testeurs peuvent s'appuyer sur les moteurs de recherche pour éventuellement affiner et incrémenter leur analyse. Cela peut fournir des preuves de noms symboliques supplémentaires appartenant à la cible ou d'applications accessibles via des URL non évidentes.

Par exemple, en considérant l'exemple précédent concernant `www.owasp.org`, le testeur pourrait interroger Google et d'autres moteurs de recherche à la recherche d'informations (donc, des noms DNS) liées aux domaines nouvellement découverts de `webgoat.org`, `webscarab. com` et `webscarab.net`.

Les techniques de recherche sur Google sont expliquées dans [Google Hacking, ou Dorking](01-Conduct_Search_Engine_Discovery_Reconnaissance_for_Information_Leakage.md).

#### Certificats numériques

Si le serveur accepte les connexions via HTTPS, le nom commun (CN) et le nom alternatif du sujet (SAN) sur le certificat peuvent contenir un ou plusieurs noms d'hôte. Cependant, si le serveur Web n'a pas de certificat de confiance ou si des caractères génériques sont utilisés, cela peut ne pas renvoyer d'informations valides.

Le CN et le SAN peuvent être obtenus en inspectant manuellement le certificat ou via d'autres outils tels que OpenSSL :

```sh
openssl s_client -connect 93.184.216.34:443 </dev/null 2>/dev/null | openssl x509 -noout -text | grep -E 'DNS:|Subject:'

Subject: C = US, ST = California, L = Los Angeles, O = Internet Corporation for Assigned Names and Numbers, CN = www.example.org
DNS:www.example.org, DNS:exemple.com, DNS:example.edu, DNS:example.net, DNS:example.org, DNS:www.exemple.com, DNS:www.example.edu, DNS:www.example.net
```

## Outils

- Outils de recherche DNS tels que `nslookup`, `dig` et similaires.
- Moteurs de recherche (Google, Bing et autres moteurs de recherche majeurs).
- Service de recherche Web spécialisé lié au DNS : voir texte.
- [Nmap](https://nmap.org/)
- [Scanner de vulnérabilité Nessus](https://www.tenable.com/products/nessus)
- [Nikto](https://www.cirt.net/nikto2)
