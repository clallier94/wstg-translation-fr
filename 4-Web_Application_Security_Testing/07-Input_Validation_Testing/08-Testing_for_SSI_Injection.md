# Test pour l'injection SSI

|ID          |
|------------|
|WSTG-INPV-08|

## Sommaire

Les serveurs Web donnent généralement aux développeurs la possibilité d'ajouter de petits morceaux de code dynamique dans des pages HTML statiques, sans avoir à gérer des langages complets côté serveur ou côté client. Cette fonctionnalité est fournie par [Server-Side Include](https://owasp.org/www-community/attacks/Server-Side_Includes_%28SSI%29_Injection)(SSI).

Les inclusions côté serveur sont des directives que le serveur Web analyse avant de servir la page à l'utilisateur. Ils représentent une alternative à l'écriture de programmes CGI ou à l'intégration de code à l'aide de langages de script côté serveur, lorsqu'il suffit d'effectuer des tâches très simples. Les implémentations SSI courantes fournissent des directives (commandes) pour inclure des fichiers externes, pour définir et imprimer des variables d'environnement CGI de serveur Web ou pour exécuter des scripts CGI externes ou des commandes système.

SSI peut conduire à une exécution de commande à distance (RCE), mais la directive "exec" est désactivée par défaut sur la plupart des serveurs Web.

Il s'agit d'une vulnérabilité très similaire à une vulnérabilité d'injection de langage de script classique. Une atténuation est que le serveur Web doit être configuré pour autoriser SSI. D'autre part, les vulnérabilités d'injection SSI sont souvent plus simples à exploiter, car les directives SSI sont faciles à comprendre et, en même temps, assez puissantes, par exemple, elles peuvent afficher le contenu des fichiers et exécuter des commandes système.

## Objectifs des tests

- Identifier les points d'injection SSI.
- Évaluer la sévérité de l'injection.

## Comment tester

Pour tester les SSI exploitables, injectez des directives SSI en tant qu'entrée utilisateur. Si les SSI sont activés et que la validation des entrées utilisateur n'a pas été correctement implémentée, le serveur exécutera la directive. Ceci est très similaire à une vulnérabilité d'injection de langage de script classique en ce sens qu'elle se produit lorsque l'entrée de l'utilisateur n'est pas correctement validée et filtrée.

Déterminez d'abord si le serveur Web prend en charge les directives SSI. Souvent, la réponse est oui, car le support SSI est assez courant. Pour déterminer si les directives SSI sont prises en charge, découvrez le type de serveur Web que la cible exécute à l'aide de techniques de collecte d'informations (voir [Fingerprint Web Server](../01-Information_Gathering/02-Fingerprint_Web_Server.md)). Si vous avez accès au code, déterminez si les directives SSI sont utilisées en recherchant dans les fichiers de configuration du serveur Web des mots-clés spécifiques.

Une autre façon de vérifier que les directives SSI sont activées consiste à vérifier les pages avec l'extension `.shtml`, qui est associée aux directives SSI. L'utilisation de l'extension `.shtml` n'est pas obligatoire, donc ne pas avoir trouvé de fichiers `.shtml` ne signifie pas nécessairement que la cible n'est pas vulnérable aux attaques par injection SSI.

L'étape suivante consiste à déterminer tous les vecteurs d'entrée utilisateur possibles et à tester pour voir si l'injection SSI est exploitable.

Recherchez d'abord toutes les pages où la saisie de l'utilisateur est autorisée. Les vecteurs d'entrée possibles peuvent également inclure des en-têtes et des cookies. Déterminez comment l'entrée est stockée et utilisée, c'est-à-dire si l'entrée est renvoyée sous forme de message d'erreur ou d'élément de page et si elle a été modifiée d'une manière ou d'une autre. L'accès au code source peut vous aider à déterminer plus facilement où se trouvent les vecteurs d'entrée et comment l'entrée est gérée.

Une fois que vous avez une liste de points d'injection potentiels, vous pouvez déterminer si l'entrée est correctement validée. Assurez-vous qu'il est possible d'injecter des caractères utilisés dans les directives SSI telles que `<!#=/."->` et `[a-zA-Z0-9]`

L'exemple ci-dessous renvoie la valeur de la variable. La section [references](#references) contient des liens utiles avec une documentation spécifique au serveur pour vous aider à mieux évaluer un système particulier.

```html
<!--#echo var="VAR" -->
```

Lors de l'utilisation de la directive `include`, si le fichier fourni est un script CGI, cette directive inclura la sortie du script CGI. Cette directive peut également être utilisée pour inclure le contenu d'un fichier ou lister les fichiers d'un répertoire :

```html
<!--#include virtual="FILENAME" -->
```

Pour renvoyer la sortie d'une commande système :

```html
<!--#exec cmd="OS_COMMAND" -->
```

Si l'application est vulnérable, la directive est injectée et elle sera interprétée par le serveur la prochaine fois que la page sera servie.

Les directives SSI peuvent également être injectées dans les en-têtes HTTP, si l'application Web utilise ces données pour créer une page générée dynamiquement :

```text
GET / HTTP/1.1
Host: www.example.com
Referer: <!--#exec cmd="/bin/ps ax"-->
User-Agent: <!--#include virtual="/proc/version"-->
```

## Outils

- [Suite Web Proxy Burp] (https://portswigger.net/burp/communitydownload)
- [OWASP ZAP](https://www.zaproxy.org/)
- [Chercheur de chaîne : grep](https://www.gnu.org/software/grep)

## Références

- [Module Nginx SSI](http://nginx.org/en/docs/http/ngx_http_ssi_module.html)
- [Apache : Module mod_include](https://httpd.apache.org/docs/current/mod/mod_include.html)
- [IIS : directives côté serveur inclus](https://docs.microsoft.com/en-us/previous-versions/iis/6.0-sdk/ms525185%28v=vs.90%29)
- [Tutoriel Apache : Introduction aux ajouts côté serveur](https://httpd.apache.org/docs/current/howto/ssi.html)
- [Apache : Conseils de sécurité pour la configuration du serveur] (https://httpd.apache.org/docs/current/misc/security_tips.html#ssi)
- [Injection SSI au lieu de JavaScript Malware](https://jeremiahgrossman.blogspot.com/2006/08/ssi-injection-instead-of-javascript.html)
- [IIS : Syntaxe des notes sur le côté serveur (SSI)] (https://blogs.iis.net/robert_mcmurray/archive/2010/12/28/iis-notes-on-server-side-includes-ssi-syntaxe-kb-203064-revisited.aspx)
- [Header Based Exploitation](https://www.cgisecurity.com/papers/header-based-exploitation.txt)
