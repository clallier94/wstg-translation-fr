# Test du schéma de gestion de session

|ID          |
|------------|
|WSTG-SESS-01|

## Sommaire

L'un des composants essentiels de toute application Web est le mécanisme par lequel elle contrôle et maintient l'état d'un utilisateur qui interagit avec elle. Pour éviter une authentification continue pour chaque page d'un site Web ou d'un service, les applications Web mettent en œuvre divers mécanismes pour stocker et valider les informations d'identification pendant une période de temps prédéterminée. Ces mécanismes sont connus sous le nom de gestion de session.

Dans ce test, le testeur souhaite vérifier que les cookies et autres jetons de session sont créés de manière sécurisée et imprévisible. Un attaquant capable de prédire et de falsifier un cookie faible peut facilement détourner les sessions d'utilisateurs légitimes.

Les cookies sont utilisés pour mettre en œuvre la gestion de session et sont décrits en détail dans la RFC 2965. En un mot, lorsqu'un utilisateur accède à une application qui doit suivre les actions et l'identité de cet utilisateur à travers plusieurs requêtes, un cookie (ou des cookies) est généré par le serveur et envoyé au client. Le client renverra ensuite le cookie au serveur dans toutes les connexions suivantes jusqu'à ce que le cookie expire ou soit détruit. Les données stockées dans le cookie peuvent fournir au serveur un large éventail d'informations sur qui est l'utilisateur, quelles actions il a effectuées jusqu'à présent, quelles sont ses préférences, etc. fournissant ainsi un état à un protocole sans état comme HTTP.

Un exemple typique est fourni par un panier d'achat en ligne. Tout au long de la session d'un utilisateur, l'application doit garder une trace de son identité, de son profil, des produits qu'il a choisi d'acheter, de la quantité, des prix individuels, des remises, etc. Les cookies sont un moyen efficace de stocker et de transmettre cette informations dans les deux sens (les autres méthodes sont les paramètres d'URL et les champs masqués).

Du fait de l'importance des données qu'ils stockent, les cookies sont donc indispensables à la sécurité globale de l'application. La possibilité d'altérer les cookies peut entraîner le détournement des sessions d'utilisateurs légitimes, l'obtention de privilèges plus élevés dans une session active et, en général, une influence non autorisée sur les opérations de l'application.

Dans ce test, le testeur doit vérifier si les cookies émis aux clients peuvent résister à un large éventail d'attaques visant à interférer avec les sessions des utilisateurs légitimes et avec l'application elle-même. L'objectif global est de pouvoir forger un cookie qui sera considéré comme valide par l'application et qui fournira une sorte d'accès non autorisé (détournement de session, élévation de privilèges, ...).

Habituellement, les principales étapes du schéma d'attaque sont les suivantes :

- **collection de cookies** : collecte d'un nombre suffisant d'échantillons de cookies ;
- **reverse engineering des cookies** : analyse de l'algorithme de génération des cookies ;
- **manipulation de cookie** : falsification d'un cookie valide afin d'effectuer l'attaque. Cette dernière étape peut nécessiter un grand nombre de tentatives, selon la façon dont le cookie est créé (attaque par force brute du cookie).

Un autre schéma d'attaque consiste à faire déborder un cookie. Stricto sensu, cette attaque a une autre nature, puisqu'ici les testeurs ne cherchent pas à recréer un cookie parfaitement valide. Au lieu de cela, le but est de déborder une zone mémoire, interférant ainsi avec le bon comportement de l'application et éventuellement injectant (et exécutant à distance) du code malveillant.

## Objectifs des tests

- Rassemblez les jetons de session, pour le même utilisateur et pour différents utilisateurs si possible.
- Analysez et assurez-vous qu'il existe suffisamment d'aléatoire pour arrêter les attaques de forgeage de session.
- Modifier les cookies non signés et contenant des informations manipulables.

## Comment tester

### Tests de boîte noire et exemples

Toute interaction entre le client et l'application doit être testée au moins par rapport aux critères suivants :

