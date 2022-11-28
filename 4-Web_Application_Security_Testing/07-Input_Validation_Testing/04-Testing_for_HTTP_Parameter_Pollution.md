# Test de la pollution des paramètres HTTP

|ID          |
|------------|
|WSTG-INPV-04|

## Sommaire

HTTP Parameter Pollution teste la réponse des applications à la réception de plusieurs paramètres HTTP portant le même nom ; par exemple, si le paramètre `username` est inclus deux fois dans les paramètres GET ou POST.

La fourniture de plusieurs paramètres HTTP avec le même nom peut amener une application à interpréter des valeurs de manière imprévue. En exploitant ces effets, un attaquant peut être en mesure de contourner la validation des entrées, de déclencher des erreurs d'application ou de modifier les valeurs des variables internes. Étant donné que la pollution des paramètres HTTP (en abrégé * HPP *) affecte un élément constitutif de toutes les technologies Web, des attaques côté serveur et côté client existent.

Les normes HTTP actuelles n'incluent pas de conseils sur la façon d'interpréter plusieurs paramètres d'entrée portant le même nom. Par exemple, [RFC 3986](https://www.ietf.org/rfc/rfc3986.txt) définit simplement le terme *Query String* comme une série de paires champ-valeur et [RFC 2396](https:// www.ietf.org/rfc/rfc2396.txt) définit les classes de caractères de chaîne de requête inversés et non réservés. Sans norme en place, les composants d'application Web gèrent ce cas marginal de différentes manières (voir le tableau ci-dessous pour plus de détails).

En soi, ce n'est pas nécessairement une indication de vulnérabilité. Cependant, si le développeur n'est pas conscient du problème, la présence de paramètres dupliqués peut produire un comportement anormal dans l'application qui peut être potentiellement exploité par un attaquant. Comme souvent en sécurité, les comportements inattendus sont une source habituelle de faiblesses pouvant conduire dans ce cas à des attaques HTTP Parameter Pollution. Pour mieux présenter cette classe de vulnérabilités et le résultat des attaques HPP, il est intéressant d'analyser quelques exemples réels qui ont été découverts dans le passé.

### Validation des entrées et contournement des filtres

En 2009, immédiatement après la publication de la première recherche sur la pollution des paramètres HTTP, la technique a attiré l'attention de la communauté de la sécurité comme un moyen possible de contourner les pare-feu des applications Web.

L'une de ces failles, affectant *ModSecurity SQL Injection Core Rules*, représente un exemple parfait de l'inadéquation d'impédance entre les applications et les filtres. Le filtre ModSecurity appliquerait correctement une liste de refus pour la chaîne suivante : `select 1,2,3 from table`, empêchant ainsi le traitement de cet exemple d'URL par le serveur Web : `/index.aspx?page=select 1,2 ,3 du tableau`. Cependant, en exploitant la concaténation de plusieurs paramètres HTTP, un attaquant pourrait amener le serveur d'applications à concaténer la chaîne après que le filtre ModSecurity ait déjà accepté l'entrée. Par exemple, l'URL `/index.aspx?page=select 1&page=2,3` de la table ne déclencherait pas le filtre ModSecurity, mais la couche application concaténerait l'entrée dans la chaîne malveillante complète.

Une autre vulnérabilité HPP s'est avérée affecter *Apple Cups*, le système d'impression bien connu utilisé par de nombreux systèmes UNIX. En exploitant HPP, un attaquant pourrait facilement déclencher une vulnérabilité de type "Cross-Site Scripting" en utilisant l'URL suivante : `http://127.0.0.1:631/admin/?kerberos=onmouseover=alert(1)&kerberos`. Le point de contrôle de validation de l'application peut être contourné en ajoutant un argument supplémentaire "kerberos" ayant une chaîne valide (par exemple, une chaîne vide). Comme le point de contrôle de validation ne prendrait en compte que la deuxième occurrence, le premier paramètre `kerberos` n'a pas été correctement filtré avant d'être utilisé pour générer du contenu HTML dynamique. Une exploitation réussie entraînerait l'exécution de code JavaScript dans le contexte du site Web d'hébergement.

### Contournement de l'authentification

Une vulnérabilité HPP encore plus critique a été découverte dans *Blogger*, la plate-forme de blogs populaire. Le bogue permettait à des utilisateurs malveillants de s'approprier le blog de la victime en utilisant la requête HTTP suivante (`https://www.blogger.com/add-authors.do`) :

```html
POST /add-authors.do HTTP/1.1
[...]

security_token=attackertoken&blogID=attackerblogidvalue&blogID=victimblogidvalue&authorsList=goldshlager19test%40gmail.com(attacker email)&ok=Invite
```

