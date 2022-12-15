# Examiner les anciennes sauvegardes et les fichiers non référencés pour les informations sensibles

|ID          |
|------------|
|WSTG-CONF-04|

## Sommaire

Alors que la plupart des fichiers d'un serveur Web sont directement gérés par le serveur lui-même, il n'est pas rare de trouver des fichiers non référencés ou oubliés qui peuvent être utilisés pour obtenir des informations importantes sur l'infrastructure ou les informations d'identification.

Les scénarios les plus courants incluent la présence d'anciennes versions renommées de fichiers modifiés, de fichiers d'inclusion chargés dans la langue de votre choix et pouvant être téléchargés en tant que source, ou même de sauvegardes automatiques ou manuelles sous forme d'archives compressées. Les fichiers de sauvegarde peuvent également être générés automatiquement par le système de fichiers sous-jacent sur lequel l'application est hébergée, une fonctionnalité généralement appelée "instantanés".

Tous ces fichiers peuvent accorder au testeur l'accès au fonctionnement interne, aux portes dérobées, aux interfaces administratives ou même aux informations d'identification pour se connecter à l'interface administrative ou au serveur de base de données.

Une source importante de vulnérabilité réside dans les fichiers qui n'ont rien à voir avec l'application, mais qui sont créés à la suite de l'édition de fichiers d'application, ou après la création de copies de sauvegarde à la volée, ou en laissant dans l'arborescence des dossiers Web des fichiers anciens ou non référencés. L'exécution d'éditions sur place ou d'autres actions administratives sur des serveurs Web de production peut laisser par inadvertance des copies de sauvegarde, soit générées automatiquement par l'éditeur lors de l'édition de fichiers, soit par l'administrateur qui compresse un ensemble de fichiers pour créer une sauvegarde.

Il est facile d'oublier de tels fichiers et cela peut constituer une menace sérieuse pour la sécurité de l'application. Cela se produit car des copies de sauvegarde peuvent être générées avec des extensions de fichier différentes de celles des fichiers d'origine. Une archive `.tar`, `.zip` ou `.gz` que nous générons (et oublions...) a évidemment une extension différente, et il en va de même avec les copies automatiques créées par de nombreux éditeurs (par exemple, emacs génère une copie de sauvegarde nommée `file~` lors de la modification de `file`). Faire une copie à la main peut produire le même effet (pensez à copier `file` dans `file.old`). Le système de fichiers sous-jacent sur lequel se trouve l'application peut créer des "instantanés" de votre application à différents moments à votre insu, qui peuvent également être accessibles via le Web, ce qui pose une menace de type "fichier de sauvegarde" similaire mais différente pour votre application.

Par conséquent, ces activités génèrent des fichiers qui ne sont pas nécessaires à l'application et peuvent être traités différemment du fichier d'origine par le serveur Web. Par exemple, si nous faisons une copie de `login.asp` nommée `login.asp.old`, nous autorisons les utilisateurs à télécharger le code source de `login.asp`. En effet, `login.asp.old` sera généralement servi sous forme de texte ou brut, plutôt que d'être exécuté en raison de son extension. En d'autres termes, l'accès à `login.asp` entraîne l'exécution du code côté serveur de `login.asp`, tandis que l'accès à `login.asp.old` entraîne le contenu de `login.asp.old` (c'est-à-dire, encore une fois, code côté serveur) pour être clairement renvoyé à l'utilisateur et affiché dans le navigateur. Cela peut poser des risques pour la sécurité, car des informations sensibles peuvent être révélées.

Généralement, exposer le code côté serveur est une mauvaise idée. Non seulement vous exposez inutilement la logique métier, mais vous pouvez révéler sans le savoir des informations liées à l'application qui peuvent aider un attaquant (noms de chemin, structures de données, etc.). Sans parler du fait qu'il y a trop de scripts avec nom d'utilisateur et mot de passe intégrés en texte clair (ce qui est une pratique négligente et très dangereuse).

D'autres causes de fichiers non référencés sont dues à des choix de conception ou de configuration lorsqu'ils permettent à divers types de fichiers liés à l'application, tels que des fichiers de données, des fichiers de configuration, des fichiers journaux, d'être stockés dans des répertoires de système de fichiers accessibles par le serveur Web. Ces fichiers n'ont normalement aucune raison d'être dans un espace de système de fichiers accessible via le Web, car ils ne doivent être accessibles qu'au niveau de l'application, par l'application elle-même (et non par l'utilisateur occasionnel naviguant autour).