- Toutes les directives `Set-Cookie` sont-elles étiquetées comme `Secure` ?
- Des opérations de cookies ont-elles lieu sur un transport non crypté ?
- Le cookie peut-il être forcé sur un transport non crypté ?
- Si oui, comment l'application maintient-elle la sécurité ?
- Certains Cookies sont-ils persistants ?
- Quelles sont les durées d'expiration utilisées pour les cookies persistants, et sont-elles raisonnables ?
- Les cookies censés être transitoires sont-ils configurés en tant que tels ?
- Quels paramètres HTTP/1.1 `Cache-Control` sont utilisés pour protéger les cookies ?
- Quels paramètres HTTP/1.0 `Cache-Control` sont utilisés pour protéger les cookies ?

#### Collection de cookies

La première étape requise pour manipuler le cookie est de comprendre comment l'application crée et gère les cookies. Pour cette tâche, les testeurs doivent essayer de répondre aux questions suivantes :

- Combien de cookies sont utilisés par l'application ?

  Surfez sur l'application. Notez quand les cookies sont créés. Faites une liste des cookies reçus, la page qui les pose (avec la directive set-cookie), le domaine pour lequel ils sont valides, leur valeur et leurs caractéristiques.

- Quelles parties de l'application génèrent ou modifient le cookie ?

  En surfant sur l'application, découvrez quels cookies restent constants et lesquels sont modifiés. Quels événements modifient le cookie ?

- Quelles parties de l'application nécessitent ce cookie pour être accessibles et utilisées ?

  Découvrez quelles parties de l'application ont besoin d'un cookie. Accédez à une page, puis réessayez sans le cookie, ou avec une valeur modifiée de celui-ci. Essayez de cartographier quels cookies sont utilisés et où.

Une feuille de calcul mappant chaque cookie aux composants d'application correspondants et aux informations associées peut être un résultat précieux de cette phase.

#### Analyse de session

Les jetons de session (Cookie, SessionID ou champ caché) eux-mêmes doivent être examinés pour garantir leur qualité du point de vue de la sécurité. Ils doivent être testés par rapport à des critères tels que leur caractère aléatoire, leur unicité, leur résistance à l'analyse statistique et cryptographique et la fuite d'informations.

- Structure des jetons et fuite d'informations

La première étape consiste à examiner la structure et le contenu d'un identifiant de session fourni par l'application. Une erreur courante consiste à inclure des données spécifiques dans le jeton au lieu d'émettre une valeur générique et de référencer des données réelles côté serveur.

Si l'ID de session est en texte clair, la structure et les données pertinentes peuvent être immédiatement évidentes, telles que `192.168.100.1:owaspuser:password:15:58`.

Si une partie ou la totalité du jeton semble être encodée ou hachée, elle doit être comparée à diverses techniques pour vérifier l'obfuscation évidente. Par exemple, la chaîne `192.168.100.1:owaspuser:password:15:58` est représentée en Hex, Base64 et sous forme de hachage MD5 :

- Hex: `3139322E3136382E3130302E313A6F77617370757365723A70617373776F72643A31353A3538`
- Base64: `MTkyLjE2OC4xMDAuMTpvd2FzcHVzZXI6cGFzc3dvcmQ6MTU6NTg=`
- MD5: `01c2fc4f0a817afd8366689bd29dd40a`

Après avoir identifié le type d'obscurcissement, il peut être possible de décoder les données d'origine. Dans la plupart des cas, cependant, cela est peu probable. Même ainsi, il peut être utile d'énumérer l'encodage en place à partir du format du message. De plus, si le format et la technique d'obscurcissement peuvent être déduits, des attaques automatisées par force brute pourraient être conçues.

Les jetons hybrides peuvent inclure des informations telles que l'adresse IP ou l'ID utilisateur avec une partie codée, telle que `owaspuser:192.168.100.1:a7656fafe94dae72b1e1487670148412`.

