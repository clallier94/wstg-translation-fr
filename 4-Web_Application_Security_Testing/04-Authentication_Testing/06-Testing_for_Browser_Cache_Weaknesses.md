# Test des faiblesses du cache du navigateur

|ID          |
|------------|
|WSTG-ATHN-06|

## Sommaire

Dans cette phase, le testeur v�rifie que l'application indique correctement au navigateur de ne pas conserver les donn�es sensibles.

Les navigateurs peuvent stocker des informations � des fins de mise en cache et d'historique. La mise en cache est utilis�e pour am�liorer les performances, afin que les informations pr�c�demment affich�es n'aient pas besoin d'�tre t�l�charg�es � nouveau. Les m�canismes d'historique sont utilis�s pour la commodit� de l'utilisateur, afin que l'utilisateur puisse voir exactement ce qu'il a vu au moment o� la ressource a �t� r�cup�r�e. Si des informations sensibles sont pr�sent�es � l'utilisateur (telles que son adresse, les d�tails de sa carte de cr�dit, son num�ro de s�curit� sociale ou son nom d'utilisateur), ces informations peuvent �tre stock�es � des fins de mise en cache ou d'historique, et donc r�cup�rables en examinant le cache du navigateur ou simplement en en appuyant sur le bouton **Pr�c�dent** du navigateur.

## Objectifs des tests

- V�rifiez si l'application stocke des informations sensibles c�t� client.
- V�rifiez si l'acc�s peut se produire sans autorisation.

## Comment tester

### Historique du navigateur

Techniquement, le bouton **Retour** est un historique et non un cache (voir [Mise en cache dans HTTP�: Listes d'historique](https://www.w3.org/Protocols/rfc2616/rfc2616-sec13.html#sec13.13 )). Le cache et l'historique sont deux entit�s diff�rentes. Cependant, ils partagent la m�me faiblesse de pr�senter des informations sensibles pr�c�demment affich�es.

Le premier test, le plus simple, consiste � entrer des informations sensibles dans l'application et � se d�connecter. Ensuite, le testeur clique sur le bouton **Retour** du navigateur pour v�rifier si les informations sensibles pr�c�demment affich�es sont accessibles sans authentification.

Si, en appuyant sur le bouton **Retour**, le testeur peut acc�der aux pages pr�c�dentes mais pas aux nouvelles, il ne s'agit pas d'un probl�me d'authentification, mais d'un probl�me d'historique du navigateur. Si ces pages contiennent des donn�es sensibles, cela signifie que l'application n'a pas interdit au navigateur de les stocker.

L'authentification n'a pas n�cessairement besoin d'�tre impliqu�e dans les tests. Par exemple, lorsqu'un utilisateur saisit son adresse e-mail pour s'inscrire � une newsletter, cette information peut �tre r�cup�rable si elle n'est pas correctement g�r�e.

Le bouton **Retour** peut �tre emp�ch� d'afficher des donn�es sensibles. Cela peut �tre fait par :

- Delivering the page over HTTPS.
- Setting `Cache-Control: must-revalidate`

### Cache du navigateur

Ici, les testeurs v�rifient que l'application ne divulgue aucune donn�e sensible dans le cache du navigateur. Pour ce faire, ils peuvent utiliser un proxy (tel que OWASP ZAP) et rechercher dans les r�ponses du serveur appartenant � la session, en v�rifiant que pour chaque page contenant des informations sensibles, le serveur a demand� au navigateur de ne mettre aucune donn�e en cache. Une telle directive peut �tre �mise dans les en-t�tes de r�ponse HTTP avec les directives suivantes�:

- `Cache-Control: no-cache, no-store`
- `Expires: 0`
- `Pragma: no-cache`

Ces directives sont g�n�ralement robustes, bien que des drapeaux suppl�mentaires puissent �tre n�cessaires pour l'en-t�te `Cache-Control` afin de mieux emp�cher les fichiers li�s de mani�re persistante sur le syst�me de fichiers. Ceux-ci inclus :

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

Par exemple, si les testeurs testent une application de commerce �lectronique, ils doivent rechercher toutes les pages contenant un num�ro de carte de cr�dit ou d'autres informations financi�res, et v�rifier que toutes ces pages appliquent la directive "no-cache". S'ils trouvent des pages qui contiennent des informations critiques mais qui ne demandent pas au navigateur de ne pas mettre en cache leur contenu, ils savent que des informations sensibles seront stock�es sur le disque, et ils peuvent v�rifier cela simplement en recherchant la page dans le cache du navigateur.

L'emplacement exact o� ces informations sont stock�es d�pend du syst�me d'exploitation client et du navigateur utilis�. Voici quelques exemples :

- Mozilla Firefox:
    - Unix/Linux: `~/.cache/mozilla/firefox/`
    - Windows: `C:\Users\<user_name>\AppData\Local\Mozilla\Firefox\Profiles\<profile-id>\Cache2\`
- Internet Explorer:
    - `C:\Users\<user_name>\AppData\Local\Microsoft\Windows\INetCache\`
- Chrome:
    - Windows: `C:\Users\<user_name>\AppData\Local\Google\Chrome\User Data\Default\Cache`
    - Unix/Linux: `~/.cache/google-chrome`

#### Examen des informations mises en cache

Firefox fournit une fonctionnalit� pour afficher les informations mises en cache, ce qui peut �tre � votre avantage en tant que testeur. Bien s�r, l'industrie a �galement produit diverses extensions et applications externes que vous pr�f�rez ou dont vous avez peut-�tre besoin pour Chrome, Internet Explorer ou Edge.

Les d�tails du cache sont �galement disponibles via des outils de d�veloppement dans la plupart des navigateurs modernes, tels que [Firefox](https://developer.mozilla.org/en-US/docs/Tools/Storage_Inspector#Cache_Storage), [Chrome](https:// d�veloppeurs.google.com/web/tools/chrome-devtools/storage/cache) et Edge. Avec Firefox, il est �galement possible d'utiliser l'URL `about:cache` pour v�rifier les d�tails du cache.

#### Traitement des v�rifications pour les navigateurs mobiles

La gestion des directives de cache peut �tre compl�tement diff�rente pour les navigateurs mobiles. Par cons�quent, les testeurs doivent d�marrer une nouvelle session de navigation avec des caches propres et profiter de fonctionnalit�s telles que le [mode appareil] de Chrome(https://developers.google.com/web/tools/chrome-devtools/device-mode) ou le [mode r�actif] de Firefox. Design Mode] (https://developer.mozilla.org/en-US/docs/Tools/Responsive_Design_Mode) pour tester � nouveau ou tester s�par�ment les concepts d�crits ci-dessus.

De plus, les proxys personnels tels que ZAP et Burp Suite permettent au testeur de sp�cifier quel `User-Agent` doit �tre envoy� par ses spiders/crawlers. Cela peut �tre d�fini pour correspondre � une cha�ne `User-Agent` de navigateur mobile et utilis� pour voir quelles directives de mise en cache sont envoy�es par l'application test�e.

###�Test de la bo�te grise

La m�thodologie de test est �quivalente au cas de la bo�te noire, car dans les deux sc�narios, les testeurs ont un acc�s complet aux en-t�tes de r�ponse du serveur et au code HTML. Cependant, avec les tests en bo�te grise, le testeur peut avoir acc�s aux informations d'identification du compte qui lui permettront de tester des pages sensibles accessibles uniquement aux utilisateurs authentifi�s.

## Outils

- [Proxy d'attaque Zed OWASP] (https://www.zaproxy.org)

## R�f�rences

### Papiers blanc

- [Mise en cache en HTTP](https://www.w3.org/Protocols/rfc2616/rfc2616-sec13.html)
