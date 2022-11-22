# Architecture des applications cartographiques

|ID          |
|------------|
|WSTG-INFO-10|

## Sommaire

Afin de tester efficacement une application et d'�tre en mesure de fournir des recommandations significatives sur la mani�re de r�soudre les probl�mes identifi�s, il est important de comprendre ce que vous testez r�ellement. De plus, cela peut aider � d�terminer si des composants sp�cifiques doivent �tre consid�r�s comme hors de port�e des tests.

Les applications Web modernes peuvent varier consid�rablement en complexit�, allant d'un simple script ex�cut� sur un seul serveur � une application tr�s complexe r�partie sur des dizaines de syst�mes, langages et composants diff�rents. Il peut �galement y avoir des composants suppl�mentaires au niveau du r�seau tels que des pare-feu ou des syst�mes de protection contre les intrusions qui peuvent avoir un impact significatif sur les tests.

## Objectifs des tests

- Comprendre l'architecture de l'application et les technologies utilis�es.

## Comment tester

Lorsque vous testez du point de vue de la bo�te noire, il est important d'essayer de cr�er une image claire du fonctionnement de l'application et des technologies et composants en place. Dans certains cas, il est possible de tester des composants sp�cifiques (comme un pare-feu d'application Web), tandis que d'autres peuvent �tre identifi�s en inspectant le comportement de l'application.

Les sections ci-dessous fournissent une vue d'ensemble de haut niveau des composants architecturaux courants, ainsi que des d�tails sur la fa�on dont ils peuvent �tre identifi�s.

### Application Components

#### Web Server

Simple applications may run on a single server, which can be identified using the steps discussed in the [Fingerprint Web Server](02-Fingerprint_Web_Server.md) section of the guide.

#### Platform-as-a-Service (PaaS)

In a Platform-as-a-Service (PaaS) model, the web server and underlying infrastructure are managed by the service provider, and the customer is only responsible for the application that this deployed on them. From a testing perspective, there are two main differences:

- The application owner has no access to the underlying infrastructure, so will be unable to directly remediate any issues.
- Infrastructure testing is likely to be out of scope for any engagements.

In some cases it is possible to identify the use of PaaS, as the application may use a specific domain name (for example, applications deployed on Azure App Services will have a `*.azurewebsites.net` domain - although they may also use custom domains). However, in other cases it is difficult to determine whether PaaS is in use.

#### Serverless

In a Serverless model, the developers provide code which is directly run on a hosting platform as individual functions, rather than as an traditional larger web application deployed in a webroot. This makes it well suited to microservice-based architectures. As with a PaaS environment, infrastructure testing is likely to be out of scope.

In some cases the use of Serverless code may be indicated by the presence of specific HTTP headers. For example, AWS Lambda functions will typically return the following headers:

```http
X-Amz-Invocation-Type
X-Amz-Log-Type
X-Amz-Client-Context
```

Les fonctions Azure sont moins �videntes. Ils renvoient g�n�ralement l'en-t�te `Server: Kestrel` - mais cela ne suffit pas en soi pour �tre s�r qu'il s'agit d'une fonction Azure App, plut�t que d'un autre code ex�cut� sur Kestrel.

####�Microservices

Dans une architecture bas�e sur des microservices, l'API de l'application est compos�e de plusieurs services discrets, plut�t que de s'ex�cuter comme une application monolithique. Les services eux-m�mes s'ex�cutent souvent dans des conteneurs (g�n�ralement avec Kubernetes) et peuvent utiliser une vari�t� de syst�mes d'exploitation et de langages diff�rents. Bien qu'ils se trouvent g�n�ralement derri�re une passerelle API et un domaine uniques, l'utilisation de plusieurs langues (souvent indiqu�e dans des messages d'erreur d�taill�s) peut sugg�rer que des microservices sont utilis�s.

#### Stockage statique

De nombreuses applications stockent le contenu statique sur des plates-formes de stockage d�di�es, plut�t que de l'h�berger directement sur le serveur Web principal. Les deux plates-formes les plus courantes sont les compartiments S3 d'Amazon et les comptes de stockage d'Azure, et peuvent �tre facilement identifi�es par les noms de domaine�:

- Les compartiments Amazon S3 sont soit `BUCKET.s3.amazonaws.com` ou `s3.REGION.amazonaws.com/BUCKET`
- Les comptes de stockage Azure sont `ACCOUNT.blob.core.windows.net`

