# Architecture des applications cartographiques

|ID          |
|------------|
|WSTG-INFO-10|

## Sommaire

Afin de tester efficacement une application et d'être en mesure de fournir des recommandations significatives sur la manière de résoudre les problèmes identifiés, il est important de comprendre ce que vous testez réellement. De plus, cela peut aider à déterminer si des composants spécifiques doivent être considérés comme hors de portée des tests.

Les applications Web modernes peuvent varier considérablement en complexité, allant d'un simple script exécuté sur un seul serveur à une application très complexe répartie sur des dizaines de systèmes, langages et composants différents. Il peut également y avoir des composants supplémentaires au niveau du réseau tels que des pare-feu ou des systèmes de protection contre les intrusions qui peuvent avoir un impact significatif sur les tests.

## Objectifs des tests

- Comprendre l'architecture de l'application et les technologies utilisées.

## Comment tester

Lorsque vous testez du point de vue de la boîte noire, il est important d'essayer de créer une image claire du fonctionnement de l'application et des technologies et composants en place. Dans certains cas, il est possible de tester des composants spécifiques (comme un pare-feu d'application Web), tandis que d'autres peuvent être identifiés en inspectant le comportement de l'application.

Les sections ci-dessous fournissent une vue d'ensemble de haut niveau des composants architecturaux courants, ainsi que des détails sur la façon dont ils peuvent être identifiés.

### Application Components

#### Web Server

Des applications simples peuvent s'exécuter sur un seul serveur, qui peut être identifié à l'aide des étapes décrites dans la section [Empreintes sur les serveurs Web](02-Fingerprint_Web_Server.md) du guide.

#### Plate-forme en tant que service (PaaS)

Dans un modèle de plate-forme en tant que service (PaaS), le serveur Web et l'infrastructure sous-jacente sont gérés par le fournisseur de services, et le client n'est responsable que de l'application déployée sur lui. Du point de vue des tests, il existe deux différences principales :

- Le propriétaire de l'application n'a pas accès à l'infrastructure sous-jacente, il ne sera donc pas en mesure de résoudre directement les problèmes.
- Les tests d'infrastructure sont susceptibles d'être hors de portée de toute mission.

Dans certains cas, il est possible d'identifier l'utilisation de PaaS, car l'application peut utiliser un nom de domaine spécifique (par exemple, les applications déployées sur Azure App Services auront un domaine `*.azurewebsites.net` - bien qu'elles puissent également utiliser des domaines). Cependant, dans d'autres cas, il est difficile de déterminer si le PaaS est utilisé.

#### Serverless

Dans un modèle sans serveur (Serverless), les développeurs fournissent du code qui est directement exécuté sur une plate-forme d'hébergement en tant que fonctions individuelles, plutôt qu'en tant qu'application Web traditionnelle plus grande déployée dans une racine Web. Cela le rend bien adapté aux architectures basées sur des microservices. Comme avec un environnement PaaS, les tests d'infrastructure sont susceptibles d'être hors de portée.

Dans certains cas, l'utilisation de code Serverless peut être indiquée par la présence d'en-têtes HTTP spécifiques. Par exemple, les fonctions AWS Lambda renvoient généralement les en-têtes suivants :

```http
X-Amz-Invocation-Type
X-Amz-Log-Type
X-Amz-Client-Context
```

Les fonctions Azure sont moins évidentes. Ils renvoient généralement l'en-tête `Server: Kestrel` - mais cela ne suffit pas en soi pour être sûr qu'il s'agit d'une fonction Azure App, plutôt que d'un autre code exécuté sur Kestrel.

#### Microservices

Dans une architecture basée sur des microservices, l'API de l'application est composée de plusieurs services discrets, plutôt que de s'exécuter comme une application monolithique. Les services eux-mêmes s'exécutent souvent dans des conteneurs (généralement avec Kubernetes) et peuvent utiliser une variété de systèmes d'exploitation et de langages différents. Bien qu'ils se trouvent généralement derrière une passerelle API et un domaine uniques, l'utilisation de plusieurs langues (souvent indiquée dans des messages d'erreur détaillés) peut suggérer que des microservices sont utilisés.

#### Stockage statique

