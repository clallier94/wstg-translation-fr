# Test de l'injection de mod�le c�t� serveur

|ID          |
|------------|
|WSTG-INPV-18|

## Sommaire

Les applications Web utilisent couramment des technologies de mod�lisation c�t� serveur (Jinja2, Twig, FreeMaker, etc.) pour g�n�rer des r�ponses HTML dynamiques. Les vuln�rabilit�s d'injection de mod�le c�t� serveur (SSTI) se produisent lorsque l'entr�e de l'utilisateur est int�gr�e dans un mod�le de mani�re non s�curis�e et entra�ne l'ex�cution de code � distance sur le serveur. Toutes les fonctionnalit�s prenant en charge le balisage avanc� fourni par l'utilisateur peuvent �tre vuln�rables au SSTI, y compris les pages wiki, les critiques, les applications marketing, les syst�mes CMS, etc. Certains moteurs de mod�les utilisent divers m�canismes (par exemple, bac � sable, autoriser la liste, etc.)

### Exemple - Brindille

L'exemple suivant est un extrait du projet [Extreme Vulnerable Web Application](https://github.com/s4n7h0/xvwa).

```php
public function getFilter($name)
{
        [snip]
        foreach ($this->filterCallbacks as $callback) {
        if (false !== $filter = call_user_func($callback, $name)) {
            return $filter;
        }
    }
    return false;
}
```

Dans la fonction getFilter, `call_user_func($callback, $name)` est vuln�rable � SSTI�: le param�tre `name` est extrait de la requ�te HTTP GET et ex�cut� par le serveur�:

![Exemple SSTI XVWA](images/SSTI_XVWA.jpeg)\
*Figure 4.7.18-1 : Exemple SSTI XVWA*

### Exemple - Flacon/Jinja2

L'exemple suivant utilise le moteur de template Flask et Jinja2. La fonction `page` accepte un param�tre 'name' d'une requ�te HTTP GET et affiche une r�ponse HTML avec le contenu de la variable `name`�:

```python
@app.route("/page")
def page():
    name = request.values.get('name')
    output = Jinja2.from_string('Hello ' + name + '!').render()
    return output
```

Cet extrait de code est vuln�rable � XSS mais il est �galement vuln�rable � SSTI. Utiliser ce qui suit comme charge utile dans le param�tre `name`�:

```bash
$ curl -g 'http://www.target.com/page?name={{7*7}}'
Hello 49!
```

## Objectifs des tests

- D�tecter les points de vuln�rabilit� d'injection de template.
- Identifier le moteur de template.
- Construire l'exploit.

## Comment tester

Les vuln�rabilit�s SSTI existent dans le contexte du texte ou du code. Dans un contexte de texte en clair, les utilisateurs sont autoris�s � utiliser du 'texte' libre avec du code HTML direct. Dans le contexte du code, l'entr�e de l'utilisateur peut �galement �tre plac�e dans une instruction de mod�le (par exemple, dans un nom de variable)

### Identifier la vuln�rabilit� d'injection de mod�le

La premi�re �tape du test de SSTI dans un contexte de texte en clair consiste � construire des expressions de mod�le communes utilis�es par divers moteurs de mod�le en tant que charges utiles et � surveiller les r�ponses du serveur pour identifier quelle expression de mod�le a �t� ex�cut�e par le serveur.

Exemples d'expressions de mod�le courantes�:

```text
a{{bar}}b
a{{7*7}}
{var} ${var} {{var}} <%var%> [% var %]
```