Ces comptes de stockage peuvent souvent exposer des fichiers sensibles, comme indiqu� dans la section [Guide de test du stockage dans le cloud](../02-Configuration_and_Deployment_Management_Testing/11-Test_Cloud_Storage.md).

#### Base de donn�es

La plupart des applications Web non triviales utilisent une sorte de base de donn�es pour stocker du contenu dynamique. Dans certains cas, il est possible de d�terminer la base de donn�es, bien qu'elle repose g�n�ralement sur d'autres probl�mes de l'application. Cela peut souvent �tre fait par :

- Port scannant le serveur et recherchant tous les ports ouverts associ�s � des bases de donn�es sp�cifiques.
- D�clenchement de messages d'erreur li�s � SQL (ou NoSQL) (ou recherche d'erreurs existantes � partir d'un [moteur de recherche] (../01-Information_Gathering/01-Conduct_Search_Engine_Discovery_Reconnaissance_for_Information_Leakage.md).

Lorsqu'il n'est pas possible de d�terminer de mani�re concluante la base de donn�es, vous pouvez souvent faire une supposition �clair�e bas�e sur d'autres aspects de l'application�:

- Windows, IIS et ASP.NET utilisent souvent le serveur Microsoft SQL.
- Les syst�mes embarqu�s utilisent souvent SQLite.
- PHP utilise souvent MySQL ou PostgreSQL.
- APEX utilise souvent Oracle.

Ce ne sont pas des r�gles strictes, mais elles peuvent certainement vous donner un point de d�part raisonnable si aucune meilleure information n'est disponible.

#### Authentification

La plupart des applications ont une certaine forme d'authentification pour les utilisateurs. Plusieurs back-ends d'authentification peuvent �tre utilis�s, tels que�:

- Configuration du serveur Web (y compris les fichiers `.htaccess`) ou codage en dur des mots de passe dans les scripts.
    - S'affiche g�n�ralement en tant qu'authentification HTTP Basic, indiqu�e par une fen�tre contextuelle dans le navigateur et un en-t�te HTTP "WWW-Authenticate�: Basic".
- Comptes d'utilisateurs locaux dans une base de donn�es.
    - G�n�ralement int�gr� dans un formulaire ou un point de terminaison API sur l'application.
- Une source d'authentification centrale existante comme Active Directory ou un serveur LDAP.
    - Peut utiliser l'authentification NTLM, indiqu�e par un en-t�te HTTP "WWW-Authenticate�: NTLM".
    - Peut �tre int�gr� � l'application web dans un formulaire.
    - Peut exiger que le nom d'utilisateur soit saisi au format � DOMAINE\nom d'utilisateur �, ou peut donner une liste d�roulante des domaines disponibles.
- Single Sign-On (SSO) avec un fournisseur interne ou externe.
    - Utilise g�n�ralement OAuth, OpenID Connect ou SAML.

