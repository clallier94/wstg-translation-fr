# Test de la redirection d'URL côté client

|ID          |
|------------|
|WSTG-CLNT-04|

## Sommaire

Cette section décrit comment vérifier la redirection d'URL côté client, également appelée redirection ouverte. Il s'agit d'un défaut de validation d'entrée qui existe lorsqu'une application accepte une entrée contrôlée par l'utilisateur qui spécifie un lien menant à une URL externe qui pourrait être malveillante. Ce type de vulnérabilité pourrait être utilisé pour réaliser une attaque de phishing ou rediriger une victime vers une page d'infection.

Cette vulnérabilité se produit lorsqu'une application accepte une entrée non approuvée contenant une valeur d'URL et ne la nettoie pas. Cette valeur d'URL pourrait amener l'application Web à rediriger l'utilisateur vers une autre page, telle qu'une page malveillante contrôlée par l'attaquant.

Cette vulnérabilité peut permettre à un attaquant de lancer avec succès une escroquerie par hameçonnage et de voler les informations d'identification de l'utilisateur. Étant donné que la redirection provient de l'application réelle, les tentatives de phishing peuvent avoir une apparence plus fiable.

Voici un exemple d'URL d'attaque de phishing.

```texte
http://www.target.site?#redirect=www.fake-target.site
```

La victime qui visite cette URL sera automatiquement redirigée vers `fake-target.site`, où un attaquant pourrait placer une fausse page qui ressemble au site prévu, afin de voler les informations d'identification de la victime.

La redirection ouverte pourrait également être utilisée pour créer une URL qui contournerait les vérifications de contrôle d'accès de l'application et redirigerait l'attaquant vers des fonctions privilégiées auxquelles il ne serait normalement pas en mesure d'accéder.

## Objectifs des tests

- Identifiez les points d'injection qui gèrent les URL ou les chemins.
- Évaluer les emplacements vers lesquels le système pourrait rediriger.

## Comment tester

Lorsque les testeurs recherchent manuellement ce type de vulnérabilité, ils identifient d'abord s'il existe des redirections côté client implémentées dans le code côté client. Ces redirections peuvent être implémentées, pour donner un exemple JavaScript, en utilisant l'objet `window.location`. Cela peut être utilisé pour diriger le navigateur vers une autre page en lui attribuant simplement une chaîne. Ceci est démontré dans l'extrait suivant :

```js
var redir = location.hash.substring(1);
if (redir) {
    window.location='http://'+decodeURIComponent(redir);
}
```

Dans cet exemple, le script n'effectue aucune validation de la variable `redir` qui contient l'entrée fournie par l'utilisateur via la chaîne de requête. Puisqu'aucune forme d'encodage n'est appliquée, cette entrée non validée est transmise à l'objet `windows.location`, créant une vulnérabilité de redirection d'URL.

Cela implique qu'un attaquant pourrait rediriger la victime vers un site malveillant simplement en soumettant la chaîne de requête suivante :

```text
http://www.victim.site/?#www.malicious.site
```

Avec une légère modification, l'exemple d'extrait ci-dessus peut être vulnérable à l'injection de JavaScript.

```js
var redir = location.hash.substring(1);
if (redir) {
    window.location=decodeURIComponent(redir);
}
```

Cela peut être exploité en soumettant la chaîne de requête suivante :

```text
http://www.victim.site/?#javascript:alert(document.cookie)
```

Lors du test de cette vulnérabilité, considérez que certains caractères sont traités différemment par différents navigateurs. Pour référence, voir [XSS basé sur DOM](https://owasp.org/www-community/attacks/DOM_Based_XSS).
