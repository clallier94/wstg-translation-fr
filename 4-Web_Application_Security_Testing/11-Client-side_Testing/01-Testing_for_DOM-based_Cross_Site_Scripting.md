# Test pour le Cross Site Scripting basé sur DOM

|ID          |
|------------|
|WSTG-CLNT-01|

## Sommaire

[Script intersite basé sur DOM](https://owasp.org/www-community/attacks/DOM_Based_XSS) est le nom de facto pour [XSS](https://owasp.org/www-community/attacks /xss/) bogues résultant du contenu actif côté navigateur sur une page, généralement JavaScript, obtenant une entrée utilisateur via une [source](https://github.com/wisec/domxsswiki/wiki/sources) et l'utilisant dans un [sink](https://github.com/wisec/domxsswiki/wiki/Sinks), conduisant à l'exécution de code injecté. Ce document ne traite que des bogues JavaScript qui conduisent à XSS.

Le DOM, ou [Document Object Model](https://en.wikipedia.org/wiki/Document_Object_Model), est le format structurel utilisé pour représenter les documents dans un navigateur. Le DOM permet aux scripts dynamiques tels que JavaScript de référencer des composants du document tels qu'un champ de formulaire ou un cookie de session. Le DOM est également utilisé par le navigateur pour la sécurité - par exemple pour empêcher les scripts sur différents domaines d'obtenir des cookies de session pour d'autres domaines. Une vulnérabilité XSS basée sur DOM peut se produire lorsqu'un contenu actif, tel qu'une fonction JavaScript, est modifié par une requête spécialement conçue, de sorte qu'un élément DOM peut être contrôlé par un attaquant.

Tous les bogues XSS n'exigent pas que l'attaquant contrôle le contenu renvoyé par le serveur, mais peuvent plutôt abuser des mauvaises pratiques de codage JavaScript pour obtenir les mêmes résultats. Les conséquences sont les mêmes qu'une faille XSS typique, seul le moyen de livraison est différent.

En comparaison avec d'autres types de vulnérabilités de script intersite ([reflected and storage](https://owasp.org/www-community/attacks/xss/), où un paramètre non nettoyé est transmis par le serveur puis renvoyé au utilisateur et exécutée dans le contexte du navigateur de l'utilisateur, une vulnérabilité XSS basée sur DOM contrôle le flux du code en utilisant des éléments du modèle d'objet de document (DOM) ainsi que du code conçu par l'attaquant pour modifier le flux.

En raison de leur nature, les vulnérabilités XSS basées sur DOM peuvent être exécutées dans de nombreux cas sans que le serveur ne puisse déterminer ce qui est réellement exécuté. Cela peut rendre de nombreuses techniques générales de filtrage et de détection XSS impuissantes à de telles attaques.

Cet exemple hypothétique utilise le code côté client suivant :

```html
<script>
document.write("Site is at : " + document.location.href + ".");
</script>
```

Un attaquant peut ajouter `#<script>alert('xss')</script>` à l'URL de la page affectée qui, une fois exécutée, afficherait la boîte d'alerte. Dans ce cas, le code ajouté ne serait pas envoyé au serveur car tout ce qui suit le caractère "#" n'est pas traité comme faisant partie de la requête par le navigateur, mais comme un fragment. Dans cet exemple, le code est immédiatement exécuté et une alerte "xss" est affichée par la page. Contrairement aux types de scripts intersites les plus courants ([réfléchis et stockés](https://owasp.org/www-community/attacks/xss/) dans lesquels le code est envoyé au serveur puis renvoyé au navigateur, cela est exécuté directement dans le navigateur de l'utilisateur sans contact avec le serveur.

Les [conséquences](https://owasp.org/www-community/attacks/xss/) des failles XSS basées sur DOM sont aussi étendues que celles observées dans des formes XSS plus connues, y compris la récupération de cookies, d'autres scripts malveillants injection, etc., et doivent donc être traités avec la même sévérité.

## Objectifs des tests

- Identifier les puits DOM.
- Créez des charges utiles qui se rapportent à chaque type de puits.

## Comment tester

Les applications JavaScript diffèrent considérablement des autres types d'applications car elles sont souvent générées dynamiquement par le serveur. Pour comprendre quel code est exécuté, le site Web testé doit être exploré pour déterminer toutes les instances de JavaScript en cours d'exécution et où l'entrée de l'utilisateur est acceptée. De nombreux sites Web s'appuient sur de grandes bibliothèques de fonctions, qui s'étendent souvent sur des centaines de milliers de lignes de code et n'ont pas été développées en interne. Dans ces cas, les tests descendants deviennent souvent la seule option viable, car de nombreuses fonctions de niveau inférieur ne sont jamais utilisées, et leur analyse pour déterminer quels sont les puits prendra plus de temps que ce qui est souvent disponible. La même chose peut également être dite pour les tests descendants si les entrées ou leur absence ne sont pas identifiées pour commencer.

L'entrée de l'utilisateur se présente sous deux formes principales :

- Entrée écrite sur la page par le serveur d'une manière qui n'autorise pas le XSS direct, et
- Entrée obtenue à partir d'objets JavaScript côté client.

Voici deux exemples de la manière dont le serveur peut insérer des données dans JavaScript :

```js
var data = "<escaped data from the server>";
var result = someFunction("<escaped data from the server>");
```

Voici deux exemples d'entrées d'objets JavaScript côté client :

```js
var data = window.location;
var result = someFunction(window.referrer);
```

Bien qu'il y ait peu de différence avec le code JavaScript dans la façon dont ils sont récupérés, il est important de noter que lorsque l'entrée est reçue via le serveur, le serveur peut appliquer toutes les permutations aux données qu'il souhaite. En revanche, les permutations effectuées par les objets JavaScript sont assez bien comprises et documentées. Si `someFunction` dans l'exemple ci-dessus était un puits, alors l'exploitabilité dans le premier cas dépendrait du filtrage effectué par le serveur, alors que dans le second cas, cela dépendrait de l'encodage effectué par le navigateur sur le `window.referrer ` objet. Stefano Di Paulo a écrit un excellent article sur ce que les navigateurs renvoient lorsqu'on leur demande les différents éléments d'une [URL utilisant les attributs de document et d'emplacement] (https://github.com/wisec/domxsswiki/wiki/location,-documentURI-and -URL-sources).

De plus, JavaScript est souvent exécuté en dehors des blocs `<script>`, comme en témoignent les nombreux vecteurs qui ont conduit à des contournements de filtres XSS dans le passé. Lors de l'exploration de l'application, il est important de noter l'utilisation de scripts dans des endroits tels que les gestionnaires d'événements et les blocs CSS avec des attributs d'expression. Notez également que tout objet CSS ou script hors site devra être évalué pour déterminer quel code est exécuté.

Les tests automatisés n'ont qu'un succès très limité pour identifier et valider les XSS basés sur DOM, car ils identifient généralement les XSS en envoyant une charge utile spécifique et tentent de l'observer dans la réponse du serveur. Cela peut fonctionner correctement pour l'exemple simple fourni ci-dessous, où le paramètre de message est renvoyé à l'utilisateur :

```html
<script>
var pos=document.URL.indexOf("message=")+5;
document.write(document.URL.substring(pos,document.URL.length));
</script>
```

Cependant, il peut ne pas être détecté dans le cas artificiel suivant :

```html
<script>
var navAgt = navigator.userAgent;

if (navAgt.indexOf("MSIE")!=-1) {
        document.write("You are using IE as a browser and visiting site: " + document.location.href + ".");
}
else
{
    document.write("You are using an unknown browser.");
}
</script>
```

Pour cette raison, les tests automatisés ne détecteront pas les zones susceptibles d'être sensibles au XSS basé sur DOM, à moins que l'outil de test ne puisse effectuer une analyse supplémentaire du code côté client.

Des tests manuels doivent donc être entrepris et peuvent être effectués en examinant les zones du code où des paramètres sont référencés qui peuvent être utiles à un attaquant. des exemples de telles zones incluent des endroits où le code est écrit dynamiquement sur la page et ailleurs où le DOM est modifié ou même où des scripts sont directement exécutés.

## Correction

Pour connaître les mesures visant à empêcher le XSS basé sur DOM, consultez la [Fiche de triche de prévention du XSS basé sur DOM](https://cheatsheetseries.owasp.org/cheatsheets/DOM_based_XSS_Prevention_Cheat_Sheet.html).

## Références

- [DomXSSWiki](https://github.com/wisec/domxsswiki/wiki/)
- [Article DOM XSS par Amit Klein](http://www.webappsec.org/projects/articles/071105.html)