Les applications peuvent fournir plusieurs options � l'utilisateur pour s'authentifier (telles que l'enregistrement d'un compte local ou l'utilisation de son compte Facebook existant) et peuvent utiliser diff�rents m�canismes pour les utilisateurs normaux et les administrateurs.

####�Services et API tiers

Presque toutes les applications Web incluent des ressources tierces qui sont charg�es ou avec lesquelles le client interagit. Ceux-ci peuvent inclure :

- [Contenu actif](https://developer.mozilla.org/en-US/docs/Web/Security/Mixed_content#mixed_active_content) (comme les scripts, les feuilles de style, les polices et les iframes).
- [Contenu passif](https://developer.mozilla.org/en-US/docs/Web/Security/Mixed_content#mixed_passivedisplay_content) (comme des images et des vid�os).
- API externes.
- Boutons de m�dias sociaux.
- R�seaux publicitaires.
- Passerelles de paiement.

Ces ressources sont demand�es directement par le navigateur de l'utilisateur, elles peuvent donc �tre facilement identifi�es � l'aide des outils de d�veloppement ou d'un proxy d'interception. Bien qu'il soit important de les identifier (car ils peuvent avoir un impact sur la s�curit� de l'application), n'oubliez pas qu'ils sont g�n�ralement hors de port�e des tests, car ils appartiennent � des tiers.

### Composants r�seau

#### Proxy inverse

Un proxy inverse se trouve devant un ou plusieurs serveurs principaux et redirige les requ�tes vers la destination appropri�e. Ils peuvent impl�menter diverses fonctionnalit�s, telles que�:

- Agir en tant que [load balancer](#load-balancer) ou [web application firewall](#web-application-firewall-waf).
- Permettre � plusieurs applications d'�tre h�berg�es sur une seule adresse IP ou un seul domaine (dans des sous-dossiers).
- Mettre en place un filtrage IP ou d'autres restrictions.
- Mise en cache du contenu du back-end pour am�liorer les performances.

Il n'est pas toujours possible de d�tecter un reverse proxy (surtout s'il n'y a qu'une application derri�re), mais on peut souvent parfois l'identifier par :

- Une incompatibilit� entre le serveur frontal et l'application principale (comme un en-t�te `Server: nginx` avec une application ASP.NET).
    - Cela peut parfois entra�ner des [vuln�rabilit�s de contrebande de requ�tes](https://portswigger.net/web-security/request-smuggling).
- En-t�tes en double (surtout l'en-t�te `Server`).
- Plusieurs applications h�berg�es sur la m�me adresse IP ou le m�me domaine (surtout si elles utilisent des langues diff�rentes).

#### �quilibreur de charge

Un �quilibreur de charge se trouve devant plusieurs serveurs principaux et alloue les requ�tes entre eux afin de fournir une plus grande redondance et une plus grande capacit� de traitement pour l'application.

Les �quilibreurs de charge peuvent �tre difficiles � d�tecter, mais peuvent parfois �tre identifi�s en faisant plusieurs requ�tes et en examinant les r�ponses pour les diff�rences, telles que�:

- Heures syst�me incoh�rentes.
- Diff�rentes adresses IP internes ou noms d'h�te dans les messages d'erreur d�taill�s.
- Diff�rentes adresses renvoy�es par [Server-Side Request Forgery (SSRF)](../07-Input_Validation_Testing/19-Testing_for_Server-Side_Request_Forgery.md).

Ils peuvent �galement �tre signal�s par la pr�sence de cookies sp�cifiques (par exemple, les �quilibreurs de charge F5 BIG-IP cr�eront un cookie appel� `BIGipServer`.

####�R�seau de diffusion de contenu (CDN)

Un r�seau de diffusion de contenu (CDN) est un ensemble g�ographiquement distribu� de serveurs proxy de mise en cache, con�u pour am�liorer les performances du site Web afin de fournir une r�silience suppl�mentaire � un site Web.

Il est g�n�ralement configur� en pointant le domaine public vers les serveurs du CDN, puis en configurant le CDN pour qu'il se connecte aux serveurs principaux corrects (parfois appel�s "l'origine").

Le moyen le plus simple de d�tecter un CDN consiste � effectuer une recherche WHOIS pour les adresses IP auxquelles le domaine se r�sout. S'ils appartiennent � une soci�t� CDN (comme Akamai, Cloudflare ou Fastly - voir [Wikipedia](https://en.wikipedia.org/wiki/Content_delivery_network#Notable_content_delivery_service_providers) pour une liste plus compl�te), alors c'est comme si un CDN �tait utilis�.

Lorsque vous testez un site derri�re un CDN, vous devez garder � l'esprit les points suivants�:

- Les adresses IP et les serveurs appartiennent au fournisseur de CDN et sont susceptibles d'�tre hors de port�e des tests d'infrastructure.
- De nombreux CDN incluent �galement des fonctionnalit�s telles que la d�tection de robots, la limitation de d�bit et les pare-feu d'applications Web.
- Les CDN mettent g�n�ralement en cache le contenu, de sorte que toute modification apport�e au site Web principal peut ne pas appara�tre imm�diatement.

Si le site se trouve derri�re un CDN, il peut �tre utile d'identifier les serveurs principaux. S'ils ne disposent pas d'un contr�le d'acc�s appropri�, vous pourrez peut-�tre contourner le CDN (et toutes les protections qu'il offre) en acc�dant directement aux serveurs principaux. Il existe une vari�t� de m�thodes diff�rentes qui peuvent vous permettre d'identifier le syst�me back-end�:

- Les e-mails envoy�s par l'application peuvent provenir directement du serveur principal, ce qui pourrait r�v�ler son adresse IP.
- Le broyage DNS, les transferts de zone ou les listes de transparence des certificats pour le domaine peuvent le r�v�ler sur un sous-domaine.
- L'analyse des plages d'adresses IP connues pour �tre utilis�es par l'entreprise peut trouver le serveur principal.
- L'exploitation de [Server-Side Request Forgery (SSRF)](../07-Input_Validation_Testing/19-Testing_for_Server-Side_Request_Forgery.md) peut r�v�ler l'adresse IP.
- Des messages d'erreur d�taill�s de l'application peuvent exposer des adresses IP ou des noms d'h�te.

### Composants de s�curit�

#### Pare-feu r�seau

La plupart des serveurs Web seront prot�g�s par un pare-feu de filtrage de paquets ou d'inspection dynamique, qui bloque tout trafic r�seau non requis. Pour le d�tecter, effectuez une analyse des ports du serveur et examinez les r�sultats.

Si la majorit� des ports sont affich�s comme "ferm�s" (c'est-�-dire qu'ils renvoient un paquet "RST" en r�ponse au paquet "SYN" initial), cela sugg�re que le serveur n'est peut-�tre pas prot�g� par un pare-feu. Si les ports sont affich�s comme "filtr�s" (c'est-�-dire qu'aucune r�ponse n'est re�ue lors de l'envoi d'un paquet "SYN" vers un port inutilis�), un pare-feu est tr�s probablement en place.

De plus, si des services inappropri�s sont expos�s au monde (tels que SMTP, IMAP, MySQL, etc.), cela sugg�re soit qu'il n'y a pas de pare-feu en place, soit que le pare-feu est mal configur�.

#### Syst�me de d�tection et de pr�vention des intrusions sur le r�seau

Un syst�me de d�tection d'intrusion (IDS) r�seau d�tecte les activit�s suspectes ou malveillantes au niveau du r�seau (telles que l'analyse des ports ou des vuln�rabilit�s) et d�clenche des alertes. Un syst�me de pr�vention des intrusions (IPS) est similaire, mais prend �galement des mesures pour emp�cher l'activit� - g�n�ralement en bloquant l'adresse IP source.

Un IPS peut g�n�ralement �tre d�tect� en ex�cutant des outils d'analyse automatis�s (tels qu'un analyseur de ports) sur la cible et en v�rifiant si l'adresse IP source est bloqu�e. Cependant, de nombreux outils au niveau de l'application peuvent ne pas �tre d�tect�s par un IPS (surtout s'il ne d�chiffre pas TLS).

#### Pare-feu d'application Web (WAF)

Un pare-feu d'application Web (WAF) inspecte le contenu des requ�tes HTTP et bloque celles qui semblent suspectes ou malveillantes, ou applique dynamiquement d'autres contr�les tels que CAPTCHA ou la limitation du d�bit. Ils sont g�n�ralement bas�s sur un ensemble de mauvaises signatures et d'expressions r�guli�res connues, telles que [OWASP Core Rule Set](https://owasp.org/www-project-modsecurity-core-rule-set/). Les WAF peuvent �tre efficaces pour prot�ger contre certains types d'attaques (comme l'injection SQL ou les scripts intersites), mais sont moins efficaces contre d'autres types (comme le contr�le d'acc�s ou les probl�mes li�s � la logique m�tier).

Un WAF peut �tre d�ploy� � plusieurs endroits, notamment�:

- Sur le serveur Web lui-m�me.
- Sur une machine virtuelle ou une appliance mat�rielle distincte.
- Dans le cloud devant le serveur back-end.

�tant donn� qu'un WAF bloque les requ�tes malveillantes, il peut �tre d�tect� en ajoutant des cha�nes d'attaque courantes aux param�tres et en observant s'ils sont bloqu�s ou non. Par exemple, essayez d'ajouter un param�tre appel� `foo` avec une valeur telle que `' UNION SELECT 1` ou `><script>alert(1)</script>`. Si ces demandes sont bloqu�es, cela sugg�re qu'il peut y avoir un WAF en place. De plus, le contenu des pages bloqu�es peut fournir des informations sur la technologie sp�cifique utilis�e. Enfin, certains WAF peuvent ajouter des cookies ou des en-t�tes HTTP aux r�ponses qui peuvent r�v�ler leur pr�sence.

Si un WAF bas� sur le cloud est utilis�, il peut �tre possible de le contourner en acc�dant directement au serveur principal, en utilisant les m�mes m�thodes d�crites dans la section [R�seau de diffusion de contenu](#content-delivery-network-cdn).