La faille résidait dans le mécanisme d'authentification utilisé par l'application Web, car le contrôle de sécurité était effectué sur le premier paramètre `blogID`, alors que l'opération réelle utilisait la seconde occurrence.

### Comportement attendu par le serveur d'applications

Le tableau suivant illustre le comportement de différentes technologies Web en présence de plusieurs occurrences du même paramètre HTTP.

Étant donné l'URL et la chaîne de requête : `http://example.com/?color=red&color=blue`

  | Backend du serveur d'applications Web | Résultat d'analyse | Exemple |
  |---------------------------------------|--------------------|---------|
  | ASP.NET / IIS | Toutes les occurrences concaténées avec une virgule | couleur=rouge,bleu |
  | ASP/IIS | Toutes les occurrences concaténées avec une virgule | couleur=rouge,bleu |
  | .NET Core 3.1 / Crécerelle | Toutes les occurrences concaténées avec une virgule | couleur=rouge,bleu |
  | .NET 5 / Crécerelle | Toutes les occurrences concaténées avec une virgule | couleur=rouge,bleu |
  | PHP/Apache | Dernière occurrence uniquement | couleur=bleu |
  | PHP / Zeus | Dernière occurrence uniquement | couleur=bleu |
  | JSP, Servlet / Apache Tomcat | Première occurrence uniquement | couleur=rouge |
  | JSP, servlet/serveur d'application Oracle 10g | Première occurrence uniquement | couleur=rouge |
  | JSP, Servlet / Jetty | Première occurrence uniquement | couleur=rouge |
  | IBMLotus Domino | Dernière occurrence uniquement | couleur=bleu |
  | Serveur HTTP IBM | Première occurrence uniquement | couleur=rouge |
  | nœud.js / express | Première occurrence uniquement | couleur=rouge |
  | mod_perl, libapreq2 / Apache | Première occurrence uniquement | couleur=rouge |
  | CGI Perl / Apache | Première occurrence uniquement | couleur=rouge |
  | mod_wsgi (Python) / Apache | Première occurrence uniquement | couleur=rouge |
  | Python / Zopé | Toutes les occurrences dans le type de données List | couleur=['rouge','bleu'] |

