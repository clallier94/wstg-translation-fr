# Examiner le contenu de la page Web pour d�tecter les fuites d'informations

|ID          |
|------------|
|WSTG-INFO-05|

## Sommaire

Il est tr�s courant, et m�me recommand�, que les programmeurs incluent des commentaires d�taill�s et des m�tadonn�es sur leur code source. Cependant, les commentaires et les m�tadonn�es inclus dans le code HTML peuvent r�v�ler des informations internes qui ne devraient pas �tre disponibles pour les attaquants potentiels. Les commentaires et l'examen des m�tadonn�es doivent �tre effectu�s afin de d�terminer si des informations sont divulgu�es. De plus, certaines applications peuvent divulguer des informations dans le corps des r�ponses de redirection.

Pour les applications Web modernes, l'utilisation de JavaScript c�t� client pour le front-end devient de plus en plus populaire. Les technologies de construction front-end populaires utilisent JavaScript c�t� client comme ReactJS, AngularJS ou Vue. Comme pour les commentaires et les m�tadonn�es dans le code HTML, de nombreux programmeurs codent �galement en dur des informations sensibles dans des variables JavaScript sur le front-end. Les informations sensibles peuvent inclure (mais sans s'y limiter)�: des cl�s d'API priv�es (*par exemple* une cl� d'API Google Map sans restriction), des adresses IP internes, des routes sensibles (*par exemple* une route vers des pages ou des fonctionnalit�s d'administration masqu�es) ou m�me des informations d'identification. Ces informations sensibles peuvent �tre divulgu�es � partir de ce code JavaScript frontal. Un examen doit �tre effectu� afin de d�terminer si des informations sensibles ont �t� divulgu�es et pourraient �tre utilis�es par des attaquants � des fins abusives.

Pour les grandes applications Web, les probl�mes de performances sont une grande pr�occupation pour les programmeurs. Les programmeurs ont utilis� diff�rentes m�thodes pour optimiser les performances frontales, notamment les feuilles de style syntaxiquement impressionnantes (SASS), Sassy CSS (SCSS), webpack, etc. En utilisant ces technologies, le code frontal deviendra parfois plus difficile � comprendre et difficile � d�boguer, et � cause de cela, les programmeurs d�ploient souvent des fichiers de carte source � des fins de d�bogage. Une "carte source" est un fichier sp�cial qui connecte une version minifi�e/uglifi�e d'un �l�ment (CSS ou JavaScript) � la version originale cr��e. Les programmeurs se demandent encore s'il faut ou non int�grer les fichiers de carte source dans l'environnement de production. Cependant, il est ind�niable que les fichiers de carte source ou les fichiers de d�bogage, s'ils sont publi�s dans l'environnement de production, rendront leur source plus lisible par l'homme. Cela peut permettre aux attaquants de trouver plus facilement des vuln�rabilit�s � partir du front-end ou de collecter des informations sensibles � partir de celui-ci. Une r�vision du code JavaScript doit �tre effectu�e afin de d�terminer si des fichiers de d�bogage sont expos�s � partir du front-end. En fonction du contexte et de la sensibilit� du projet, un expert en s�curit� doit d�cider si les fichiers doivent exister ou non dans l'environnement de production.

## Objectifs des tests

- Examinez les commentaires de la page Web, les m�tadonn�es et redirigez les corps pour d�tecter toute fuite d'informations.
- Rassemblez les fichiers JavaScript et examinez le code JS pour mieux comprendre l'application et d�tecter toute fuite d'informations.
- Identifiez si des fichiers de carte source ou d'autres fichiers de d�bogage frontaux existent.

## Comment tester

### Examiner les commentaires et les m�tadonn�es de la page Web

Les commentaires HTML sont souvent utilis�s par les d�veloppeurs pour inclure des informations de d�bogage sur l'application. Parfois, ils oublient les commentaires et les laissent dans des environnements de production. Les testeurs doivent rechercher les commentaires HTML qui commencent par `<!--`.

Recherchez dans le code source HTML des commentaires contenant des informations sensibles susceptibles d'aider l'attaquant � mieux comprendre l'application. Il peut s'agir de code SQL, de noms d'utilisateur et de mots de passe, d'adresses IP internes ou d'informations de d�bogage.

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

Le testeur peut m�me trouver quelque chose comme ceci :

```html
<!-- Use the DB administrator password for testing:  f@keP@a$$w0rD -->
```

V�rifiez les informations de version HTML pour les num�ros de version valides et les URL de d�finition de type de donn�es (DTD)

```html
<!DOCTYPE HTML PUBLIC "-//W3C//DTD HTML 4.01//EN" "http://www.w3.org/TR/html4/strict.dtd">
```

- `strict.dtd` -- DTD stricte par d�faut
- `loose.dtd` -- DTD libre
- `frameset.dtd` -- DTD pour les documents de jeu de cadres

Certaines balises "META" ne fournissent pas de vecteurs d'attaque actifs, mais permettent plut�t � un attaquant de profiler une application�:

```html
<META name="Author" content="Andrew Muller">
```

Une balise `META` commune (mais non conforme aux [WCAG](https://www.w3.org/WAI/standards-guidelines/wcag/) est [Refresh](https://en.wikipedia.org/wiki /Meta_refresh).

```html
<META http-equiv="Refresh" content="15;URL=https://www.owasp.org/index.html">
```

Une utilisation courante de la balise "META" consiste � sp�cifier des mots cl�s qu'un moteur de recherche peut utiliser pour am�liorer la qualit� des r�sultats de recherche.

```html
<META name="keywords" lang="en-us" content="OWASP, security, sunshine, lollipops">
```

Bien que la plupart des serveurs Web g�rent l'indexation des moteurs de recherche via le fichier `robots.txt`, elle peut �galement �tre g�r�e par les balises `META`. La balise ci-dessous conseillera aux robots de ne pas indexer et de ne pas suivre les liens sur la page HTML contenant la balise.

```html
<META name="robots" content="none">
```

La [Platform for Internet Content Selection (PICS)](https://www.w3.org/PICS/) et le [Protocol for Web Description Resources (POWDER)](https://www.w3.org/2007/powder /) fournissent une infrastructure pour associer des m�tadonn�es au contenu Internet.

### Identification du code JavaScript et collecte des fichiers JavaScript

Les programmeurs codent souvent en dur des informations sensibles avec des variables JavaScript sur le front-end. Les testeurs doivent v�rifier le code source HTML et rechercher le code JavaScript entre les balises `<script>` et `</script>`. Les testeurs doivent �galement identifier les fichiers JavaScript externes pour examiner le code (les fichiers JavaScript ont l'extension de fichier `.js` et le nom du fichier JavaScript est g�n�ralement plac� dans l'attribut `src` (source) d'une balise `<script>`).

V�rifiez le code JavaScript pour toute fuite d'informations sensibles qui pourrait �tre utilis�e par des attaquants pour abuser ou manipuler davantage le syst�me. Recherchez des valeurs telles que�: cl�s d'API, adresses�IP internes, routes sensibles ou informations d'identification. Par exemple:

```javascript
const myS3Credentials = {
  accessKeyId: config('AWSS3AccessKeyID'),
  secretAcccessKey: config('AWSS3SecretAccessKey'),
};
```

Le testeur peut m�me trouver quelque chose comme ceci :

```javascript
var conString = "tcp://postgres:1234@localhost/postgres";
```

Lorsqu'une cl� API est trouv�e, les testeurs peuvent v�rifier si les restrictions de cl� API sont d�finies par service ou par IP, r�f�rent HTTP, application, SDK, etc.

Par exemple, si les testeurs ont trouv� une cl� API Google Map, ils peuvent v�rifier si cette cl� API est restreinte par adresse IP ou restreinte uniquement par les API Google Map. Si la cl� API Google est limit�e uniquement par les API Google Map, les attaquants peuvent toujours utiliser cette cl� API pour interroger les API Google Map sans restriction et le propri�taire de l'application doit payer pour cela.

```html

<script type="application/json">
...
{"GOOGLE_MAP_API_KEY":"AIzaSyDUEBnKgwiqMNpDplT6ozE4Z0XxuAbqDi4", "RECAPTCHA_KEY":"6LcPscEUiAAAAHOwwM3fGvIx9rsPYUq62uRhGjJ0"}
...
</script>
```

Dans certains cas, les testeurs peuvent trouver des itin�raires sensibles � partir du code JavaScript, tels que des liens vers des pages d'administration internes ou masqu�es.

```html
<script type="application/json">
...
"runtimeConfig":{"BASE_URL_VOUCHER_API":"https://staging-voucher.victim.net/api", "BASE_BACKOFFICE_API":"https://10.10.10.2/api", "ADMIN_PAGE":"/hidden_administrator"}
...
</script>
```

### Identification des fichiers de carte source

Les fichiers de carte source seront g�n�ralement charg�s lors de l'ouverture de DevTools. Les testeurs peuvent �galement trouver des fichiers de carte source en ajoutant l'extension ".map" apr�s l'extension de chaque fichier JavaScript externe. Par exemple, si un testeur voit un fichier `/static/js/main.chunk.js`, il peut alors rechercher son fichier de carte source en visitant `/static/js/main.chunk.js.map`.

V�rifiez les fichiers de carte source pour toute information sensible qui peut aider l'attaquant � mieux comprendre l'application. Par exemple :

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

Lorsque les sites Web chargent des fichiers de carte source, le code source frontal devient lisible et plus facile � d�boguer.

### Identifier les r�ponses de redirection qui divulguent des informations

Bien que l'on ne s'attende g�n�ralement pas � ce que les r�ponses de redirection contiennent du contenu Web significatif, rien ne garantit qu'elles ne peuvent pas contenir de contenu. Ainsi, bien que les r�ponses de la s�rie 300 (redirection) contiennent souvent du contenu de type "redirection vers `https://example.com/`", elles peuvent �galement divulguer du contenu.

Consid�rez une situation dans laquelle une r�ponse de redirection est le r�sultat d'une v�rification d'authentification ou d'autorisation, si cette v�rification �choue, le serveur peut r�pondre en redirigeant l'utilisateur vers une page "s�re" ou "par d�faut", mais la r�ponse de redirection elle-m�me peut toujours contenir du contenu qui ne s'affiche pas dans le navigateur mais est bien transmis au client. Cela peut �tre vu soit en tirant parti des outils de d�veloppement de navigateur, soit via un proxy personnel (tel que ZAP, Burp, Fiddler ou Charles).

## Outils

- [Wget](https://www.gnu.org/software/wget/wget.html)
- Browser "view source" function
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
