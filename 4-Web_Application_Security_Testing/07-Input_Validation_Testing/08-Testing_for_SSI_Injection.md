# Test pour l'injection SSI

|ID          |
|------------|
|WSTG-INPV-08|

## Sommaire

Les serveurs Web donnent g�n�ralement aux d�veloppeurs la possibilit� d'ajouter de petits morceaux de code dynamique dans des pages HTML statiques, sans avoir � g�rer des langages complets c�t� serveur ou c�t� client. Cette fonctionnalit� est fournie par [Server-Side Include](https://owasp.org/www-community/attacks/Server-Side_Includes_%28SSI%29_Injection)(SSI).

Les inclusions c�t� serveur sont des directives que le serveur Web analyse avant de servir la page � l'utilisateur. Ils repr�sentent une alternative � l'�criture de programmes CGI ou � l'int�gration de code � l'aide de langages de script c�t� serveur, lorsqu'il suffit d'effectuer des t�ches tr�s simples. Les impl�mentations SSI courantes fournissent des directives (commandes) pour inclure des fichiers externes, pour d�finir et imprimer des variables d'environnement CGI de serveur Web ou pour ex�cuter des scripts CGI externes ou des commandes syst�me.

SSI peut conduire � une ex�cution de commande � distance (RCE), mais la directive "exec" est d�sactiv�e par d�faut sur la plupart des serveurs Web.

Il s'agit d'une vuln�rabilit� tr�s similaire � une vuln�rabilit� d'injection de langage de script classique. Une att�nuation est que le serveur Web doit �tre configur� pour autoriser SSI. D'autre part, les vuln�rabilit�s d'injection SSI sont souvent plus simples � exploiter, car les directives SSI sont faciles � comprendre et, en m�me temps, assez puissantes, par exemple, elles peuvent afficher le contenu des fichiers et ex�cuter des commandes syst�me.

## Objectifs des tests

- Identifier les points d'injection SSI.
- �valuer la s�v�rit� de l'injection.

## Comment tester

Pour tester les SSI exploitables, injectez des directives SSI en tant qu'entr�e utilisateur. Si les SSI sont activ�s et que la validation des entr�es utilisateur n'a pas �t� correctement impl�ment�e, le serveur ex�cutera la directive. Ceci est tr�s similaire � une vuln�rabilit� d'injection de langage de script classique en ce sens qu'elle se produit lorsque l'entr�e de l'utilisateur n'est pas correctement valid�e et filtr�e.

D�terminez d'abord si le serveur Web prend en charge les directives SSI. Souvent, la r�ponse est oui, car le support SSI est assez courant. Pour d�terminer si les directives SSI sont prises en charge, d�couvrez le type de serveur Web que la cible ex�cute � l'aide de techniques de collecte d'informations (voir [Fingerprint Web Server](../01-Information_Gathering/02-Fingerprint_Web_Server.md)). Si vous avez acc�s au code, d�terminez si les directives SSI sont utilis�es en recherchant dans les fichiers de configuration du serveur Web des mots-cl�s sp�cifiques.

Une autre fa�on de v�rifier que les directives SSI sont activ�es consiste � v�rifier les pages avec l'extension `.shtml`, qui est associ�e aux directives SSI. L'utilisation de l'extension `.shtml` n'est pas obligatoire, donc ne pas avoir trouv� de fichiers `.shtml` ne signifie pas n�cessairement que la cible n'est pas vuln�rable aux attaques par injection SSI.

L'�tape suivante consiste � d�terminer tous les vecteurs d'entr�e utilisateur possibles et � tester pour voir si l'injection SSI est exploitable.

Recherchez d'abord toutes les pages o� la saisie de l'utilisateur est autoris�e. Les vecteurs d'entr�e possibles peuvent �galement inclure des en-t�tes et des cookies. D�terminez comment l'entr�e est stock�e et utilis�e, c'est-�-dire si l'entr�e est renvoy�e sous forme de message d'erreur ou d'�l�ment de page et si elle a �t� modifi�e d'une mani�re ou d'une autre. L'acc�s au code source peut vous aider � d�terminer plus facilement o� se trouvent les vecteurs d'entr�e et comment l'entr�e est g�r�e.

Une fois que vous avez une liste de points d'injection potentiels, vous pouvez d�terminer si l'entr�e est correctement valid�e. Assurez-vous qu'il est possible d'injecter des caract�res utilis�s dans les directives SSI telles que `<!#=/."->` et `[a-zA-Z0-9]`

L'exemple ci-dessous renvoie la valeur de la variable. La section [references](#references) contient des liens utiles avec une documentation sp�cifique au serveur pour vous aider � mieux �valuer un syst�me particulier.

```html
<!--#echo var="VAR" -->
```

Lors de l'utilisation de la directive `include`, si le fichier fourni est un script CGI, cette directive inclura la sortie du script CGI. Cette directive peut �galement �tre utilis�e pour inclure le contenu d'un fichier ou lister les fichiers d'un r�pertoire�:

```html
<!--#include virtual="FILENAME" -->
```

Pour renvoyer la sortie d'une commande syst�me�:

```html
<!--#exec cmd="OS_COMMAND" -->
```

Si l'application est vuln�rable, la directive est inject�e et elle sera interpr�t�e par le serveur la prochaine fois que la page sera servie.

Les directives SSI peuvent �galement �tre inject�es dans les en-t�tes HTTP, si l'application Web utilise ces donn�es pour cr�er une page g�n�r�e dynamiquement�:

```text
GET / HTTP/1.1
Host: www.example.com
Referer: <!--#exec cmd="/bin/ps ax"-->
User-Agent: <!--#include virtual="/proc/version"-->
```

## Outils

- [Suite Web Proxy Burp] (https://portswigger.net/burp/communitydownload)
- [OWASP ZAP](https://www.zaproxy.org/)
- [Chercheur de cha�ne : grep](https://www.gnu.org/software/grep)

## R�f�rences

- [Module Nginx SSI](http://nginx.org/en/docs/http/ngx_http_ssi_module.html)
- [Apache�: Module mod_include](https://httpd.apache.org/docs/current/mod/mod_include.html)
- [IIS�: directives c�t� serveur inclus](https://docs.microsoft.com/en-us/previous-versions/iis/6.0-sdk/ms525185%28v=vs.90%29)
- [Tutoriel Apache�: Introduction aux ajouts c�t� serveur](https://httpd.apache.org/docs/current/howto/ssi.html)
- [Apache�: Conseils de s�curit� pour la configuration du serveur] (https://httpd.apache.org/docs/current/misc/security_tips.html#ssi)
- [Injection SSI au lieu de JavaScript Malware](https://jeremiahgrossman.blogspot.com/2006/08/ssi-injection-instead-of-javascript.html)
- [IIS�: Syntaxe des notes sur le c�t� serveur (SSI)] (https://blogs.iis.net/robert_mcmurray/archive/2010/12/28/iis-notes-on-server-side-includes-ssi-syntaxe-kb-203064-revisited.aspx)
- [Header Based Exploitation](https://www.cgisecurity.com/papers/header-based-exploitation.txt)
