# Test pour l'injection HTML

|ID          |
|------------|
|WSTG-CLNT-03|

## Sommaire

L'injection HTML est un type de vulnérabilité d'injection qui se produit lorsqu'un utilisateur est capable de contrôler un point d'entrée et est capable d'injecter du code HTML arbitraire dans une page Web vulnérable. Cette vulnérabilité peut avoir de nombreuses conséquences, comme la divulgation des cookies de session d'un utilisateur qui pourraient être utilisés pour usurper l'identité de la victime, ou, plus généralement, elle peut permettre à l'attaquant de modifier le contenu de la page vue par les victimes.

Cette vulnérabilité se produit lorsque l'entrée utilisateur n'est pas correctement filtrée et que la sortie n'est pas codée. Une injection permet à l'attaquant d'envoyer une page HTML illicite à une victime. Le navigateur ciblé ne sera pas en mesure de distinguer (faire confiance) aux parties légitimes des parties malveillantes de la page, et par conséquent analysera et exécutera la page entière dans le contexte de la victime.

Il existe un large éventail de méthodes et d'attributs pouvant être utilisés pour afficher du contenu HTML. Si ces méthodes sont fournies avec une entrée non fiable, il existe un risque élevé de vulnérabilité d'injection HTML. Par exemple, du code HTML malveillant peut être injecté via la méthode JavaScript `innerHTML`, généralement utilisée pour rendre le code HTML inséré par l'utilisateur. Si les chaînes ne sont pas correctement filtrées, la méthode peut activer l'injection HTML. Une fonction JavaScript qui peut être utilisée à cette fin est `document.write()`.

L'exemple suivant montre un extrait de code vulnérable qui permet d'utiliser une entrée non validée pour créer du HTML dynamique dans le contexte de la page :

```js
var userposition=location.href.indexOf("user=");
var user=location.href.substring(userposition+5);
document.getElementById("Welcome").innerHTML=" Hello, "+user;
```

L'exemple suivant montre du code vulnérable utilisant la fonction `document.write()` :

```js
var userposition=location.href.indexOf("user=");
var user=location.href.substring(userposition+5);
document.write("<h1>Hello, " + user +"</h1>");
```

Dans les deux exemples, cette vulnérabilité peut être exploitée avec une entrée telle que :

```text
http://vulnerable.site/page.html?user=<img%20src='aaa'%20onerror=alert(1)>
```

Cette entrée ajoutera une balise d'image à la page qui exécutera du code JavaScript arbitraire inséré par l'utilisateur malveillant dans le contexte HTML.

## Objectifs des tests

- Identifiez les points d'injection HTML et évaluez la gravité du contenu injecté.

## Comment tester

Considérez l'exercice DOM XSS suivant <http://www.domxss.com/domxss/01_Basics/06_jquery_old_html.html>

Le code HTML contient le script suivant :

```html
<script src="../js/jquery-1.7.1.js"></script>
<script>
function setMessage(){
    var t=location.hash.slice(1);
    $("div[id="+t+"]").text("The DOM is now loaded and can be manipulated.");
}
$(document).ready(setMessage  );
$(window).bind("hashchange",setMessage)
</script>
<body>
    <script src="../js/embed.js"></script>
    <span><a href="#message" > Show Here</a><div id="message">Showing Message1</div></span>
    <span><a href="#message1" > Show Here</a><div id="message1">Showing Message2</div>
    <span><a href="#message2" > Show Here</a><div id="message2">Showing Message3</div>
</body>
```

Il est possible d'injecter du code HTML.
