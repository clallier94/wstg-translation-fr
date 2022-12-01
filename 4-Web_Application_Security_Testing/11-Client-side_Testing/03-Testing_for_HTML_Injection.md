# Test pour l'injection HTML

|ID          |
|------------|
|WSTG-CLNT-03|

## Sommaire

L'injection HTML est un type de vuln�rabilit� d'injection qui se produit lorsqu'un utilisateur est capable de contr�ler un point d'entr�e et est capable d'injecter du code HTML arbitraire dans une page Web vuln�rable. Cette vuln�rabilit� peut avoir de nombreuses cons�quences, comme la divulgation des cookies de session d'un utilisateur qui pourraient �tre utilis�s pour usurper l'identit� de la victime, ou, plus g�n�ralement, elle peut permettre � l'attaquant de modifier le contenu de la page vue par les victimes.

Cette vuln�rabilit� se produit lorsque l'entr�e utilisateur n'est pas correctement filtr�e et que la sortie n'est pas cod�e. Une injection permet � l'attaquant d'envoyer une page HTML illicite � une victime. Le navigateur cibl� ne sera pas en mesure de distinguer (faire confiance) aux parties l�gitimes des parties malveillantes de la page, et par cons�quent analysera et ex�cutera la page enti�re dans le contexte de la victime.

Il existe un large �ventail de m�thodes et d'attributs pouvant �tre utilis�s pour afficher du contenu HTML. Si ces m�thodes sont fournies avec une entr�e non fiable, il existe un risque �lev� de vuln�rabilit� d'injection HTML. Par exemple, du code HTML malveillant peut �tre inject� via la m�thode JavaScript `innerHTML`, g�n�ralement utilis�e pour rendre le code HTML ins�r� par l'utilisateur. Si les cha�nes ne sont pas correctement filtr�es, la m�thode peut activer l'injection HTML. Une fonction JavaScript qui peut �tre utilis�e � cette fin est `document.write()`.

L'exemple suivant montre un extrait de code vuln�rable qui permet d'utiliser une entr�e non valid�e pour cr�er du HTML dynamique dans le contexte de la page�:

```js
var userposition=location.href.indexOf("user=");
var user=location.href.substring(userposition+5);
document.getElementById("Welcome").innerHTML=" Hello, "+user;
```

L'exemple suivant montre du code vuln�rable utilisant la fonction `document.write()`�:

```js
var userposition=location.href.indexOf("user=");
var user=location.href.substring(userposition+5);
document.write("<h1>Hello, " + user +"</h1>");
```

Dans les deux exemples, cette vuln�rabilit� peut �tre exploit�e avec une entr�e telle que�:

```text
http://vulnerable.site/page.html?user=<img%20src='aaa'%20onerror=alert(1)>
```

Cette entr�e ajoutera une balise d'image � la page qui ex�cutera du code JavaScript arbitraire ins�r� par l'utilisateur malveillant dans le contexte HTML.

## Objectifs des tests

- Identifiez les points d'injection HTML et �valuez la gravit� du contenu inject�.

## Comment tester

Consid�rez l'exercice DOM XSS suivant <http://www.domxss.com/domxss/01_Basics/06_jquery_old_html.html>

Le code HTML contient le script suivant�:

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