### Des menaces

Les fichiers anciens, de sauvegarde et non référencés présentent diverses menaces pour la sécurité d'une application Web :

- Les fichiers non référencés peuvent divulguer des informations sensibles susceptibles de faciliter une attaque ciblée contre l'application ; par exemple, inclure des fichiers contenant des informations d'identification de base de données, des fichiers de configuration contenant des références à d'autres contenus cachés, des chemins de fichiers absolus, etc.
- Les pages non référencées peuvent contenir des fonctionnalités puissantes qui peuvent être utilisées pour attaquer l'application ; par exemple une page d'administration qui n'est pas liée à un contenu publié mais accessible à tout utilisateur qui sait où le trouver.
- Les fichiers anciens et de sauvegarde peuvent contenir des vulnérabilités qui ont été corrigées dans des versions plus récentes ; par exemple, `viewdoc.old.jsp` peut contenir une vulnérabilité de traversée de répertoire qui a été corrigée dans `viewdoc.jsp` mais qui peut toujours être exploitée par quiconque trouve l'ancienne version.
- Les fichiers de sauvegarde peuvent divulguer le code source des pages conçues pour s'exécuter sur le serveur ; par exemple, demander `viewdoc.bak` peut renvoyer le code source de `viewdoc.jsp`, qui peut être examiné pour les vulnérabilités qui peuvent être difficiles à trouver en faisant des requêtes aveugles à la page exécutable. Bien que cette menace s'applique évidemment aux langages de script, tels que Perl, PHP, ASP, les scripts shell, JSP, etc., elle ne se limite pas à eux, comme le montre l'exemple fourni au point suivant.
- Les archives de sauvegarde peuvent contenir des copies de tous les fichiers à l'intérieur (ou même à l'extérieur) de la racine Web. Cela permet à un attaquant d'énumérer rapidement l'intégralité de l'application, y compris les pages non référencées, le code source, les fichiers d'inclusion, etc. Par exemple, si vous oubliez un fichier nommé `myservlets.jar.old` contenant (une copie de sauvegarde de) votre implémentation de servlet classes, vous exposez beaucoup d'informations sensibles susceptibles d'être décompilées et inversées.
- Dans certains cas, la copie ou la modification d'un fichier ne modifie pas l'extension du fichier, mais modifie le nom du fichier. Cela se produit par exemple dans les environnements Windows, où les opérations de copie de fichiers génèrent des noms de fichiers précédés de "Copie de" ou des versions localisées de cette chaîne. Étant donné que l'extension de fichier reste inchangée, il ne s'agit pas d'un cas où un fichier exécutable est renvoyé en texte brut par le serveur Web, et donc pas d'un cas de divulgation du code source. Cependant, ces fichiers sont également dangereux car il est possible qu'ils incluent une logique obsolète et incorrecte qui, lorsqu'elle est invoquée, pourrait déclencher des erreurs d'application, ce qui pourrait fournir des informations précieuses à un attaquant, si l'affichage des messages de diagnostic est activé.
- Les fichiers journaux peuvent contenir des informations sensibles sur les activités des utilisateurs de l'application, par exemple des données sensibles transmises dans les paramètres d'URL, les identifiants de session, les URL visitées (qui peuvent divulguer du contenu supplémentaire non référencé), etc. D'autres fichiers journaux (par exemple, les journaux ftp) peuvent contenir des informations sensibles, des informations sur la maintenance de l'application par les administrateurs système.
- Les instantanés du système de fichiers peuvent contenir des copies du code contenant des vulnérabilités qui ont été corrigées dans des versions plus récentes. Par exemple, `/.snapshot/monthly.1/view.php` peut contenir une vulnérabilité de traversée de répertoire qui a été corrigée dans `/view.php` mais qui peut toujours être exploitée par quiconque trouve l'ancienne version.

## Objectifs des tests

- Recherchez et analysez des fichiers non référencés susceptibles de contenir des informations sensibles.

## Comment tester

### Test de la boîte noire

Le test des fichiers non référencés utilise à la fois des techniques automatisées et manuelles et implique généralement une combinaison des éléments suivants :

