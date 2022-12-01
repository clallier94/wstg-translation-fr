# Test d'exécution JavaScript

|ID          |
|------------|
|WSTG-CLNT-02|

## Sommaire

Une vulnérabilité d'injection JavaScript est un sous-type de script intersite (XSS) qui implique la capacité d'injecter du code JavaScript arbitraire qui est exécuté par l'application dans le navigateur de la victime. Cette vulnérabilité peut avoir de nombreuses conséquences, comme la divulgation des cookies de session d'un utilisateur qui pourraient être utilisés pour usurper l'identité de la victime, ou, plus généralement, elle peut permettre à l'attaquant de modifier le contenu de la page vue par les victimes ou le comportement de l'application.

Des vulnérabilités d'injection JavaScript peuvent se produire lorsque l'application ne dispose pas d'une validation d'entrée et de sortie appropriée fournie par l'utilisateur. Comme JavaScript est utilisé pour peupler dynamiquement les pages web, cette injection se produit pendant cette phase de traitement du contenu et affecte par conséquent la victime.

Lors du test de cette vulnérabilité, considérez que certains caractères sont traités différemment par différents navigateurs. Pour référence, voir [XSS basé sur DOM](https://owasp.org/www-community/attacks/DOM_Based_XSS).

Voici un exemple de script qui n'effectue aucune validation de la variable `rr`. La variable contient une entrée fournie par l'utilisateur via la chaîne de requête et n'applique en outre aucune forme d'encodage :

```js
var rr = location.search.substring(1);
if(rr) {
    window.location=decodeURIComponent(rr);
}
```

Cela implique qu'un attaquant pourrait injecter du code JavaScript simplement en soumettant la chaîne de requête suivante : `www.victim.com/?javascript:alert(1)`.

## Objectifs des tests

- Identifier les puits et les points d'injection JavaScript possibles.

## Comment tester

Considérez ce qui suit : [Exercice DOM XSS](http://www.domxss.com/domxss/01_Basics/04_eval.html)

La page contient le script suivant :

```html
<script>
function loadObj(){
    var cc=eval('('+aMess+')');
    document.getElementById('mess').textContent=cc.message;
}

if(window.location.hash.indexOf('message')==-1) {
    var aMess='({"message":"Hello User!"})';
} else {
    var aMess=location.hash.substr(window.location.hash.indexOf('message=')+8)
}
</script>
```

Le code ci-dessus contient une source `location.hash` qui est contrôlée par l'attaquant qui peut injecter directement dans la valeur `message` un code JavaScript pour prendre le contrôle du navigateur de l'utilisateur.
