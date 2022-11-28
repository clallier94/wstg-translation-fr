# Test de l'injection de chaîne de format

|ID          |
|------------|
|WSTG-INPV-13|

## Sommaire

Une chaîne de format est une séquence de caractères à terminaison nulle qui contient également des spécificateurs de conversion interprétés ou convertis au moment de l'exécution. Si le code côté serveur [concatène l'entrée d'un utilisateur avec une chaîne de format](https://www.netsparker.com/blog/web-security/string-concatenation-format-string-vulnerabilities/), un attaquant peut ajouter une conversion supplémentaire spécificateurs pour provoquer une erreur d'exécution, la divulgation d'informations ou un dépassement de mémoire tampon.

Le pire des cas pour les vulnérabilités des chaînes de format se produit dans les langages qui ne vérifient pas les arguments et incluent également un spécificateur `%n` qui écrit en mémoire. Ces fonctions, si elles sont exploitées par un attaquant modifiant une chaîne de format, pourraient entraîner [la divulgation d'informations et l'exécution de code](https://www.veracode.com/security/format-string) :

- C et C++ [printf](https://en.cppreference.com/w/c/io/fprintf) et méthodes similaires fprintf, sprintf, snprintf
- Perl [printf](https://perldoc.perl.org/functions/printf.html) et sprintf

Ces fonctions de chaîne de format ne peuvent pas écrire dans la mémoire, mais les attaquants peuvent toujours provoquer la divulgation d'informations en modifiant les chaînes de format en valeurs de sortie que les développeurs n'avaient pas l'intention d'envoyer :

- Python 2.6 et 2.7 [str.format](https://docs.python.org/2/library/string.html) et Python 3 unicode [str.format](https://docs.python.org/3 /library/stdtypes.html#str.format) peut être modifié en injectant des chaînes pouvant pointer vers [d'autres variables](https://lucumr.pocoo.org/2016/12/29/careful-with-str-format/ ) en mémoire

Les fonctions de chaîne de format suivantes peuvent provoquer des erreurs d'exécution si l'attaquant ajoute des spécificateurs de conversion :

- Java [String.format](https://docs.oracle.com/en/java/javase/11/docs/api/java.base/java/lang/String.html#format%28java.util.Locale% 2Cjava.lang.String%2Cjava.lang.Object...%29) et [PrintStream.format](https://docs.oracle.com/en/java/javase/11/docs/api/java.base/ java/io/PrintStream.html#format%2528java.util.Locale%252Cjava.lang.String%252Cjava.lang.Object...%2529)
- PHP [printf](https://www.php.net/manual/es/function.printf.php)

Le modèle de code qui provoque une vulnérabilité de chaîne de format est un appel à une fonction de format de chaîne qui contient une entrée utilisateur non filtrée. L'exemple suivant montre comment un "printf" de débogage peut rendre un programme vulnérable :

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

Dans cet exemple particulier, si l'attaquant définissait son `userName` pour avoir un ou plusieurs spécificateurs de conversion, il y aurait un comportement indésirable. L'exemple C [imprimerait le contenu de la mémoire] (https://www.defcon.org/images/defcon-18/dc-18-presentations/Haas/DEFCON-18-Haas-Adv-Format-String-Attacks.pdf ) si `userName` contenait `%p%p%p%p%p`, et il peut corrompre le contenu de la mémoire s'il y a un `%n` dans la chaîne. Dans l'exemple Java, un `username` contenant n'importe quel spécificateur nécessitant une entrée (y compris `%x` ou `%s`) entraînerait le plantage du programme avec `IllegalFormatException`. Bien que les exemples soient toujours sujets à d'autres problèmes, la vulnérabilité peut être corrigée par les arguments printf de `printf("DEBUG Current user: %s", userName)`.

## Objectifs des tests

- Évaluer si l'injection de spécificateurs de conversion de chaîne de format dans des champs contrôlés par l'utilisateur provoque un comportement indésirable de l'application.

## Comment tester

Les tests incluent l'analyse du code et l'injection de spécificateurs de conversion en tant qu'entrée utilisateur dans l'application testée.

### Analyse statique

Les outils d'analyse statique peuvent trouver des vulnérabilités de chaîne de format dans le code ou dans les binaires. Voici des exemples d'outils :

- C et C++ : [Flawfinder](https://dwheeler.com/flawfinder/)
- Java : règle FindSecurityBugs [FORMAT_STRING_MANIPULATION](https://find-sec-bugs.github.io/bugs.htm#FORMAT_STRING_MANIPULATION)
- PHP : Analyseur de formatage de chaînes dans [phpsa] (https://github.com/ovr/phpsa/blob/master/docs/05_Analyzers.md#function_string_formater)

### Inspection manuelle des codes

L'analyse statique peut manquer des cas plus subtils, notamment des chaînes de format générées par un code complexe. Pour rechercher manuellement des vulnérabilités dans une base de code, un testeur peut rechercher tous les appels dans la base de code qui acceptent une chaîne de format et retracer pour s'assurer que les entrées non fiables ne peuvent pas modifier la chaîne de format.

### Injection du spécificateur de conversion

Les testeurs peuvent vérifier au niveau du test unitaire ou du test système complet en envoyant des spécificateurs de conversion dans n'importe quelle entrée de chaîne. [Fuzz](https://owasp.org/www-community/Fuzzing) le programme utilisant tous les spécificateurs de conversion pour toutes les langues utilisées par le système testé. Consultez la page [Attaque de chaîne de format OWASP](https://owasp.org/www-community/attacks/Format_string_attack) pour les entrées possibles à utiliser. Si le test échoue, le programme plantera ou affichera une sortie inattendue. Si le test réussit, la tentative d'envoi d'un spécificateur de conversion doit être bloquée, ou la chaîne doit passer par le système sans problème comme avec toute autre entrée valide.

Les exemples dans les sous-sections suivantes ont une URL de cette forme :

`https://vulnerable_host/userinfo?username=x`

- La valeur contrôlée par l'utilisateur est `x` (pour le paramètre `username`).

#### Injection manuelle

Les testeurs peuvent effectuer un test manuel à l'aide d'un navigateur Web ou d'autres outils de débogage d'API Web. Accédez à l'application Web ou au site de sorte que la requête comporte des spécificateurs de conversion. Notez que la plupart des spécificateurs de conversion nécessitent [encoding](https://tools.ietf.org/html/rfc3986#section-2.1) s'ils sont envoyés dans une URL, car ils contiennent des caractères spéciaux, notamment `%` et `{`. Le test peut introduire une chaîne de spécificateurs `%s%s%s%n` en naviguant avec l'URL suivante :

`https://vulnerable_host/userinfo?username=%25s%25s%25s%25n`

Si le site Web est vulnérable, le navigateur ou l'outil devrait recevoir une erreur, qui peut inclure un délai d'attente ou un code de retour HTTP 500.

Le code Java renvoie l'erreur

`java.util.MissingFormatArgumentException: Format specifier '%s'`

Selon l'implémentation C, le processus peut se bloquer complètement avec `Segmentation Fault`.

#### Fuzzing assisté par outil

Les outils de fuzzing, dont [wfuzz](https://github.com/xmendez/wfuzz), peuvent automatiser les tests d'injection. Pour wfuzz, commencez par un fichier texte (fuzz.txt dans cet exemple) avec une entrée par ligne :

fuzz.txt :

```text
alice
%s%s%s%n
%p%p%p%p%p
{event.__init__.__globals__[CONFIG][SECRET_KEY]}
```

Le fichier `fuzz.txt` contient les éléments suivants :

- Une entrée valide "alice" pour vérifier que l'application peut traiter une entrée normale
- Deux chaînes avec des spécificateurs de conversion de type C
- Un spécificateur de conversion Python pour tenter de lire les variables globales

Pour envoyer le fichier d'entrée de fuzzing à l'application Web testée, utilisez la commande suivante :

`wfuzz -c -z file,fuzz.txt,urlencode https://vulnerable_host/userinfo?username=FUZZ`

Dans l'appel ci-dessus, l'argument `urlencode` active l'échappement approprié pour les chaînes et `FUZZ` (avec les lettres majuscules) indique à l'outil où introduire les entrées.

Un exemple de sortie est le suivant

```text
ID           Response   Lines    Word     Chars       Payload
===================================================================

000000002:   500        0 L      5 W      142 Ch      "%25s%25s%25s%25n"
000000003:   500        0 L      5 W      137 Ch      "%25p%25p%25p%25p%25p"
000000004:   200        0 L      1 W      48 Ch       "%7Bevent.__init__.__globals__%5BCONFIG%5D%5BSECRET_KEY%5D%7D"
000000001:   200        0 L      1 W      5 Ch        "alice"
```

Le résultat ci-dessus valide la faiblesse de l'application face à l'injection de spécificateurs de conversion de type C `%s` et `%p`.
