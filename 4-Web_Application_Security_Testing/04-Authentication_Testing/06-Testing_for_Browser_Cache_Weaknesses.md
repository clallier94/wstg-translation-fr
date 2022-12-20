# Test des faiblesses du cache du navigateur

|ID          |
|------------|
|WSTG-ATHN-06|

## Sommaire

Dans cette phase, le testeur vérifie que l'application indique correctement au navigateur de ne pas conserver les données sensibles.

Les navigateurs peuvent stocker des informations à des fins de mise en cache et d'historique. La mise en cache est utilisée pour améliorer les performances, afin que les informations précédemment affichées n'aient pas besoin d'être téléchargées à nouveau. Les mécanismes d'historique sont utilisés pour la commodité de l'utilisateur, afin que l'utilisateur puisse voir exactement ce qu'il a vu au moment où la ressource a été récupérée. Si des informations sensibles sont présentées à l'utilisateur (telles que son adresse, les détails de sa carte de crédit, son numéro de sécurité sociale ou son nom d'utilisateur), ces informations peuvent être stockées à des fins de mise en cache ou d'historique, et donc récupérables en examinant le cache du navigateur ou simplement en en appuyant sur le bouton **Précédent** du navigateur.

## Objectifs des tests

- Vérifiez si l'application stocke des informations sensibles côté client.
- Vérifiez si l'accès peut se produire sans autorisation.

## Comment tester

### Historique du navigateur

Techniquement, le bouton **Retour** est un historique et non un cache (voir [Mise en cache dans HTTP : Listes d'historique](https://www.w3.org/Protocols/rfc2616/rfc2616-sec13.html#sec13.13 )). Le cache et l'historique sont deux entités différentes. Cependant, ils partagent la même faiblesse de présenter des informations sensibles précédemment affichées.

Le premier test, le plus simple, consiste à entrer des informations sensibles dans l'application et à se déconnecter. Ensuite, le testeur clique sur le bouton **Retour** du navigateur pour vérifier si les informations sensibles précédemment affichées sont accessibles sans authentification.

Si, en appuyant sur le bouton **Retour**, le testeur peut accéder aux pages précédentes mais pas aux nouvelles, il ne s'agit pas d'un problème d'authentification, mais d'un problème d'historique du navigateur. Si ces pages contiennent des données sensibles, cela signifie que l'application n'a pas interdit au navigateur de les stocker.

L'authentification n'a pas nécessairement besoin d'être impliquée dans les tests. Par exemple, lorsqu'un utilisateur saisit son adresse e-mail pour s'inscrire à une newsletter, cette information peut être récupérable si elle n'est pas correctement gérée.

Le bouton **Retour** peut être empêché d'afficher des données sensibles. Cela peut être fait par :

- Delivering the page over HTTPS.
- Setting `Cache-Control: must-revalidate`

### Cache du navigateur

Ici, les testeurs vérifient que l'application ne divulgue aucune donnée sensible dans le cache du navigateur. Pour ce faire, ils peuvent utiliser un proxy (tel que OWASP ZAP) et rechercher dans les réponses du serveur appartenant à la session, en vérifiant que pour chaque page contenant des informations sensibles, le serveur a demandé au navigateur de ne mettre aucune donnée en cache. Une telle directive peut être émise dans les en-têtes de réponse HTTP avec les directives suivantes :

- `Cache-Control: no-cache, no-store`
- `Expires: 0`
- `Pragma: no-cache`

Ces directives sont généralement robustes, bien que des drapeaux supplémentaires puissent être nécessaires pour l'en-tête `Cache-Control` afin de mieux empêcher les fichiers liés de manière persistante sur le système de fichiers. Ceux-ci inclus :

- `Cache-Control: must-revalidate, max-age=0, s-maxage=0`

```http
HTTP/1.1:
Cache-Control: no-cache
```

```html
HTTP/1.0:
Pragma: no-cache
Expires: "past date or illegal value (e.g., 0)"
```

Par exemple, si les testeurs testent une application de commerce électronique, ils doivent rechercher toutes les pages contenant un numéro de carte de crédit ou d'autres informations financières, et vérifier que toutes ces pages appliquent la directive "no-cache". S'ils trouvent des pages qui contiennent des informations critiques mais qui ne demandent pas au navigateur de ne pas mettre en cache leur contenu, ils savent que des informations sensibles seront stockées sur le disque, et ils peuvent vérifier cela simplement en recherchant la page dans le cache du navigateur.

L'emplacement exact où ces informations sont stockées dépend du système d'exploitation client et du navigateur utilisé. Voici quelques exemples :

- Mozilla Firefox:
    - Unix/Linux: `~/.cache/mozilla/firefox/`
    - Windows: `C:\Users\<user_name>\AppData\Local\Mozilla\Firefox\Profiles\<profile-id>\Cache2\`
- Internet Explorer:
    - `C:\Users\<user_name>\AppData\Local\Microsoft\Windows\INetCache\`
- Chrome:
    - Windows: `C:\Users\<user_name>\AppData\Local\Google\Chrome\User Data\Default\Cache`
    - Unix/Linux: `~/.cache/google-chrome`

#### Examen des informations mises en cache

Firefox fournit une fonctionnalité pour afficher les informations mises en cache, ce qui peut être à votre avantage en tant que testeur. Bien sûr, l'industrie a également produit diverses extensions et applications externes que vous préférez ou dont vous avez peut-être besoin pour Chrome, Internet Explorer ou Edge.

Les détails du cache sont également disponibles via des outils de développement dans la plupart des navigateurs modernes, tels que [Firefox](https://developer.mozilla.org/en-US/docs/Tools/Storage_Inspector#Cache_Storage), [Chrome](https://developer.google.com/web/tools/chrome-devtools/storage/cache) et Edge. Avec Firefox, il est également possible d'utiliser l'URL `about:cache` pour vérifier les détails du cache.

#### Traitement des vérifications pour les navigateurs mobiles

La gestion des directives de cache peut être complètement différente pour les navigateurs mobiles. Par conséquent, les testeurs doivent démarrer une nouvelle session de navigation avec des caches propres et profiter de fonctionnalités telles que le [mode appareil](https://developers.google.com/web/tools/chrome-devtools/device-mode) de Chrome ou le [mode réactif](https://developer.mozilla.org/en-US/docs/Tools/Responsive_Design_Mode) de Firefox. pour tester à nouveau ou tester séparément les concepts décrits ci-dessus.

De plus, les proxys personnels tels que ZAP et Burp Suite permettent au testeur de spécifier quel `User-Agent` doit être envoyé par ses spiders/crawlers. Cela peut être défini pour correspondre à une chaîne `User-Agent` de navigateur mobile et utilisé pour voir quelles directives de mise en cache sont envoyées par l'application testée.

### Test de la boîte grise

La méthodologie de test est équivalente au cas de la boîte noire, car dans les deux scénarios, les testeurs ont un accès complet aux en-têtes de réponse du serveur et au code HTML. Cependant, avec les tests en boîte grise, le testeur peut avoir accès aux informations d'identification du compte qui lui permettront de tester des pages sensibles accessibles uniquement aux utilisateurs authentifiés.

## Outils

- [Proxy d'attaque Zed OWASP](https://www.zaproxy.org)

## Références

### Papiers blanc

- [Mise en cache en HTTP](https://www.w3.org/Protocols/rfc2616/rfc2616-sec13.html)