Après avoir analysé un seul jeton de session, l'échantillon représentatif doit être examiné. Une simple analyse des jetons devrait immédiatement révéler tout schéma évident. Par exemple, un jeton 32 bits peut inclure 16 bits de données statiques et 16 bits de données variables. Cela peut indiquer que les 16 premiers bits représentent un attribut fixe de l'utilisateur - par ex. le nom d'utilisateur ou l'adresse IP. Si le deuxième bloc de 16 bits s'incrémente à un rythme régulier, il peut indiquer un élément séquentiel ou même temporel à la génération du jeton. Voir exemples.

Si des éléments statiques des jetons sont identifiés, d'autres échantillons doivent être collectés, en faisant varier un élément d'entrée potentiel à la fois. Par exemple, les tentatives de connexion via un compte d'utilisateur différent ou à partir d'une adresse IP différente peuvent entraîner une variation dans la partie précédemment statique du jeton de session.

Les domaines suivants doivent être abordés lors des tests de structure d'ID de session unique et multiple :

- Quelles parties de l'identifiant de session sont statiques ?
- Quelles informations confidentielles en clair sont stockées dans l'identifiant de session ? Par exemple. noms d'utilisateur/UID, adresses IP
- Quelles sont les informations confidentielles facilement décodées qui sont stockées ?
- Quelles informations peut-on déduire de la structure du Session ID ?
- Quelles parties de l'ID de session sont statiques pour les mêmes conditions de connexion ?
- Quels modèles évidents sont présents dans l'ID de session dans son ensemble ou dans des parties individuelles ?

#### Prévisibilité et caractère aléatoire de l'ID de session

L'analyse des zones variables (le cas échéant) de l'identifiant de session doit être entreprise pour établir l'existence de modèles reconnaissables ou prévisibles. Ces analyses peuvent être effectuées manuellement et avec des outils statistiques ou cryptanalytiques sur mesure ou OTS pour déduire tout modèle dans le contenu de l'ID de session. Les vérifications manuelles doivent inclure des comparaisons des identifiants de session émis pour les mêmes conditions de connexion - par exemple, le même nom d'utilisateur, mot de passe et adresse IP.

Le temps est un facteur important qui doit également être contrôlé. Un nombre élevé de connexions simultanées doit être effectué afin de collecter des échantillons dans la même fenêtre temporelle et de maintenir cette variable constante. Même une quantification de 50 ms ou moins peut être trop grossière et un échantillon prélevé de cette manière peut révéler des composants basés sur le temps qui seraient autrement manqués.

Les éléments variables doivent être analysés au fil du temps pour déterminer s'ils sont de nature incrémentale. Lorsqu'ils sont incrémentiels, les schémas liés au temps absolu ou écoulé doivent être étudiés. De nombreux systèmes utilisent le temps comme graine pour leurs éléments pseudo-aléatoires. Lorsque les modèles sont apparemment aléatoires, des hachages unidirectionnels du temps ou d'autres variations environnementales doivent être considérés comme une possibilité. En règle générale, le résultat d'un hachage cryptographique est un nombre décimal ou hexadécimal et doit donc être identifiable.

Lors de l'analyse des séquences, modèles ou cycles d'ID de session, les éléments statiques et les dépendances client doivent tous être considérés comme des éléments pouvant contribuer à la structure et à la fonction de l'application.

- Les identifiants de session sont-ils de nature aléatoire ? Les valeurs résultantes peuvent-elles être reproduites ?
- Les mêmes conditions d'entrée produisent-elles le même ID lors d'une exécution ultérieure ?
- Les identifiants de session résistent-ils de manière prouvée à l'analyse statistique ou à la cryptanalyse ?
- Quels éléments des identifiants de session sont liés dans le temps ?
- Quelles parties des identifiants de session sont prévisibles ?
- Le prochain identifiant peut-il être déduit en connaissant parfaitement l'algorithme de génération et les identifiants précédents ?

#### Ingénierie inverse des cookies

Maintenant que le testeur a énuméré les cookies et a une idée générale de leur utilisation, il est temps d'approfondir les cookies qui lui semblent intéressants. Par quels cookies le testeur s'intéresse-t-il ? Un cookie, afin de fournir une méthode sécurisée de gestion de session, doit combiner plusieurs caractéristiques, dont chacune vise à protéger le cookie d'une classe d'attaques différente.