#### Inférence à partir du schéma de nommage utilisé pour le contenu publié

Énumérez toutes les pages et fonctionnalités de l'application. Cela peut être fait manuellement à l'aide d'un navigateur ou à l'aide d'un outil de recherche d'applications. La plupart des applications utilisent un schéma de nommage reconnaissable et organisent les ressources en pages et répertoires en utilisant des mots qui décrivent leur fonction. À partir du schéma de nommage utilisé pour le contenu publié, il est souvent possible de déduire le nom et l'emplacement des pages non référencées. Par exemple, si une page `viewuser.asp` est trouvée, recherchez également `edituser.asp`, `adduser.asp` et `deleteuser.asp`. Si un répertoire `/app/user` est trouvé, recherchez également `/app/admin` et `/app/manager`.

#### Autres indices dans le contenu publié

De nombreuses applications Web laissent des indices dans le contenu publié qui peuvent conduire à la découverte de pages et de fonctionnalités cachées. Ces indices apparaissent souvent dans le code source des fichiers HTML et JavaScript. Le code source de tout contenu publié doit être examiné manuellement pour identifier des indices sur d'autres pages et fonctionnalités. Par exemple :

Les commentaires des programmeurs et les sections commentées du code source peuvent faire référence à du contenu caché :

```html
<!-- <A HREF="uploadfile.jsp">Upload a document to the server</A> -->
<!-- Link removed while bugs in uploadfile.jsp are fixed          -->
```

JavaScript peut contenir des liens de page qui ne sont rendus dans l'interface graphique de l'utilisateur que dans certaines circonstances :

```javascript
var adminUser=false;
if (adminUser) menu.add (new menuItem ("Maintain users", "/admin/useradmin.jsp"));
```

Les pages HTML peuvent contenir des FORM qui ont été masqués en désactivant l'élément SUBMIT :

```html
<form action="forgotPassword.jsp" method="post">
    <input type="hidden" name="userID" value="123">
    <!-- <input type="submit" value="Forgot Password"> -->
</form>
```

Une autre source d'indices sur les répertoires non référencés est le fichier `/robots.txt` utilisé pour fournir des instructions aux robots Web :

```html
User-agent: *
Disallow: /Admin
Disallow: /uploads
Disallow: /backup
Disallow: /~jbloggs
Disallow: /include
```

#### Deviner à l'aveugle

Dans sa forme la plus simple, cela implique l'exécution d'une liste de noms de fichiers communs via un moteur de requêtes dans le but de deviner les fichiers et répertoires qui existent sur le serveur. Le script wrapper netcat suivant lira une liste de mots à partir de stdin et effectuera une attaque de devinette de base :

```bash
#!/bin/bash

server=example.org
port=80

while read url
do
echo -ne "$url\t"
echo -e "GET /$url HTTP/1.0\nHost: $server\n" | netcat $server $port | head -1
done | tee outputfile
```

Selon le serveur, GET peut être remplacé par HEAD pour des résultats plus rapides. Le fichier de sortie spécifié peut être greppé pour les codes de réponse "intéressants". Le code de réponse 200 (OK) indique généralement qu'une ressource valide a été trouvée (à condition que le serveur ne délivre pas de page personnalisée "introuvable" à l'aide du code 200). Mais recherchez également 301 (Moved), 302 (Found), 401 (Unauthorized), 403 (Forbidden) et 500 (Internal error), qui peuvent également indiquer des ressources ou des répertoires qui méritent une enquête plus approfondie.

L'attaque par estimation de base doit être exécutée contre la racine Web, ainsi que contre tous les répertoires qui ont été identifiés par d'autres techniques d'énumération. Des attaques de devinettes plus avancées/efficaces peuvent être effectuées comme suit :

- Identifiez les extensions de fichier utilisées dans les zones connues de l'application (par exemple, jsp, aspx, html) et utilisez une liste de mots de base ajoutée à chacune de ces extensions (ou utilisez une liste plus longue d'extensions courantes si les ressources le permettent).
- Pour chaque fichier identifié par d'autres techniques d'énumération, créez une liste de mots personnalisée dérivée de ce nom de fichier. Obtenez une liste des extensions de fichiers courantes (y compris ~, bak, txt, src, dev, old, inc, orig, copy, tmp, swp, etc.) et utilisez chaque extension avant, après et à la place de l'extension du nom de fichier réel.

