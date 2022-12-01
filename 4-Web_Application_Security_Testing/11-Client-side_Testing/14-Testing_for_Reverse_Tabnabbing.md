# Test de Tabnabbing inversé

|ID          |
|------------|
|WSTG-CLNT-14|

## Sommaire

[Reverse Tabnabbing](https://owasp.org/www-community/attacks/Reverse_Tabnabbing) est une attaque qui peut être utilisée pour rediriger les utilisateurs vers des pages de phishing. Cela devient généralement possible grâce à l'attribut `target` de la balise `<a>` défini sur `_blank`, ce qui entraîne l'ouverture du lien dans un nouvel onglet. Lorsque l'attribut `rel='noopener noreferrer'` n'est pas utilisé dans la même balise `<a>`, la page nouvellement ouverte peut influencer la page d'origine et la rediriger vers un domaine contrôlé par l'attaquant.

Étant donné que l'utilisateur se trouvait sur le domaine d'origine lorsque le nouvel onglet s'est ouvert, il est moins susceptible de remarquer que la page a changé, surtout si la page de phishing est identique au domaine d'origine. Toutes les informations d'identification saisies sur le domaine contrôlé par l'attaquant se retrouveront ainsi en possession de l'attaquant.

Les liens ouverts via la fonction JavaScript `window.open` sont également vulnérables à cette attaque.

_REMARQUE : Il s'agit d'un problème hérité qui n'affecte pas les [navigateurs modernes] (https://caniuse.com/mdn-html_elements_a_implicit_noopener). Les anciennes versions des navigateurs populaires (par exemple, les versions antérieures à Google Chrome 88) ainsi qu'Internet Explorer sont vulnérables à cette attaque._

### exemple

Imaginez une application Web où les utilisateurs sont autorisés à insérer une URL dans leur profil. Si l'application est vulnérable au reverse tabnabbing, un utilisateur malveillant pourra fournir un lien vers une page contenant le code suivant :

```html
<html>
 <body>
  <script>
    window.opener.location = "https://exemple.org";
  </script>
<b>Error loading...</b>
 </body>
</html>
```

Cliquer sur le lien ouvrira un nouvel onglet tandis que l'onglet d'origine redirigera vers "exemple.org". Supposons que "exemple.org" ressemble à l'application Web vulnérable, l'utilisateur est moins susceptible de remarquer le changement et est plus susceptible de saisir des informations sensibles sur la page.

## Comment tester

- Vérifiez la source HTML de l'application pour voir si les liens avec `target="_blank"` utilisent les mots clés `noopener` et `noreferrer` dans l'attribut `rel`. Si ce n'est pas le cas, il est probable que l'application soit vulnérable au reverse tabnabbing. Un tel lien devient exploitable s'il pointe vers un site tiers compromis par l'attaquant ou s'il est contrôlé par l'utilisateur.
- Vérifie les zones où un attaquant peut insérer des liens, c'est-à-dire contrôler l'argument `href` d'une balise `<a>`. Essayez d'insérer un lien vers une page qui a le code source donné dans l'exemple ci-dessus, et voyez si le domaine d'origine redirige. Ce test peut être fait dans IE si les autres navigateurs ne fonctionnent pas.

## Correction

Il est recommandé de s'assurer que l'attribut HTML `rel` est défini avec les mots clés `noreferrer` et `noopener` pour tous les liens.

## Références

- [Tabnabbing - Aide-mémoire HTML5](https://cheatsheetseries.owasp.org/cheatsheets/HTML5_Security_Cheat_Sheet.html#tabnabbing)
- [La vulnérabilité target="_blank" par exemple](https://dev.to/ben/the-targetblank-vulnerability-by-exemple)
- [À propos de rel=noopener](https://mathiasbynens.github.io/rel-noopener/)
- [Target="_blank" — la vulnérabilité la plus sous-estimée de tous les temps](https://medium.com/@jitbit/target-blank-the-most-underestimated-vulnerability-ever-96e328301f4c)
- [La vulnérabilité de tabnabbing inversé affecte IBM Business Automation Workflow et IBM Business Process Manager](https://www.ibm.com/support/pages/security-bulletin-reverse-tabnabbing-vulnerability-affects-ibm-business-automation-workflow -et-ibm-business-process-manager-bpm-cve-2020-4490-0)