De nombreuses applications stockent le contenu statique sur des plates-formes de stockage dédiées, plutôt que de l'héberger directement sur le serveur Web principal. Les deux plates-formes les plus courantes sont les compartiments S3 d'Amazon et les comptes de stockage d'Azure, et peuvent être facilement identifiées par les noms de domaine :

- Les compartiments Amazon S3 sont soit `BUCKET.s3.amazonaws.com` ou `s3.REGION.amazonaws.com/BUCKET`
- Les comptes de stockage Azure sont `ACCOUNT.blob.core.windows.net`

Ces comptes de stockage peuvent souvent exposer des fichiers sensibles, comme indiqué dans la section [Guide de test du stockage dans le cloud](../02-Configuration_and_Deployment_Management_Testing/11-Test_Cloud_Storage.md).

#### Base de données

La plupart des applications Web non triviales utilisent une sorte de base de données pour stocker du contenu dynamique. Dans certains cas, il est possible de déterminer la base de données, bien qu'elle repose généralement sur d'autres problèmes de l'application. Cela peut souvent être fait par :

- Port scannant le serveur et recherchant tous les ports ouverts associés à des bases de données spécifiques.
- Déclenchement de messages d'erreur liés à SQL (ou NoSQL) (ou recherche d'erreurs existantes à partir d'un [moteur de recherche](../01-Information_Gathering/01-Conduct_Search_Engine_Discovery_Reconnaissance_for_Information_Leakage.md).

Lorsqu'il n'est pas possible de déterminer de manière concluante la base de données, vous pouvez souvent faire une supposition éclairée basée sur d'autres aspects de l'application :

- Windows, IIS et ASP.NET utilisent souvent le serveur Microsoft SQL.
- Les systèmes embarqués utilisent souvent SQLite.
- PHP utilise souvent MySQL ou PostgreSQL.
- APEX utilise souvent Oracle.

Ce ne sont pas des règles strictes, mais elles peuvent certainement vous donner un point de départ raisonnable si aucune meilleure information n'est disponible.

#### Authentification

La plupart des applications ont une certaine forme d'authentification pour les utilisateurs. Plusieurs back-ends d'authentification peuvent être utilisés, tels que :

- Configuration du serveur Web (y compris les fichiers `.htaccess`) ou codage en dur des mots de passe dans les scripts.
    - S'affiche généralement en tant qu'authentification HTTP Basic, indiquée par une fenêtre contextuelle dans le navigateur et un en-tête HTTP `WWW-Authenticate : Basic`.
- Comptes d'utilisateurs locaux dans une base de données.
    - Généralement intégré dans un formulaire ou un point de terminaison API sur l'application.
- Une source d'authentification centrale existante comme Active Directory ou un serveur LDAP.
    - Peut utiliser l'authentification NTLM, indiquée par un en-tête HTTP `WWW-Authenticate : NTLM`.
    - Peut être intégré à l'application web dans un formulaire.
    - Peut exiger que le nom d'utilisateur soit saisi au format « DOMAINE\nom d'utilisateur », ou peut donner une liste déroulante des domaines disponibles.
- Single Sign-On (SSO) avec un fournisseur interne ou externe.
    - Utilise généralement OAuth, OpenID Connect ou SAML.