Remarque : les opérations de copie de fichiers Windows génèrent des noms de fichiers précédés de "Copie de" ou des versions localisées de cette chaîne, elles ne modifient donc pas les extensions de fichier. Bien que les fichiers "Copie de" ne divulguent généralement pas le code source lorsqu'ils sont consultés, ils peuvent fournir des informations précieuses au cas où ils provoqueraient des erreurs lorsqu'ils seraient invoqués.

#### Informations obtenues via les vulnérabilités du serveur et la mauvaise configuration

Le moyen le plus évident par lequel un serveur mal configuré peut divulguer des pages non référencées consiste à lister les répertoires. Demandez à tous les répertoires énumérés d'identifier ceux qui fournissent une liste de répertoires.

De nombreuses vulnérabilités ont été trouvées dans des serveurs Web individuels qui permettent à un attaquant d'énumérer du contenu non référencé, par exemple :

- Vulnérabilité de liste de répertoire Apache ?M=D.
- Diverses vulnérabilités de divulgation de source de script IIS.
- Répertoire IIS WebDAV listant les vulnérabilités.

#### Utilisation des informations accessibles au public

Les pages et les fonctionnalités des applications Web accessibles sur Internet qui ne sont pas référencées à partir de l'application elle-même peuvent être référencées à partir d'autres sources du domaine public. Il existe différentes sources de ces références :

- Des pages autrefois référencées peuvent encore apparaître dans les archives des moteurs de recherche Internet. Par exemple, `1998results.asp` peut ne plus être lié à partir du site Web d'une entreprise, mais peut rester sur le serveur et dans les bases de données des moteurs de recherche. Cet ancien script peut contenir des vulnérabilités qui pourraient être utilisées pour compromettre l'ensemble du site. L'opérateur de recherche Google `site:` peut être utilisé pour exécuter une requête uniquement sur le domaine de votre choix, par exemple dans : `site:www.example.com`. L'utilisation des moteurs de recherche de cette manière a conduit à un large éventail de techniques qui peuvent vous être utiles et qui sont décrites dans la section "Google Hacking" de ce guide. Vérifiez-le pour perfectionner vos compétences de test via Google. Les fichiers de sauvegarde ne sont pas susceptibles d'être référencés par d'autres fichiers et peuvent donc ne pas avoir été indexés par Google, mais s'ils se trouvent dans des répertoires navigables, le moteur de recherche peut les connaître.
- De plus, Google et Yahoo conservent en cache des versions des pages trouvées par leurs robots. Même si `1998results.asp` a été supprimé du serveur cible, une version de sa sortie peut toujours être stockée par ces moteurs de recherche. La version en cache peut contenir des références ou des indices sur du contenu caché supplémentaire qui reste sur le serveur.
- Le contenu qui n'est pas référencé dans une application cible peut être lié à des sites Web tiers. Par exemple, une application qui traite des paiements en ligne pour le compte de commerçants tiers peut contenir une variété de fonctionnalités sur mesure qui ne peuvent (normalement) être trouvées qu'en suivant des liens sur les sites Web de ses clients.

#### Contournement du filtre de nom de fichier

Étant donné que les filtres de liste de refus sont basés sur des expressions régulières, on peut parfois tirer parti des fonctionnalités obscures d'extension des noms de fichiers du système d'exploitation qui fonctionnent d'une manière inattendue par le développeur. Le testeur peut parfois exploiter les différences dans la manière dont les noms de fichiers sont analysés par l'application, le serveur Web et le système d'exploitation sous-jacent et ses conventions de nom de fichier.

Exemple : l'extension de nom de fichier Windows 8.3 `c:\\program files` devient `C:\\PROGRA\~1`

- Supprimer les caractères incompatibles
- Convertir les espaces en traits de soulignement
- Prenez les six premiers caractères du nom de base
- Ajoutez `~<digit>` qui est utilisé pour distinguer les fichiers dont les noms utilisent les mêmes six caractères initiaux
- Cette convention change après les 3 premières collisions de noms
- Tronquer l'extension de fichier à trois caractères
- Mettre tous les caractères en majuscule

### Test de la boîte grise