Ces caractéristiques sont résumées ci-dessous :

1. Imprévisibilité : un cookie doit contenir une certaine quantité de données difficiles à deviner. Plus il est difficile de falsifier un cookie valide, plus il est difficile de s'introduire dans la session d'un utilisateur légitime. Si un attaquant peut deviner le cookie utilisé dans une session active d'un utilisateur légitime, il sera en mesure d'usurper complètement l'identité de cet utilisateur (détournement de session). Afin de rendre un cookie imprévisible, des valeurs aléatoires ou une cryptographie peuvent être utilisées.
2. Inviolabilité : un cookie doit résister aux tentatives malveillantes de modification. Si le testeur reçoit un cookie comme `IsAdmin=No`, il est trivial de le modifier pour obtenir des droits d'administrateur, à moins que l'application n'effectue une double vérification (par exemple, en ajoutant au cookie un hachage crypté de sa valeur)
3. Expiration : un cookie critique ne doit être valide que pendant une période de temps appropriée et doit ensuite être supprimé du disque ou de la mémoire pour éviter le risque d'être rejoué. Cela ne s'applique pas aux cookies qui stockent des données non critiques qui doivent être mémorisées au fil des sessions (par exemple, l'apparence du site).
4. Drapeau `Secure` : un cookie dont la valeur est critique pour l'intégrité de la session doit avoir ce drapeau activé afin de permettre sa transmission uniquement dans un canal crypté pour dissuader les écoutes clandestines.

L'approche ici consiste à collecter un nombre suffisant d'instances d'un cookie et à commencer à rechercher des modèles dans leur valeur. La signification exacte de "suffisant" peut varier d'une poignée d'échantillons, si la méthode de génération de cookies est très facile à casser, à plusieurs milliers, si le testeur doit procéder à une analyse mathématique (par exemple, chi-carrés, attracteurs. Voir plus tard pour plus d'informations).

Il est important de porter une attention particulière au workflow de l'application, car l'état d'une session peut avoir un impact important sur les cookies collectés. Un cookie collecté avant d'être authentifié peut être très différent d'un cookie obtenu après l'authentification.

Un autre aspect à prendre en considération est le temps. Enregistrez toujours l'heure exacte à laquelle un cookie a été obtenu, lorsqu'il est possible que le temps joue un rôle dans la valeur du cookie (le serveur pourrait utiliser un horodatage dans le cadre de la valeur du cookie). L'heure enregistrée peut être l'heure locale ou l'horodatage du serveur inclus dans la réponse HTTP (ou les deux).

Lors de l'analyse des valeurs collectées, le testeur doit essayer de comprendre toutes les variables qui pourraient avoir influencé la valeur du cookie et essayer de les faire varier une à la fois. La transmission au serveur de versions modifiées du même cookie peut être très utile pour comprendre comment l'application lit et traite le cookie.

Exemples de vérifications à effectuer à ce stade :

- Quel jeu de caractères est utilisé dans le cookie ? Le cookie a-t-il une valeur numérique ? alphanumérique ? hexadécimal? Que se passe-t-il si le testeur insère dans un cookie des caractères qui n'appartiennent pas au charset attendu ?
- Le cookie est-il composé de différentes sous-parties portant différentes informations ? Comment les différentes parties sont-elles séparées ? Avec quels délimiteurs ? Certaines parties du cookie pourraient avoir une variance plus élevée, d'autres pourraient être constantes, d'autres pourraient ne prendre qu'un ensemble limité de valeurs. Décomposer le cookie en ses composants de base est la première étape fondamentale.

Un exemple de cookie structuré facile à repérer est le suivant :

```text
ID=5a0acfc7ffeb919:CR=1:TM=1120514521:LM=1120514521:S=j3am5KzC4v01ba3q
```

Cet exemple montre 5 champs différents, contenant différents types de données :

- ID - hexadécimal
- CR – petit entier
- TM et LM – grand entier. (Et curieusement, ils ont la même valeur. Cela vaut la peine de voir ce qui se passe en modifiant l'un d'eux)
- S - alphanumérique