(source : [Appsec EU 2009 Carettoni & Paola](https://owasp.org/www-pdf-archive/AppsecEU09_CarettoniDiPaola_v0.8.pdf))

## Objectifs des tests

- Identifier le backend et la méthode d'analyse utilisée.
- Évaluez les points d'injection et essayez de contourner les filtres d'entrée à l'aide de HPP.

## Comment tester

Heureusement, étant donné que l'attribution des paramètres HTTP est généralement gérée via le serveur d'applications Web, et non le code d'application lui-même, le test de la réponse à la pollution des paramètres doit être standard sur toutes les pages et actions. Cependant, comme une connaissance approfondie de la logique métier est nécessaire, tester HPP nécessite des tests manuels. Les outils automatiques ne peuvent aider que partiellement les auditeurs car ils ont tendance à générer trop de faux positifs. De plus, HPP peut se manifester dans les composants côté client et côté serveur.

### HPP côté serveur

Pour tester les vulnérabilités HPP, identifiez tout formulaire ou action qui autorise la saisie fournie par l'utilisateur. Les paramètres de chaîne de requête dans les requêtes HTTP GET sont faciles à modifier dans la barre de navigation du navigateur. Si l'action de formulaire soumet des données via POST, le testeur devra utiliser un proxy d'interception pour falsifier les données POST lorsqu'elles sont envoyées au serveur. Après avoir identifié un paramètre d'entrée particulier à tester, on peut modifier les données GET ou POST en interceptant la requête, ou modifier la chaîne de requête après le chargement de la page de réponse. Pour tester les vulnérabilités HPP, ajoutez simplement le même paramètre aux données GET ou POST mais avec une valeur différente attribuée.

Par exemple : si vous testez le paramètre `search_string` dans la chaîne de requête, l'URL de la requête inclura le nom et la valeur de ce paramètre :

```text
http://example.com/?search_string=kittens
```

Le paramètre particulier peut être caché parmi plusieurs autres paramètres, mais l'approche est la même ; laissez les autres paramètres en place et ajoutez le duplicata :

```text
http://example.com/?mode=guest&search_string=kittens&num_results=100
```

Ajoutez le même paramètre avec une valeur différente :

```text
http://example.com/?mode=guest&search_string=kittens&num_results=100&search_string=puppies
```

et soumettre la nouvelle demande.

Analysez la page de réponse pour déterminer quelles valeurs ont été analysées. Dans l'exemple ci-dessus, les résultats de la recherche peuvent afficher `chatons`, `chiots`, une combinaison des deux (`chatons, chiots` ou `chatons~chiots` ou `['chatons','chiots']`), peut donner un résultat vide ou une page d'erreur.

Ce comportement, qu'il s'agisse d'utiliser le premier, le dernier ou une combinaison de paramètres d'entrée portant le même nom, est très susceptible d'être cohérent dans l'ensemble de l'application. Que ce comportement par défaut révèle ou non une vulnérabilité potentielle dépend de la validation d'entrée et du filtrage spécifiques à une application particulière. En règle générale : si la validation des entrées existantes et d'autres mécanismes de sécurité sont suffisants sur des entrées uniques, et si le serveur n'affecte que le premier ou le dernier paramètre pollué, alors la pollution des paramètres ne révèle pas de vulnérabilité. Si les paramètres en double sont concaténés, que différents composants d'application Web utilisent des occurrences différentes ou que les tests génèrent une erreur, il y a une probabilité accrue de pouvoir utiliser la pollution des paramètres pour déclencher des vulnérabilités de sécurité.

Une analyse plus approfondie nécessiterait trois requêtes HTTP pour chaque paramètre HTTP :

1. Soumettez une requête HTTP contenant le nom et la valeur du paramètre standard et enregistrez la réponse HTTP. Par exemple. `page?par1=val1`
2. Remplacez la valeur du paramètre par une valeur falsifiée, soumettez et enregistrez la réponse HTTP. Par exemple. `page?par1=HPP_TEST1`
3. Envoyez une nouvelle requête combinant les étapes (1) et (2). Encore une fois, enregistrez la réponse HTTP. Par exemple. `page?par1=val1&par1=HPP_TEST1`
4. Comparez les réponses obtenues lors de toutes les étapes précédentes. Si la réponse de (3) est différente de (1) et que la réponse de (3) est également différente de (2), il existe une inadéquation d'impédance qui peut éventuellement être abusée pour déclencher des vulnérabilités HPP.

Créer un exploit complet à partir d'une faiblesse de pollution de paramètres dépasse le cadre de ce texte. Voir les références pour des exemples et des détails.

### HPP côté client

Comme pour HPP côté serveur, les tests manuels sont la seule technique fiable pour auditer les applications Web afin de détecter les vulnérabilités de pollution des paramètres affectant les composants côté client. Alors que dans la variante côté serveur, l'attaquant exploite une application Web vulnérable pour accéder à des données protégées ou pour effectuer des actions qui ne sont pas autorisées ou ne sont pas censées être exécutées, les attaques côté client visent à subvertir les composants et technologies côté client.

Pour tester les vulnérabilités côté client HPP, identifiez tout formulaire ou action qui autorise la saisie de l'utilisateur et affiche le résultat de cette saisie à l'utilisateur. Une page de recherche est idéale, mais une boîte de connexion peut ne pas fonctionner (car elle peut ne pas afficher un nom d'utilisateur invalide à l'utilisateur).

Comme pour HPP côté serveur, polluez chaque paramètre HTTP avec `%26HPP_TEST` et recherchez les occurrences *url-decodé* de la charge utile fournie par l'utilisateur :

- `&HPP_TEST`
- `&amp;HPP_TEST`
- etc.

En particulier, faites attention aux réponses ayant des vecteurs HPP dans les attributs `data`, `src`, `href` ou les actions de formulaires. Là encore, le fait que ce comportement par défaut révèle ou non une vulnérabilité potentielle dépend de la validation d'entrée spécifique, du filtrage et de la logique métier de l'application. En outre, il est important de noter que cette vulnérabilité peut également affecter les paramètres de chaîne de requête utilisés dans XMLHttpRequest (XHR), la création d'attributs d'exécution et d'autres technologies de plug-in (par exemple, les variables flashvars d'Adobe Flash).

## Outils

- [Analyseurs passifs/actifs OWASP ZAP] (https://www.zaproxy.org)

## Références

### Papiers blanc

- [Pollution des paramètres HTTP - Luca Carettoni, Stefano di Paola](https://owasp.org/www-pdf-archive/AppsecEU09_CarettoniDiPaola_v0.8.pdf)
- [Exemple de pollution des paramètres HTTP côté client (défaut Yahoo! Classic Mail) - Stefano di Paola](https://blog.mindedsecurity.com/2009/05/client-side-http-parameter-pollution.html)
- [Comment détecter les attaques par pollution des paramètres HTTP - Chrysostomos Daniel](https://www.acunetix.com/blog/whitepaper-http-parameter-pollution/)
- [CAPEC-460 : Pollution des paramètres HTTP (HPP) - Evgeny Lebanidze](https://capec.mitre.org/data/definitions/460.html)
- [Découverte automatisée des vulnérabilités de pollution des paramètres dans les applications Web - Marco Balduzzi, Carmen Torrano Gimenez, Davide Balzarotti, Engin Kirda](http://s3.eurecom.fr/docs/ndss11_hpp.pdf)