L'exécution de tests en boîte grise par rapport aux anciens fichiers et aux fichiers de sauvegarde nécessite d'examiner les fichiers contenus dans les répertoires appartenant à l'ensemble des répertoires Web servis par le ou les serveurs Web de l'infrastructure d'application Web. Théoriquement, l'examen doit être effectué à la main pour être approfondi. Cependant, étant donné que dans la plupart des cas, des copies de fichiers ou de fichiers de sauvegarde ont tendance à être créées en utilisant les mêmes conventions de dénomination, la recherche peut être facilement scénarisée. Par exemple, les éditeurs laissent derrière eux des copies de sauvegarde en les nommant avec une extension ou une fin reconnaissable et les humains ont tendance à laisser derrière eux des fichiers avec un ".old" ou des extensions prévisibles similaires. Une bonne stratégie consiste à planifier périodiquement une tâche en arrière-plan pour vérifier les fichiers avec des extensions susceptibles de les identifier comme des fichiers de copie ou de sauvegarde, et à effectuer également des vérifications manuelles sur une plus longue période.

## Correction

Pour garantir une stratégie de protection efficace, les tests doivent être complétés par une politique de sécurité qui interdit clairement les pratiques dangereuses, telles que :

- Modification de fichiers sur place sur les systèmes de fichiers du serveur Web ou du serveur d'applications. C'est une habitude particulièrement mauvaise, car elle est susceptible de générer des sauvegardes ou des fichiers temporaires par les éditeurs. Il est étonnant de voir à quelle fréquence cela se fait, même dans les grandes organisations. Si vous devez absolument modifier des fichiers sur un système de production, assurez-vous de ne rien laisser de côté qui n'est pas explicitement prévu et considérez que vous le faites à vos risques et périls.
- Vérifiez attentivement toute autre activité effectuée sur les systèmes de fichiers exposés par le serveur Web, telles que les activités d'administration ponctuelles. Par exemple, si vous avez occasionnellement besoin de prendre un instantané de quelques répertoires (ce que vous ne devriez pas faire sur un système de production), vous pourriez être tenté de les compresser d'abord. Veillez à ne pas laisser derrière vous ces fichiers d'archive.
- Des politiques de gestion de configuration appropriées devraient aider à prévenir les fichiers obsolètes et non référencés.
- Les applications doivent être conçues pour ne pas créer (ou s'appuyer sur) des fichiers stockés sous les arborescences de répertoires Web servies par le serveur Web. Les fichiers de données, les fichiers journaux, les fichiers de configuration, etc. doivent être stockés dans des répertoires non accessibles par le serveur Web, pour contrer la possibilité de divulgation d'informations (sans parler de la modification des données si les autorisations du répertoire Web permettent l'écriture).
- Les instantanés du système de fichiers ne doivent pas être accessibles via le Web si la racine du document se trouve sur un système de fichiers utilisant cette technologie. Configurez votre serveur Web pour refuser l'accès à ces répertoires, par exemple sous Apache, une directive d'emplacement telle que celle-ci doit être utilisée :

```xml
<Location ~ ".snapshot">
    Order deny,allow
    Deny from all
</Location>
```

## Outils

Les outils d'évaluation des vulnérabilités ont tendance à inclure des vérifications pour repérer les répertoires Web ayant des noms standard (tels que "admin", "test", "backup", etc.) et à signaler tout répertoire Web qui permet l'indexation. Si vous ne pouvez pas obtenir de liste de répertoires, vous devriez essayer de vérifier les extensions de sauvegarde probables. Vérifiez par exemple

- [Nessus](https://www.tenable.com/products/nessus)
- [Nikto2](https://cirt.net/Nikto2)

Outils des Spiders Web

- [wget](https://www.gnu.org/software/wget/)
- [Wget pour Windows](http://www.interlog.com/~tcharron/wgetwin.html)
- [Sam Spade](https://web.archive.org/web/20090926061558/http://preview.samspade.org/ssw/download.html)
- [Le proxy Spike inclut une fonction de robot d'exploration de site Web](https://www.spikeproxy.com/)
- [Xenu](http://home.snafu.de/tilman/xenulink.html)
- [curl](https://curl.haxx.se)

Certains d'entre eux sont également inclus dans les distributions Linux standard. Les outils de développement Web incluent généralement des fonctionnalités permettant d'identifier les liens brisés et les fichiers non référencés.
