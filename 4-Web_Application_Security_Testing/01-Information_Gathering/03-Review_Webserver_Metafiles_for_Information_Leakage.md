# Examiner les métafichiers du serveur Web pour détecter les fuites d'informations

|ID          |
|------------|
|WSTG-INFO-03|

## Sommaire

Cette section décrit comment tester divers fichiers de métadonnées pour détecter les fuites d'informations sur le(s) chemin(s) ou la fonctionnalité de l'application Web. De plus, la liste des répertoires à éviter par les Spiders, les Robots ou les Crawlers peut également être créée en tant que dépendance pour [Map execution paths through application](07-Map_Execution_Paths_Through_Application.md). D'autres informations peuvent également être collectées pour identifier la surface d'attaque, les détails de la technologie ou pour une utilisation dans le cadre d'un engagement d'ingénierie sociale.

## Objectifs des tests

- Identifiez les chemins et les fonctionnalités cachés ou obscurcis grâce à l'analyse des fichiers de métadonnées.
- Extraire et cartographier d'autres informations qui pourraient conduire à une meilleure compréhension des systèmes à portée de main.

## Comment tester

> Toutes les actions effectuées ci-dessous avec `wget` peuvent également être effectuées avec `curl`. De nombreux outils de test dynamique de la sécurité des applications (DAST) tels que ZAP et Burp Suite incluent des vérifications ou une analyse de ces ressources dans le cadre de leur fonctionnalité spider/crawler. Ils peuvent également être identifiés à l'aide de divers [Google Dorks](https://en.wikipedia.org/wiki/Google_hacking) ou en utilisant des fonctionnalités de recherche avancées telles que `inurl:`.

### Robots