Dans cette �tape, une [cha�nes de test d'expression de mod�le/liste de charges utiles](https://github.com/swisskyrepo/PayloadsAllTheThings/tree/master/Server%20Side%20Template%20Injection) compl�te est recommand�e.

Le test de SSTI dans le contexte du code est l�g�rement diff�rent. Tout d'abord, le testeur construit la requ�te qui aboutit � des r�ponses vides ou erron�es du serveur. Dans l'exemple ci-dessous, le param�tre HTTP GET est ins�r� dans la variable `personal_greeting` dans une d�claration de mod�le�:

```text
personal_greeting=username
Hello user01
```

En utilisant la charge utile suivante - la r�ponse du serveur est vide "Hello":

```text
personal_greeting=username<tag>
Hello
```

L'�tape suivante consiste � sortir de l'instruction de mod�le et � injecter la balise HTML apr�s celle-ci en utilisant la charge utile suivante

```text
personal_greeting=username}}<tag>
Hello user01 <tag>
```

### Identifier le moteur de template

Sur la base des informations de l'�tape pr�c�dente, le testeur doit maintenant identifier le moteur de mod�le utilis� en fournissant diverses expressions de mod�le. Sur la base des r�ponses du serveur, le testeur d�duit le moteur de template utilis�. Cette approche manuelle est d�crite plus en d�tail dans [cet](https://portswigger.net/blog/server-side-template-injection?#Identify) article PortSwigger. Pour automatiser l'identification de la vuln�rabilit� SSTI et le moteur de template, divers outils sont disponibles, notamment [Tplmap](https://github.com/epinna/tplmap) ou l'[extension Backslash Powered Scanner Burp Suite](https://github. com/PortSwigger/backslash-powered-scanner).

### Construire l'exploit RCE

L'objectif principal de cette �tape est d'identifier pour obtenir un contr�le suppl�mentaire sur le serveur avec un exploit RCE en �tudiant la documentation et la recherche du mod�le. Les principaux domaines d'int�r�t sont�:

- Sections **Pour les auteurs de mod�les** couvrant la syntaxe de base.
- Sections **Consid�rations de s�curit�**.
- Listes de m�thodes, fonctions, filtres et variables int�gr�s.
- Listes d'extensions/plugins.

Le testeur peut �galement identifier quels autres objets, m�thodes et propri�t�s peuvent �tre expos�s en se concentrant sur l'objet "soi". Si l'objet `self` n'est pas disponible et que la documentation ne r�v�le pas les d�tails techniques, une force brute du nom de la variable est recommand�e. Une fois l'objet identifi�, l'�tape suivante consiste � parcourir l'objet pour identifier toutes les m�thodes, propri�t�s et attributs accessibles via le moteur de mod�le. Cela pourrait conduire � d'autres types de r�sultats de s�curit�, notamment des escalades de privil�ges, la divulgation d'informations sur les mots de passe d'application, les cl�s API, les configurations et les variables d'environnement, etc.

## Outils

- [Tplmap] (https://github.com/epinna/tplmap)
- [Extension Backslash Powered Scanner Burp Suite] (https://github.com/PortSwigger/backslash-powered-scanner)
- [Cha�nes de test d'expression de mod�le/liste de charges utiles] (https://github.com/swisskyrepo/PayloadsAllTheThings/tree/master/Server%20Side%20Template%20Injection)

## R�f�rences

- [James Kettle�: Injection de mod�les c�t� serveur�: RCE pour l'application Web moderne (livre blanc)] (https://portswigger.net/kb/papers/serversidetemplateinjection.pdf)
- [Injection de mod�le c�t� serveur] (https://portswigger.net/blog/server-side-template-injection)
- [Explorer SSTI dans Flask/Jinja2](https://www.lanmaster53.com/2016/03/exploring-ssti-flask-jinja2/)
- [Injection de mod�le c�t� serveur�: de la d�tection au shell distant] (https://www.okiok.com/server-side-template-injection-from-detection-to-remote-shell/)
- [Application Web extr�mement vuln�rable] (https://github.com/s4n7h0/xvwa)
- [Divine Selorm Tsa�: Exploitation de l'injection de mod�le c�t� serveur avec tplmap](https://owasp.org/www-pdf-archive/Owasp_SSTI_final.pdf)
- [Exploitation de SSTI dans Thymeleaf](https://www.acunetix.com/blog/web-security-zone/exploiting-ssti-in-thymeleaf/)