Même lorsqu'aucun délimiteur n'est utilisé, le fait d'avoir suffisamment d'échantillons peut aider à comprendre la structure.

#### Attaques par force brute

Les attaques par force brute découlent inévitablement de questions relatives à la prévisibilité et au hasard. L'écart au sein des ID de session doit être pris en compte avec la durée et les délais d'expiration de la session d'application. Si la variation au sein des ID de session est relativement faible et que la validité de l'ID de session est longue, la probabilité d'une attaque par force brute réussie est beaucoup plus élevée.

Un identifiant de session long (ou plutôt avec beaucoup de variance) et une période de validité plus courte rendraient beaucoup plus difficile la réussite d'une attaque par force brute.

- Combien de temps prendrait une attaque par force brute sur tous les identifiants de session possibles ?
- L'espace d'ID de session est-il suffisamment grand pour empêcher le forçage brutal ? Par exemple, la longueur de la clé est-elle suffisante par rapport à la durée de vie valide ?
- Les délais entre les tentatives de connexion avec différents identifiants de session atténuent-ils le risque de cette attaque ?

### Test de boîte grise et exemple

Si le testeur a accès à l'implémentation du schéma de gestion de session, il peut vérifier les éléments suivants :

- Jeton de session aléatoire

  L'identifiant de session ou le cookie envoyé au client ne doit pas être facilement prévisible (n'utilisez pas d'algorithmes linéaires basés sur des variables prévisibles telles que l'adresse IP du client). L'utilisation d'algorithmes cryptographiques avec une longueur de clé de 256 bits est encouragée (comme AES).

- Longueur du jeton

  L'ID de session comportera au moins 50 caractères.

- Expiration de la session

  Le jeton de session doit avoir un délai d'expiration défini (cela dépend de la criticité des données gérées par l'application)

- Paramétrage des cookies :
    - non persistant : uniquement de la mémoire RAM
    - sécurisé (défini uniquement sur le canal HTTPS) : `Set-Cookie : cookie=data ; path=/; domaine=.aaa.it ; secure`
    - [HTTPOnly](https://owasp.org/www-community/HttpOnly) (non lisible par un script) : `Set-Cookie : cookie=data ; chemin=/; domaine=.aaa.it ; HttpOnly`

Plus d'informations ici : [Test des attributs des cookies](02-Testing_for_Cookies_Attributes.md)

## Outils

- [OWASP Zed Attack Proxy Project (ZAP)](https://www.zaproxy.org) - dispose d'un mécanisme d'analyse des jetons de session.
- [Burp Sequencer] (https://portswigger.net/burp/documentation/desktop/tools/sequencer)
- [JHijack de YEHG](https://github.com/yehgdotnet/JHijack)

## Références

### Papiers blanc

- [RFC 2965 "Mécanisme de gestion d'état HTTP"](https://tools.ietf.org/html/rfc2965)
- [RFC 1750 "Recommandations aléatoires pour la sécurité"](https://www.ietf.org/rfc/rfc1750.txt)
- [Michal Zalewski : "Attracteurs étranges et analyse des numéros de séquence TCP/IP" (2001)](http://lcamtuf.coredump.cx/oldtcp/tcpseq.html)
- [Michal Zalewski : "Attracteurs étranges et analyse des numéros de séquence TCP/IP - Un an plus tard" (2002)](http://lcamtuf.coredump.cx/newtcp/)
- [Coefficient de corrélation](http://mathworld.wolfram.com/CorrelationCoefficient.html)
- [ENT](https://fourmilab.ch/random/)
- [DMA[2005-0614a] - 'Global Hauri ViRobot Server cookie overflow'](https://seclists.org/lists/fulldisclosure/2005/Jun/0188.html)
- [Gunter Ollmann : "Gestion de session basée sur le Web"](http://www.technicalinfo.net)
- [Guide de révision du code OWASP](https://wiki.owasp.org/index.php/Category:OWASP_Code_Review_Project)