Les Spiders Web, les robots ou les robots d'indexation récupèrent une page Web, puis traversent de manière récursive des hyperliens pour récupérer d'autres contenus Web. Leur comportement accepté est spécifié par le [Robots Exclusion Protocol](https://www.robotstxt.org) du fichier [robots.txt](https://www.robotstxt.org/) dans le répertoire racine Web.

À titre d'exemple, le début du fichier "robots.txt" de [Google](https://www.google.com/robots.txt) échantillonné le 5 mai 2020 est cité ci-dessous :

```text
User-agent: *
Disallow: /search
Allow: /search/about
Allow: /search/static
Allow: /search/howsearchworks
Disallow: /sdch
...
```

La directive [User-Agent](https://developer.mozilla.org/en-US/docs/Web/HTTP/Headers/User-Agent) fait référence au spider/robot/crawler spécifique. Par exemple, `User-Agent : Googlebot` fait référence au spider de Google tandis que `User-Agent : bingbot` fait référence à un crawler de Microsoft. `User-Agent : *` dans l'exemple ci-dessus s'applique à tous les [web spiders/robots/crawlers](https://support.google.com/webmasters/answer/6062608?visit_id=637173940975499736-3548411022&rd=1).

La directive `Disallow` spécifie quelles ressources sont interdites par les spiders/robots/crawlers. Dans l'exemple ci-dessus, les éléments suivants sont interdits :

```text
...
Disallow: /search
...
Disallow: /sdch
...
```

Les spiders/robots/crawlers Web peuvent [intentionnellement ignorer](https://blog.isc2.org/isc2_blog/2008/07/the-attack-of-t.html) les directives `Disallow` spécifiées dans un `robots.txt ` fichier. Par conséquent, `robots.txt` ne doit pas être considéré comme un mécanisme permettant d'appliquer des restrictions sur la manière dont le contenu Web est consulté, stocké ou republié par des tiers.

Le fichier `robots.txt` est récupéré à partir du répertoire racine Web du serveur Web. Par exemple, pour récupérer le `robots.txt` de `www.google.com` en utilisant `wget` ou `curl` :

```bash
$ curl -O -Ss http://www.google.com/robots.txt && head -n5 robots.txt
User-agent: *
Disallow: /search
Allow: /search/about
Allow: /search/static
Allow: /search/howsearchworks
...
```

#### Analysez le fichier robots.txt à l'aide des outils Google pour les webmasters

Les propriétaires de sites Web peuvent utiliser la fonction Google "Analyze robots.txt" pour analyser le site Web dans le cadre de ses [Google Webmaster Tools](https://www.google.com/webmasters/tools). Cet outil peut aider à tester et la procédure est la suivante :

1. Connectez-vous à Google Webmaster Tools avec un compte Google.
2. Sur le tableau de bord, entrez l'URL du site à analyser.
3. Choisissez entre les méthodes disponibles et suivez les instructions à l'écran.

### Balises META

Les balises `<META>` sont situées dans la section `HEAD` de chaque document HTML et doivent être cohérentes sur un site Web dans le cas où le point de départ du robot/spider/crawler ne commence pas à partir d'un lien de document autre que la racine Web, c'est-à-dire un [lien profond](https://en.wikipedia.org/wiki/Deep_linking). La directive Robots peut également être spécifiée via l'utilisation d'une [balise META](https://www.robotstxt.org/meta.html) spécifique.

#### Balise META Robots

S'il n'y a pas d'entrée `<META NAME="ROBOTS" ... >`, alors le "Robots Exclusion Protocol" est par défaut `INDEX,FOLLOW` respectivement. Par conséquent, les deux autres entrées valides définies par le "Robots Exclusion Protocol" sont préfixées par `NO...` c'est-à-dire `NOINDEX` et `NOFOLLOW`.

Sur la base de la ou des directives Disallow répertoriées dans le fichier `robots.txt` dans la racine Web, une recherche d'expression régulière pour `<META NAME="ROBOTS"` dans chaque page Web est entreprise et le résultat est comparé au `robots.txt ` fichier dans la racine Web.

#### Balises d'informations META diverses

Les organisations intègrent souvent des balises META informatives dans le contenu Web pour prendre en charge diverses technologies telles que les lecteurs d'écran, les aperçus des réseaux sociaux, l'indexation des moteurs de recherche, etc. Ces méta-informations peuvent être utiles aux testeurs pour identifier les technologies utilisées et les chemins/fonctionnalités supplémentaires à explorer. et tester. Les méta-informations suivantes ont été extraites de `www.whitehouse.gov` via View Page Source le 05 mai 2020 :

```html
...
<meta property="og:locale" content="en_US" />
<meta property="og:type" content="website" />
<meta property="og:title" content="The White House" />
<meta property="og:description" content="We, the citizens of America, are now joined in a great national effort to rebuild our country and to restore its promise for all. – President Donald Trump." />
<meta property="og:url" content="https://www.whitehouse.gov/" />
<meta property="og:site_name" content="The White House" />
<meta property="fb:app_id" content="1790466490985150" />
<meta property="og:image" content="https://www.whitehouse.gov/wp-content/uploads/2017/12/wh.gov-share-img_03-1024x538.png" />
<meta property="og:image:secure_url" content="https://www.whitehouse.gov/wp-content/uploads/2017/12/wh.gov-share-img_03-1024x538.png" />
<meta name="twitter:card" content="summary_large_image" />
<meta name="twitter:description" content="We, the citizens of America, are now joined in a great national effort to rebuild our country and to restore its promise for all. – President Donald Trump." />
<meta name="twitter:title" content="The White House" />
<meta name="twitter:site" content="@whitehouse" />
<meta name="twitter:image" content="https://www.whitehouse.gov/wp-content/uploads/2017/12/wh.gov-share-img_03-1024x538.png" />
<meta name="twitter:creator" content="@whitehouse" />
...
<meta name="apple-mobile-web-app-title" content="The White House">
<meta name="application-name" content="The White House">
<meta name="msapplication-TileColor" content="#0c2644">
<meta name="theme-color" content="#f5f5f5">
...
```

### Plans du site

Un sitemap est un fichier dans lequel un développeur ou une organisation peut fournir des informations sur les pages, vidéos et autres fichiers proposés par le site ou l'application, ainsi que sur la relation entre eux. Les moteurs de recherche peuvent utiliser ce fichier pour explorer plus intelligemment votre site. Les testeurs peuvent utiliser les fichiers `sitemap.xml` pour en savoir plus sur le site ou l'application afin de l'explorer plus complètement.

L'extrait suivant provient du plan du site principal de Google récupéré le 05 mai 2020.

```bash
$ wget --no-verbose https://www.google.com/sitemap.xml && head -n8 sitemap.xml
2020-05-05 12:23:30 URL:https://www.google.com/sitemap.xml [2049] -> "sitemap.xml" [1]

<?xml version="1.0" encoding="UTF-8"?>
<sitemapindex xmlns="http://www.google.com/schemas/sitemap/0.84">
  <sitemap>
    <loc>https://www.google.com/gmail/sitemap.xml</loc>
  </sitemap>
  <sitemap>
    <loc>https://www.google.com/forms/sitemaps.xml</loc>
  </sitemap>
...
```

En explorant à partir de là, un testeur peut souhaiter récupérer le sitemap gmail `https://www.google.com/gmail/sitemap.xml` :

```xml
<?xml version="1.0" encoding="UTF-8"?>
<urlset xmlns="http://www.sitemaps.org/schemas/sitemap/0.9" xmlns:xhtml="http://www.w3.org/1999/xhtml">
  <url>
    <loc>https://www.google.com/intl/am/gmail/about/</loc>
    <xhtml:link href="https://www.google.com/gmail/about/" hreflang="x-default" rel="alternate"/>
    <xhtml:link href="https://www.google.com/intl/el/gmail/about/" hreflang="el" rel="alternate"/>
    <xhtml:link href="https://www.google.com/intl/it/gmail/about/" hreflang="it" rel="alternate"/>
    <xhtml:link href="https://www.google.com/intl/ar/gmail/about/" hreflang="ar" rel="alternate"/>
...
```

### Sécurité TXT

[security.txt](https://securitytxt.org) a été ratifié par l'IETF en tant que [RFC 9116 - A File Format to Aid in Security Vulnerability Disclosure](https://www.rfc-editor.org/rfc/rfc9116 .html) qui permet aux sites Web de définir des politiques de sécurité et des coordonnées. Il existe plusieurs raisons pour lesquelles cela pourrait être intéressant dans les scénarios de test, y compris, mais sans s'y limiter :

- Identifier d'autres voies ou ressources à inclure dans la découverte/analyse.
- Collecte de renseignements Open Source.
- Trouver des informations sur les Bug Bounties, etc.
- Ingénierie sociale.

Le fichier peut être présent soit à la racine du serveur Web, soit dans le répertoire `.well-known/`. Exemple :

- `https://example.com/security.txt`
- `https://example.com/.well-known/security.txt`

Voici un exemple concret extrait de LinkedIn le 05 mai 2020 :

```bash
$ wget --no-verbose https://www.linkedin.com/.well-known/security.txt && cat security.txt
2020-05-07 12:56:51 URL:https://www.linkedin.com/.well-known/security.txt [333/333] -> "security.txt" [1]
# Conforms to IETF `draft-foudil-securitytxt-07`
Contact: mailto:security@linkedin.com
Contact: https://www.linkedin.com/help/linkedin/answer/62924
Encryption: https://www.linkedin.com/help/linkedin/answer/79676
Canonical: https://www.linkedin.com/.well-known/security.txt
Policy: https://www.linkedin.com/help/linkedin/answer/62924
```

### Humains TXT

`humans.txt` est une initiative pour connaître les personnes derrière un site Web. Il prend la forme d'un fichier texte qui contient des informations sur les différentes personnes qui ont contribué à la construction du site Web. Ce fichier contient souvent (mais pas toujours) des informations sur les carrières ou les sites/chemins d'emploi.

L'exemple suivant a été extrait de Google le 05 mai 2020 :

```bash
$ wget --no-verbose  https://www.google.com/humans.txt && cat humans.txt
2020-05-07 12:57:52 URL:https://www.google.com/humans.txt [286/286] -> "humans.txt" [1]
Google is built by a large team of engineers, designers, researchers, robots, and others in many different sites across the globe. It is updated continuously, and built with more tools and technologies than we can shake a stick at. If you'd like to help us out, see careers.google.com.
```

### Autres sources d'informations bien connues

Il existe d'autres RFC et brouillons Internet qui suggèrent des utilisations standardisées des fichiers dans le répertoire `.well-known/`. Dont les listes peuvent être trouvées [ici](https://en.wikipedia.org/wiki/List_of_/.well-known/_services_offered_by_webservers) ou [ici](https://www.iana.org/assignments/well-uris-connus/uris-connus.xhtml).

Il serait assez simple pour un testeur d'examiner les RFC/drafts et de créer une liste à fournir à un crawler ou à un fuzzer, afin de vérifier l'existence ou le contenu de tels fichiers.

## Outils

- Navigateur (fonctionnalité Afficher la source ou les outils de développement)
- boucle
- wget
- Suite Burp
- ZAP
