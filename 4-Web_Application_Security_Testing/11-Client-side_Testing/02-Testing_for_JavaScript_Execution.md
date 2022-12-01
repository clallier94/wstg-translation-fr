# Test d'ex�cution JavaScript

|ID          |
|------------|
|WSTG-CLNT-02|

## Sommaire

Une vuln�rabilit� d'injection JavaScript est un sous-type de script intersite (XSS) qui implique la capacit� d'injecter du code JavaScript arbitraire qui est ex�cut� par l'application dans le navigateur de la victime. Cette vuln�rabilit� peut avoir de nombreuses cons�quences, comme la divulgation des cookies de session d'un utilisateur qui pourraient �tre utilis�s pour usurper l'identit� de la victime, ou, plus g�n�ralement, elle peut permettre � l'attaquant de modifier le contenu de la page vue par les victimes ou le comportement de l'application.

Des vuln�rabilit�s d'injection JavaScript peuvent se produire lorsque l'application ne dispose pas d'une validation d'entr�e et de sortie appropri�e fournie par l'utilisateur. Comme JavaScript est utilis� pour peupler dynamiquement les pages web, cette injection se produit pendant cette phase de traitement du contenu et affecte par cons�quent la victime.

Lors du test de cette vuln�rabilit�, consid�rez que certains caract�res sont trait�s diff�remment par diff�rents navigateurs. Pour r�f�rence, voir [XSS bas� sur DOM](https://owasp.org/www-community/attacks/DOM_Based_XSS).

Voici un exemple de script qui n'effectue aucune validation de la variable `rr`. La variable contient une entr�e fournie par l'utilisateur via la cha�ne de requ�te et n'applique en outre aucune forme d'encodage�:

```js
var rr = location.search.substring(1);
if(rr) {
    window.location=decodeURIComponent(rr);
}
```

Cela implique qu'un attaquant pourrait injecter du code JavaScript simplement en soumettant la cha�ne de requ�te suivante�: `www.victim.com/?javascript:alert(1)`.

## Objectifs des tests

- Identifier les puits et les points d'injection JavaScript possibles.

## Comment tester

Consid�rez ce qui suit�: [Exercice DOM XSS](http://www.domxss.com/domxss/01_Basics/04_eval.html)

La page contient le script suivant�:

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

Le code ci-dessus contient une source `location.hash` qui est contr�l�e par l'attaquant qui peut injecter directement dans la valeur `message` un code JavaScript pour prendre le contr�le du navigateur de l'utilisateur.