Les applications peuvent fournir plusieurs options à l'utilisateur pour s'authentifier (telles que l'enregistrement d'un compte local ou l'utilisation de son compte Facebook existant) et peuvent utiliser différents mécanismes pour les utilisateurs normaux et les administrateurs.

#### Services et API tiers

Presque toutes les applications Web incluent des ressources tierces qui sont chargées ou avec lesquelles le client interagit. Ceux-ci peuvent inclure :

- [Contenu actif](https://developer.mozilla.org/en-US/docs/Web/Security/Mixed_content#mixed_active_content) (comme les scripts, les feuilles de style, les polices et les iframes).
- [Contenu passif](https://developer.mozilla.org/en-US/docs/Web/Security/Mixed_content#mixed_passivedisplay_content) (comme des images et des vidéos).
- API externes.
- Boutons de médias sociaux.
- Réseaux publicitaires.
- Passerelles de paiement.

Ces ressources sont demandées directement par le navigateur de l'utilisateur, elles peuvent donc être facilement identifiées à l'aide des outils de développement ou d'un proxy d'interception. Bien qu'il soit important de les identifier (car ils peuvent avoir un impact sur la sécurité de l'application), n'oubliez pas qu'ils sont généralement hors de portée des tests, car ils appartiennent à des tiers.

### Composants réseau

#### Proxy inversé

Un proxy inversé se trouve devant un ou plusieurs serveurs principaux et redirige les requêtes vers la destination appropriée. Ils peuvent implémenter diverses fonctionnalités, telles que :

- Agir en tant qu'[équilibreur de charge](#Équilibreur-de-charge) ou [web application firewall](#web-application-firewall-waf).
- Permettre à plusieurs applications d'être hébergées sur une seule adresse IP ou un seul domaine (dans des sous-dossiers).
- Mettre en place un filtrage IP ou d'autres restrictions.
- Mise en cache du contenu du back-end pour améliorer les performances.

Il n'est pas toujours possible de détecter un reverse proxy (surtout s'il n'y a qu'une application derrière), mais on peut souvent parfois l'identifier par :

- Une incompatibilité entre le serveur frontal et l'application principale (comme un en-tête `Server: nginx` avec une application ASP.NET).
    - Cela peut parfois entraîner des [vulnérabilités de contrebande de requêtes](https://portswigger.net/web-security/request-smuggling).
- En-têtes en double (surtout l'en-tête `Server`).
- Plusieurs applications hébergées sur la même adresse IP ou le même domaine (surtout si elles utilisent des langues différentes).

#### Équilibreur de charge

Un équilibreur de charge se trouve devant plusieurs serveurs principaux et alloue les requêtes entre eux afin de fournir une plus grande redondance et une plus grande capacité de traitement pour l'application.

Les équilibreurs de charge peuvent être difficiles à détecter, mais peuvent parfois être identifiés en faisant plusieurs requêtes et en examinant les réponses pour les différences, telles que :

- Heures système incohérentes.
- Différentes adresses IP internes ou noms d'hôte dans les messages d'erreur détaillés.
- Différentes adresses renvoyées par [test de contrefaçon de requête côté serveur (SSRF)](../07-Input_Validation_Testing/19-Testing_for_Server-Side_Request_Forgery.md).

Ils peuvent également être signalés par la présence de cookies spécifiques (par exemple, les équilibreurs de charge F5 BIG-IP créeront un cookie appelé `BIGipServer`.

#### Réseau de diffusion de contenu (CDN)

Un réseau de diffusion de contenu (CDN) est un ensemble géographiquement distribué de serveurs proxy de mise en cache, conçu pour améliorer les performances du site Web afin de fournir une résilience supplémentaire à un site Web.

Il est généralement configuré en pointant le domaine public vers les serveurs du CDN, puis en configurant le CDN pour qu'il se connecte aux serveurs principaux corrects (parfois appelés "l'origine").

Le moyen le plus simple de détecter un CDN consiste à effectuer une recherche WHOIS pour les adresses IP auxquelles le domaine se résout. S'ils appartiennent à une société CDN (comme Akamai, Cloudflare ou Fastly - voir [Wikipedia](https://en.wikipedia.org/wiki/Content_delivery_network#Notable_content_delivery_service_providers) pour une liste plus complète), alors c'est comme si un CDN était utilisé.

Lorsque vous testez un site derrière un CDN, vous devez garder à l'esprit les points suivants :

- Les adresses IP et les serveurs appartiennent au fournisseur de CDN et sont susceptibles d'être hors de portée des tests d'infrastructure.
- De nombreux CDN incluent également des fonctionnalités telles que la détection de robots, la limitation de débit et les pare-feu d'applications Web.
- Les CDN mettent généralement en cache le contenu, de sorte que toute modification apportée au site Web principal peut ne pas apparaître immédiatement.

Si le site se trouve derrière un CDN, il peut être utile d'identifier les serveurs principaux. S'ils ne disposent pas d'un contrôle d'accès approprié, vous pourrez peut-être contourner le CDN (et toutes les protections qu'il offre) en accédant directement aux serveurs principaux. Il existe une variété de méthodes différentes qui peuvent vous permettre d'identifier le système back-end :

- Les e-mails envoyés par l'application peuvent provenir directement du serveur principal, ce qui pourrait révéler son adresse IP.
- Le broyage DNS, les transferts de zone ou les listes de transparence des certificats pour le domaine peuvent le révéler sur un sous-domaine.
- L'analyse des plages d'adresses IP connues pour être utilisées par l'entreprise peut trouver le serveur principal.
- L'exploitation de [test de contrefaçon de requête côté serveur (SSRF)](../07-Input_Validation_Testing/19-Testing_for_Server-Side_Request_Forgery.md) peut révéler l'adresse IP.
- Des messages d'erreur détaillés de l'application peuvent exposer des adresses IP ou des noms d'hôte.

### Composants de sécurité

#### Pare-feu réseau

La plupart des serveurs Web seront protégés par un pare-feu de filtrage de paquets ou d'inspection dynamique, qui bloque tout trafic réseau non requis. Pour le détecter, effectuez une analyse des ports du serveur et examinez les résultats.

Si la majorité des ports sont affichés comme "fermés" (c'est-à-dire qu'ils renvoient un paquet `RST` en réponse au paquet `SYN` initial), cela suggère que le serveur n'est peut-être pas protégé par un pare-feu. Si les ports sont affichés comme "filtrés" (c'est-à-dire qu'aucune réponse n'est reçue lors de l'envoi d'un paquet `SYN` vers un port inutilisé), un pare-feu est très probablement en place.

De plus, si des services inappropriés sont exposés au monde (tels que SMTP, IMAP, MySQL, etc.), cela suggère soit qu'il n'y a pas de pare-feu en place, soit que le pare-feu est mal configuré.

#### Système de détection et de prévention des intrusions sur le réseau

Un système de détection d'intrusion (IDS) réseau détecte les activités suspectes ou malveillantes au niveau du réseau (telles que l'analyse des ports ou des vulnérabilités) et déclenche des alertes. Un système de prévention des intrusions (IPS) est similaire, mais prend également des mesures pour empêcher l'activité - généralement en bloquant l'adresse IP source.

Un IPS peut généralement être détecté en exécutant des outils d'analyse automatisés (tels qu'un analyseur de ports) sur la cible et en vérifiant si l'adresse IP source est bloquée. Cependant, de nombreux outils au niveau de l'application peuvent ne pas être détectés par un IPS (surtout s'il ne déchiffre pas TLS).

#### Pare-feu d'application Web (WAF)

Un pare-feu d'application Web (WAF) inspecte le contenu des requêtes HTTP et bloque celles qui semblent suspectes ou malveillantes, ou applique dynamiquement d'autres contrôles tels que CAPTCHA ou la limitation du débit. Ils sont généralement basés sur un ensemble de mauvaises signatures et d'expressions régulières connues, telles que [OWASP Core Rule Set](https://owasp.org/www-project-modsecurity-core-rule-set/). Les WAF peuvent être efficaces pour protéger contre certains types d'attaques (comme l'injection SQL ou les scripts intersites), mais sont moins efficaces contre d'autres types (comme le contrôle d'accès ou les problèmes liés à la logique métier).

Un WAF peut être déployé à plusieurs endroits, notamment :

- Sur le serveur Web lui-même.
- Sur une machine virtuelle ou une appliance matérielle distincte.
- Dans le cloud devant le serveur back-end.

Étant donné qu'un WAF bloque les requêtes malveillantes, il peut être détecté en ajoutant des chaînes d'attaque courantes aux paramètres et en observant s'ils sont bloqués ou non. Par exemple, essayez d'ajouter un paramètre appelé `foo` avec une valeur telle que `' UNION SELECT 1` ou `><script>alert(1)</script>`. Si ces demandes sont bloquées, cela suggère qu'il peut y avoir un WAF en place. De plus, le contenu des pages bloquées peut fournir des informations sur la technologie spécifique utilisée. Enfin, certains WAF peuvent ajouter des cookies ou des en-têtes HTTP aux réponses qui peuvent révéler leur présence.

Si un WAF basé sur le cloud est utilisé, il peut être possible de le contourner en accédant directement au serveur principal, en utilisant les mêmes méthodes décrites dans la section [Réseau de diffusion de contenu (CDN)](#Réseau-de-diffusion-de-contenu-(CDN)).
