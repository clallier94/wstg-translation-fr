# Test de l'injection de cha�ne de format

|ID          |
|------------|
|WSTG-INPV-13|

## Sommaire

Une cha�ne de format est une s�quence de caract�res � terminaison nulle qui contient �galement des sp�cificateurs de conversion interpr�t�s ou convertis au moment de l'ex�cution. Si le code c�t� serveur [concat�ne l'entr�e d'un utilisateur avec une cha�ne de format](https://www.netsparker.com/blog/web-security/string-concatenation-format-string-vulnerabilities/), un attaquant peut ajouter une conversion suppl�mentaire sp�cificateurs pour provoquer une erreur d'ex�cution, la divulgation d'informations ou un d�passement de m�moire tampon.

Le pire des cas pour les vuln�rabilit�s des cha�nes de format se produit dans les langages qui ne v�rifient pas les arguments et incluent �galement un sp�cificateur `%n` qui �crit en m�moire. Ces fonctions, si elles sont exploit�es par un attaquant modifiant une cha�ne de format, pourraient entra�ner [la divulgation d'informations et l'ex�cution de code](https://www.veracode.com/security/format-string)�:

- C et C++ [printf](https://en.cppreference.com/w/c/io/fprintf) et m�thodes similaires fprintf, sprintf, snprintf
- Perl [printf](https://perldoc.perl.org/functions/printf.html) et sprintf

Ces fonctions de cha�ne de format ne peuvent pas �crire dans la m�moire, mais les attaquants peuvent toujours provoquer la divulgation d'informations en modifiant les cha�nes de format en valeurs de sortie que les d�veloppeurs n'avaient pas l'intention d'envoyer�:

- Python 2.6 et 2.7 [str.format](https://docs.python.org/2/library/string.html) et Python 3 unicode [str.format](https://docs.python.org/3 /library/stdtypes.html#str.format) peut �tre modifi� en injectant des cha�nes pouvant pointer vers [d'autres variables](https://lucumr.pocoo.org/2016/12/29/careful-with-str-format/ ) en m�moire

Les fonctions de cha�ne de format suivantes peuvent provoquer des erreurs d'ex�cution si l'attaquant ajoute des sp�cificateurs de conversion�:

- Java [String.format](https://docs.oracle.com/en/java/javase/11/docs/api/java.base/java/lang/String.html#format%28java.util.Locale% 2Cjava.lang.String%2Cjava.lang.Object...%29) et [PrintStream.format](https://docs.oracle.com/en/java/javase/11/docs/api/java.base/ java/io/PrintStream.html#format%2528java.util.Locale%252Cjava.lang.String%252Cjava.lang.Object...%2529)
- PHP [printf](https://www.php.net/manual/es/function.printf.php)

Le mod�le de code qui provoque une vuln�rabilit� de cha�ne de format est un appel � une fonction de format de cha�ne qui contient une entr�e utilisateur non filtr�e. L'exemple suivant montre comment un "printf" de d�bogage peut rendre un programme vuln�rable�:

L'exemple en C :

```c
char *userName = /* input from user controlled field */;

printf("DEBUG Current user: ");
// Vulnerable debugging code
printf(userName);
```

L'exemple en Java :

```java
final String userName = /* input from user controlled field */;

System.out.printf("DEBUG Current user: ");
// Vulnerable code:
System.out.printf(userName);
```

Dans cet exemple particulier, si l'attaquant d�finissait son `userName` pour avoir un ou plusieurs sp�cificateurs de conversion, il y aurait un comportement ind�sirable. L'exemple C [imprimerait le contenu de la m�moire] (https://www.defcon.org/images/defcon-18/dc-18-presentations/Haas/DEFCON-18-Haas-Adv-Format-String-Attacks.pdf ) si `userName` contenait `%p%p%p%p%p`, et il peut corrompre le contenu de la m�moire s'il y a un `%n` dans la cha�ne. Dans l'exemple Java, un `username` contenant n'importe quel sp�cificateur n�cessitant une entr�e (y compris `%x` ou `%s`) entra�nerait le plantage du programme avec `IllegalFormatException`. Bien que les exemples soient toujours sujets � d'autres probl�mes, la vuln�rabilit� peut �tre corrig�e par les arguments printf de `printf("DEBUG Current user: %s", userName)`.

## Objectifs des tests

- �valuer si l'injection de sp�cificateurs de conversion de cha�ne de format dans des champs contr�l�s par l'utilisateur provoque un comportement ind�sirable de l'application.

## Comment tester

Les tests incluent l'analyse du code et l'injection de sp�cificateurs de conversion en tant qu'entr�e utilisateur dans l'application test�e.

### Analyse statique

Les outils d'analyse statique peuvent trouver des vuln�rabilit�s de cha�ne de format dans le code ou dans les binaires. Voici des exemples d'outils�:

- C et C++ : [Flawfinder](https://dwheeler.com/flawfinder/)
- Java�: r�gle FindSecurityBugs [FORMAT_STRING_MANIPULATION](https://find-sec-bugs.github.io/bugs.htm#FORMAT_STRING_MANIPULATION)
- PHP�: Analyseur de formatage de cha�nes dans [phpsa] (https://github.com/ovr/phpsa/blob/master/docs/05_Analyzers.md#function_string_formater)

###�Inspection manuelle des codes

L'analyse statique peut manquer des cas plus subtils, notamment des cha�nes de format g�n�r�es par un code complexe. Pour rechercher manuellement des vuln�rabilit�s dans une base de code, un testeur peut rechercher tous les appels dans la base de code qui acceptent une cha�ne de format et retracer pour s'assurer que les entr�es non fiables ne peuvent pas modifier la cha�ne de format.

### Injection du sp�cificateur de conversion

Les testeurs peuvent v�rifier au niveau du test unitaire ou du test syst�me complet en envoyant des sp�cificateurs de conversion dans n'importe quelle entr�e de cha�ne. [Fuzz](https://owasp.org/www-community/Fuzzing) le programme utilisant tous les sp�cificateurs de conversion pour toutes les langues utilis�es par le syst�me test�. Consultez la page [Attaque de cha�ne de format OWASP](https://owasp.org/www-community/attacks/Format_string_attack) pour les entr�es possibles � utiliser. Si le test �choue, le programme plantera ou affichera une sortie inattendue. Si le test r�ussit, la tentative d'envoi d'un sp�cificateur de conversion doit �tre bloqu�e, ou la cha�ne doit passer par le syst�me sans probl�me comme avec toute autre entr�e valide.

Les exemples dans les sous-sections suivantes ont une URL de cette forme�:

`https://vulnerable_host/userinfo?username=x`

- La valeur contr�l�e par l'utilisateur est `x` (pour le param�tre `username`).

#### Injection manuelle

Les testeurs peuvent effectuer un test manuel � l'aide d'un navigateur Web ou d'autres outils de d�bogage d'API Web. Acc�dez � l'application Web ou au site de sorte que la requ�te comporte des sp�cificateurs de conversion. Notez que la plupart des sp�cificateurs de conversion n�cessitent [encoding](https://tools.ietf.org/html/rfc3986#section-2.1) s'ils sont envoy�s dans une URL, car ils contiennent des caract�res sp�ciaux, notamment `%` et `{`. Le test peut introduire une cha�ne de sp�cificateurs `%s%s%s%n` en naviguant avec l'URL suivante�:

`https://vulnerable_host/userinfo?username=%25s%25s%25s%25n`

Si le site Web est vuln�rable, le navigateur ou l'outil devrait recevoir une erreur, qui peut inclure un d�lai d'attente ou un code de retour HTTP 500.

Le code Java renvoie l'erreur

`java.util.MissingFormatArgumentException: Format specifier '%s'`

Selon l'impl�mentation C, le processus peut se bloquer compl�tement avec `Segmentation Fault`.

#### Fuzzing assist� par outil

Les outils de fuzzing, dont [wfuzz](https://github.com/xmendez/wfuzz), peuvent automatiser les tests d'injection. Pour wfuzz, commencez par un fichier texte (fuzz.txt dans cet exemple) avec une entr�e par ligne�:

fuzz.txt�:

```text
alice
%s%s%s%n
%p%p%p%p%p
{event.__init__.__globals__[CONFIG][SECRET_KEY]}
```

Le fichier `fuzz.txt` contient les �l�ments suivants�:

- Une entr�e valide "alice" pour v�rifier que l'application peut traiter une entr�e normale
- Deux cha�nes avec des sp�cificateurs de conversion de type C
- Un sp�cificateur de conversion Python pour tenter de lire les variables globales

Pour envoyer le fichier d'entr�e de fuzzing � l'application Web test�e, utilisez la commande suivante�:

`wfuzz -c -z file,fuzz.txt,urlencode https://vulnerable_host/userinfo?username=FUZZ`

Dans l'appel ci-dessus, l'argument `urlencode` active l'�chappement appropri� pour les cha�nes et `FUZZ` (avec les lettres majuscules) indique � l'outil o� introduire les entr�es.

Un exemple de sortie est le suivant

```text
ID           Response   Lines    Word     Chars       Payload
===================================================================

000000002:   500        0 L      5 W      142 Ch      "%25s%25s%25s%25n"
000000003:   500        0 L      5 W      137 Ch      "%25p%25p%25p%25p%25p"
000000004:   200        0 L      1 W      48 Ch       "%7Bevent.__init__.__globals__%5BCONFIG%5D%5BSECRET_KEY%5D%7D"
000000001:   200        0 L      1 W      5 Ch        "alice"
```

Le r�sultat ci-dessus valide la faiblesse de l'application face � l'injection de sp�cificateurs de conversion de type C `%s` et `%p`.
