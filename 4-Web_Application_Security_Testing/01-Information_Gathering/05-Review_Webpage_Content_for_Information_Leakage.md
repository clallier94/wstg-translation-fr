# Examiner le contenu de la page Web pour détecter les fuites d'informations

|ID          |
|------------|
|WSTG-INFO-05|

## Sommaire

Il est très courant, et même recommandé, que les programmeurs incluent des commentaires détaillés et des métadonnées sur leur code source. Cependant, les commentaires et les métadonnées inclus dans le code HTML peuvent révéler des informations internes qui ne devraient pas être disponibles pour les attaquants potentiels. Les commentaires et l'examen des métadonnées doivent être effectués afin de déterminer si des informations sont divulguées. De plus, certaines applications peuvent divulguer des informations dans le corps des réponses de redirection.

Pour les applications Web modernes, l'utilisation de JavaScript côté client pour le front-end devient de plus en plus populaire. Les technologies de construction front-end populaires utilisent JavaScript côté client comme ReactJS, AngularJS ou Vue. Comme pour les commentaires et les métadonnées dans le code HTML, de nombreux programmeurs codent également en dur des informations sensibles dans des variables JavaScript sur le front-end. Les informations sensibles peuvent inclure (mais sans s'y limiter) : des clés d'API privées (*par exemple* une clé d'API Google Map sans restriction), des adresses IP internes, des routes sensibles (*par exemple* une route vers des pages ou des fonctionnalités d'administration masquées) ou même des informations d'identification. Ces informations sensibles peuvent être divulguées à partir de ce code JavaScript frontal. Un examen doit être effectué afin de déterminer si des informations sensibles ont été divulguées et pourraient être utilisées par des attaquants à des fins abusives.

Pour les grandes applications Web, les problèmes de performances sont une grande préoccupation pour les programmeurs. Les programmeurs ont utilisé différentes méthodes pour optimiser les performances frontales, notamment les feuilles de style syntaxiquement impressionnantes (SASS), Sassy CSS (SCSS), webpack, etc. En utilisant ces technologies, le code frontal deviendra parfois plus difficile à comprendre et difficile à déboguer, et à cause de cela, les programmeurs déploient souvent des fichiers de carte source à des fins de débogage. Une "carte source" est un fichier spécial qui connecte une version minifiée/uglifiée d'un élément (CSS ou JavaScript) à la version originale créée. Les programmeurs se demandent encore s'il faut ou non intégrer les fichiers de carte source dans l'environnement de production. Cependant, il est indéniable que les fichiers de carte source ou les fichiers de débogage, s'ils sont publiés dans l'environnement de production, rendront leur source plus lisible par l'homme. Cela peut permettre aux attaquants de trouver plus facilement des vulnérabilités à partir du front-end ou de collecter des informations sensibles à partir de celui-ci. Une révision du code JavaScript doit être effectuée afin de déterminer si des fichiers de débogage sont exposés à partir du front-end. En fonction du contexte et de la sensibilité du projet, un expert en sécurité doit décider si les fichiers doivent exister ou non dans l'environnement de production.

## Objectifs des tests

- Examinez les commentaires de la page Web, les métadonnées et redirigez les corps pour détecter toute fuite d'informations.
- Rassemblez les fichiers JavaScript et examinez le code JS pour mieux comprendre l'application et détecter toute fuite d'informations.
- Identifiez si des fichiers de carte source ou d'autres fichiers de débogage frontaux existent.

## Comment tester

### Examiner les commentaires et les métadonnées de la page Web

Les commentaires HTML sont souvent utilisés par les développeurs pour inclure des informations de débogage sur l'application. Parfois, ils oublient les commentaires et les laissent dans des environnements de production. Les testeurs doivent rechercher les commentaires HTML qui commencent par `<!--`.

Recherchez dans le code source HTML des commentaires contenant des informations sensibles susceptibles d'aider l'attaquant à mieux comprendre l'application. Il peut s'agir de code SQL, de noms d'utilisateur et de mots de passe, d'adresses IP internes ou d'informations de débogage.

```html
...
<div class="table2">
  <div class="col1">1</div><div class="col2">Mary</div>
  <div class="col1">2</div><div class="col2">Peter</div>
  <div class="col1">3</div><div class="col2">Joe</div>

<!-- Query: SELECT id, name FROM app.users WHERE active='1' -->

</div>
...
```

Le testeur peut même trouver quelque chose comme ceci :

```html
<!-- Use the DB administrator password for testing:  f@keP@a$$w0rD -->
```

Vérifiez les informations de version HTML pour les numéros de version valides et les URL de définition de type de données (DTD)

```html
<!DOCTYPE HTML PUBLIC "-//W3C//DTD HTML 4.01//EN" "http://www.w3.org/TR/html4/strict.dtd">
```

- `strict.dtd` -- DTD stricte par défaut
- `loose.dtd` -- DTD libre
- `frameset.dtd` -- DTD pour les documents de jeu de cadres

Certaines balises `META` ne fournissent pas de vecteurs d'attaque actifs, mais permettent plutôt à un attaquant de profiler une application :

```html
<META name="Author" content="Andrew Muller">
```

Une balise `META` commune (mais non conforme aux [WCAG](https://www.w3.org/WAI/standards-guidelines/wcag/) est [Refresh](https://en.wikipedia.org/wiki/Meta_refresh).

```html
<META http-equiv="Refresh" content="15;URL=https://www.owasp.org/index.html">
```

Une utilisation courante de la balise `META` consiste à spécifier des mots clés qu'un moteur de recherche peut utiliser pour améliorer la qualité des résultats de recherche.

```html
<META name="keywords" lang="en-us" content="OWASP, security, sunshine, lollipops">
```

Bien que la plupart des serveurs Web gèrent l'indexation des moteurs de recherche via le fichier `robots.txt`, elle peut également être gérée par les balises `META`. La balise ci-dessous conseillera aux robots de ne pas indexer et de ne pas suivre les liens sur la page HTML contenant la balise.

```html
<META name="robots" content="none">
```

La [Platform for Internet Content Selection (PICS)](https://www.w3.org/PICS/) et le [Protocol for Web Description Resources (POWDER)](https://www.w3.org/2007/powder/) fournissent une infrastructure pour associer des métadonnées au contenu Internet.

### Identification du code JavaScript et collecte des fichiers JavaScript

Les programmeurs codent souvent en dur des informations sensibles avec des variables JavaScript sur le front-end. Les testeurs doivent vérifier le code source HTML et rechercher le code JavaScript entre les balises `<script>` et `</script>`. Les testeurs doivent également identifier les fichiers JavaScript externes pour examiner le code (les fichiers JavaScript ont l'extension de fichier `.js` et le nom du fichier JavaScript est généralement placé dans l'attribut `src` (source) d'une balise `<script>`).

Vérifiez le code JavaScript pour toute fuite d'informations sensibles qui pourrait être utilisée par des attaquants pour abuser ou manipuler davantage le système. Recherchez des valeurs telles que : clés d'API, adresses IP internes, routes sensibles ou informations d'identification. Par exemple:

```javascript
const myS3Credentials = {
  accessKeyId: config('AWSS3AccessKeyID'),
  secretAcccessKey: config('AWSS3SecretAccessKey'),
};
```

Le testeur peut même trouver quelque chose comme ceci :

```javascript
var conString = "tcp://postgres:1234@localhost/postgres";
```

Lorsqu'une clé API est trouvée, les testeurs peuvent vérifier si les restrictions de clé API sont définies par service ou par IP, référent HTTP, application, SDK, etc.

Par exemple, si les testeurs ont trouvé une clé API Google Map, ils peuvent vérifier si cette clé API est restreinte par adresse IP ou restreinte uniquement par les API Google Map. Si la clé API Google est limitée uniquement par les API Google Map, les attaquants peuvent toujours utiliser cette clé API pour interroger les API Google Map sans restriction et le propriétaire de l'application doit payer pour cela.

```html

<script type="application/json">
...
{"GOOGLE_MAP_API_KEY":"AIzaSyDUEBnKgwiqMNpDplT6ozE4Z0XxuAbqDi4", "RECAPTCHA_KEY":"6LcPscEUiAAAAHOwwM3fGvIx9rsPYUq62uRhGjJ0"}
...
</script>
```

Dans certains cas, les testeurs peuvent trouver des itinéraires sensibles à partir du code JavaScript, tels que des liens vers des pages d'administration internes ou masquées.

```html
<script type="application/json">
...
"runtimeConfig":{"BASE_URL_VOUCHER_API":"https://staging-voucher.victim.net/api", "BASE_BACKOFFICE_API":"https://10.10.10.2/api", "ADMIN_PAGE":"/hidden_administrator"}
...
</script>
```

### Identification des fichiers de carte source

Les fichiers de carte source seront généralement chargés lors de l'ouverture de DevTools. Les testeurs peuvent également trouver des fichiers de carte source en ajoutant l'extension ".map" après l'extension de chaque fichier JavaScript externe. Par exemple, si un testeur voit un fichier `/static/js/main.chunk.js`, il peut alors rechercher son fichier de carte source en visitant `/static/js/main.chunk.js.map`.

Vérifiez les fichiers de carte source pour toute information sensible qui peut aider l'attaquant à mieux comprendre l'application. Par exemple :

```json
{
  "version": 3,
  "file": "static/js/main.chunk.js",
  "sources": [
    "/home/sysadmin/cashsystem/src/actions/index.js",
    "/home/sysadmin/cashsystem/src/actions/reportAction.js",
    "/home/sysadmin/cashsystem/src/actions/cashoutAction.js",
    "/home/sysadmin/cashsystem/src/actions/userAction.js",
    "..."
  ],
  "..."
}
```

Lorsque les sites Web chargent des fichiers de carte source, le code source frontal devient lisible et plus facile à déboguer.

### Identifier les réponses de redirection qui divulguent des informations

Bien que l'on ne s'attende généralement pas à ce que les réponses de redirection contiennent du contenu Web significatif, rien ne garantit qu'elles ne peuvent pas contenir de contenu. Ainsi, bien que les réponses de la série 300 (redirection) contiennent souvent du contenu de type "redirection vers `https://example.com/`", elles peuvent également divulguer du contenu.

Considérez une situation dans laquelle une réponse de redirection est le résultat d'une vérification d'authentification ou d'autorisation, si cette vérification échoue, le serveur peut répondre en redirigeant l'utilisateur vers une page "sûre" ou "par défaut", mais la réponse de redirection elle-même peut toujours contenir du contenu qui ne s'affiche pas dans le navigateur mais est bien transmis au client. Cela peut être vu soit en tirant parti des outils de développement de navigateur, soit via un proxy personnel (tel que ZAP, Burp, Fiddler ou Charles).

## Outils

- [Wget](https://www.gnu.org/software/wget/wget.html)
- Fonction "view source" du navigateur
- Eyeballs
- [Curl](https://curl.haxx.se/)
- [Zaproxy](https://www.zaproxy.org)
- [Burp Suite](https://portswigger.net/burp)
- [Waybackurls](https://github.com/tomnomnom/waybackurls)
- [Analyseur d'API Google Maps](https://github.com/ozguralp/gmapsapiscanner/)

## References

- [KeyHacks](https://github.com/streaak/keyhacks)
- [RingZer0 Online CTF](https://ringzer0ctf.com/challenges/104) - Challenge 104 "Admin Panel".

### Papiers blanc

- [HTML version 4.01](https://www.w3.org/TR/1999/REC-html401-19991224)
- [XHTML](https://www.w3.org/TR/2010/REC-xhtml-basic-20101123/)
- [HTML version 5](https://www.w3.org/TR/html5/)
