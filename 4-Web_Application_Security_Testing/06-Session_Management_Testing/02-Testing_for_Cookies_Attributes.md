# Test des attributs des cookies

|ID          |
|------------|
|WSTG-SESS-02|

## Sommaire

Les cookies Web (ci-après dénommés cookies) sont souvent un vecteur d'attaque clé pour les utilisateurs malveillants (ciblant généralement d'autres utilisateurs) et l'application doit toujours faire preuve de diligence raisonnable pour protéger les cookies.

HTTP est un protocole sans état, ce qui signifie qu'il ne contient aucune référence aux requêtes envoyées par le même utilisateur. Afin de résoudre ce problème, des sessions ont été créées et ajoutées aux requêtes HTTP. Les navigateurs, comme indiqué dans [tester le stockage du navigateur](../11-Client-side_Testing/12-Testing_Browser_Storage.md), contiennent une multitude de mécanismes de stockage. Dans cette section du guide, chacun est discuté en profondeur.

Le mécanisme de stockage de session le plus utilisé dans les navigateurs est le stockage de cookies. Les cookies peuvent être définis par le serveur, en incluant un en-tête [`Set-Cookie`](https://developer.mozilla.org/en-US/docs/Web/HTTP/Headers/Set-Cookie) dans la réponse HTTP ou via JavaScript. Les cookies peuvent être utilisés pour une multitude de raisons, telles que :

- gestion des sessions
- personnalisation
- suivi

Afin de sécuriser les données des cookies, l'industrie a développé des moyens pour aider à verrouiller ces cookies et limiter leur surface d'attaque. Au fil du temps, les cookies sont devenus un mécanisme de stockage privilégié pour les applications Web, car ils permettent une grande flexibilité d'utilisation et de protection.

Les moyens de protection des cookies sont :

- [Attributs des cookies](https://developer.mozilla.org/en-US/docs/Web/HTTP/Cookies#Creating_cookies)
- [Préfixes des cookies](https://developer.mozilla.org/en-US/docs/Web/HTTP/Cookies#Cookie_prefixes)

## Objectifs des tests

- Assurez-vous que la configuration de sécurité appropriée est définie pour les cookies.

## Comment tester

Ci-dessous, une description de chaque attribut et préfixe sera discutée. Le testeur doit valider qu'ils sont utilisés correctement par l'application. Les cookies peuvent être examinés à l'aide d'un [intercepting proxy](#intercepting-proxy) ou en examinant la boîte à cookies du navigateur.

### Attributs des cookies

#### Attribut sécurisé

L'attribut [`Secure`](https://developer.mozilla.org/en-US/docs/Web/HTTP/Headers/Set-Cookie#Secure) indique au navigateur de n'envoyer le cookie que si la demande est envoyée via un canal sécurisé tel que "HTTPS". Cela aidera à empêcher le cookie d'être transmis dans des requêtes non chiffrées. Si l'application est accessible à la fois via `HTTP` et `HTTPS`, un attaquant pourrait être en mesure de rediriger l'utilisateur pour qu'il envoie son cookie dans le cadre de requêtes non protégées.

#### Attribut HttpOnly

L'attribut [`HttpOnly`](https://developer.mozilla.org/en-US/docs/Web/HTTP/Headers/Set-Cookie#HttpOnly) est utilisé pour aider à prévenir les attaques telles que les fuites de session, car il ne ne pas autoriser l'accès au cookie via un script côté client tel que JavaScript.

> Cela ne limite pas toute la surface d'attaque des attaques XSS, car un attaquant peut toujours envoyer une requête à la place de l'utilisateur, mais limite énormément la portée des vecteurs d'attaque XSS.

#### Attribut de domaine

L'attribut [`Domain`](https://developer.mozilla.org/en-US/docs/Web/HTTP/Cookies#Scope_of_cookies) est utilisé pour comparer le domaine du cookie avec le domaine du serveur pour lequel la requête HTTP est en train de se faire. Si le domaine correspond ou s'il s'agit d'un sous-domaine, l'attribut [`path`](#path-attribute) sera ensuite vérifié.

Notez que seuls les hôtes appartenant au domaine spécifié peuvent définir un cookie pour ce domaine. De plus, l'attribut `domain` ne peut pas être un domaine de premier niveau (tel que `.gov` ou `.com`) pour empêcher les serveurs de définir des cookies arbitraires pour un autre domaine (comme définir un cookie pour `owasp.org`). Si l'attribut de domaine n'est pas défini, le nom d'hôte du serveur qui a généré le cookie est utilisé comme valeur par défaut du "domaine".

Par exemple, si un cookie est défini par une application sur `app.mydomain.com` sans définir d'attribut de domaine, le cookie sera soumis à nouveau pour toutes les demandes ultérieures pour `app.mydomain.com` et ses sous-domaines (tels que ` hacker.app.mydomain.com`), mais pas à `otherapp.mydomain.com`. Si un développeur souhaitait assouplir cette restriction, il pourrait définir l'attribut `domain` sur `mydomain.com`. Dans ce cas, le cookie serait envoyé à toutes les requêtes pour les sous-domaines `app.mydomain.com` et `mydomain.com`, tels que `hacker.app.mydomain.com`, et même `bank.mydomain.com`. S'il y avait un serveur vulnérable sur un sous-domaine (par exemple, `otherapp.mydomain.com`) et que l'attribut `domain` a été défini de manière trop lâche (par exemple, `mydomain.com`), alors le serveur vulnérable pourrait être utilisé pour récolter des cookies (tels que des jetons de session) sur l'ensemble de `mydomain.com`.

#### Attribut de chemin

L'attribut [`Path`](https://developer.mozilla.org/en-US/docs/Web/HTTP/Cookies#Scope_of_cookies) joue un rôle majeur dans la définition de la portée des cookies en conjonction avec le [`domain `](#attribut-domaine). En plus du domaine, le chemin URL pour lequel le cookie est valide peut être spécifié. Si le domaine et le chemin correspondent, le cookie sera envoyé dans la requête. Tout comme avec l'attribut de domaine, si l'attribut de chemin est défini de manière trop lâche, cela pourrait rendre l'application vulnérable aux attaques d'autres applications sur le même serveur. Par exemple, si l'attribut path a été défini sur la racine du serveur Web `/`, les cookies d'application seront envoyés à toutes les applications du même domaine (si plusieurs applications résident sur le même serveur). Quelques exemples pour plusieurs applications sous le même serveur :

- `path=/bank`
- `path=/private`
- `path=/docs`
- `path=/docs/admin`

#### Attribut d'expiration

L'attribut [`Expires`](https://developer.mozilla.org/en-US/docs/Web/HTTP/Cookies#Permanent_cookies) est utilisé pour :

- définir des cookies persistants
- limiter la durée de vie si une session dure trop longtemps
- supprimer un cookie de force en le fixant à une date passée

Contrairement aux [cookies de session](https://developer.mozilla.org/en-US/docs/Web/HTTP/Cookies#Session_cookies), les cookies persistants seront utilisés par le navigateur jusqu'à l'expiration du cookie. Une fois que la date d'expiration a dépassé le temps fixé, le navigateur supprimera le cookie.

#### Attribut SameSite

L'attribut [`SameSite`](https://developer.mozilla.org/en-US/docs/Web/HTTP/Cookies#SameSite_cookies) peut être utilisé pour déterminer si un cookie doit être envoyé avec les requêtes intersites. Cette fonctionnalité permet au serveur d'atténuer le risque de fuite d'informations d'origine croisée. Dans certains cas, il est également utilisé comme stratégie de réduction des risques (ou de mécanisme de défense en profondeur) pour empêcher les attaques de [falsification de demande intersite](05-Testing_for_Cross_Site_Request_Forgery.md). Cet attribut peut être configuré selon trois modes différents :

- `Strict`
- `Lax`
- `None`

##### Valeur stricte

La valeur `Strict` est l'utilisation la plus restrictive de `SameSite`, permettant au navigateur d'envoyer le cookie uniquement au contexte propriétaire sans navigation de niveau supérieur. En d'autres termes, les données associées au cookie ne seront envoyées que sur les requêtes correspondant au site actuel affiché dans la barre d'URL du navigateur. Le cookie ne sera pas envoyé sur les demandes générées par des sites Web tiers. Cette valeur est particulièrement recommandée pour les actions effectuées sur le même domaine. Cependant, il peut avoir certaines limites avec certains systèmes de gestion de session affectant négativement l'expérience de navigation de l'utilisateur. Étant donné que le navigateur n'enverrait pas le cookie sur les demandes générées à partir d'un domaine ou d'un e-mail tiers, l'utilisateur serait tenu de se reconnecter même s'il a déjà une session authentifiée.

##### Valeur laxiste

La valeur `Lax` est moins restrictive que `Strict`. Le cookie sera envoyé si l'URL correspond au domaine du cookie (première partie) même si le lien provient d'un domaine tiers. Cette valeur est considérée par la plupart des navigateurs comme le comportement par défaut car elle offre une meilleure expérience utilisateur que la valeur "Strict". Il ne se déclenche pas pour les actifs, tels que les images, où les cookies peuvent ne pas être nécessaires pour y accéder.

##### Aucune valeur

La valeur `None` spécifie que le navigateur enverra le cookie dans tous les contextes, y compris les requêtes intersites (le comportement normal avant la mise en œuvre de `SameSite`). Si `Samesite=None` est défini, alors l'attribut Secure doit être défini, sinon les navigateurs modernes ignoreront l'attribut SameSite, _e.g._ `SameSite=None; Sécurisé`.

### Préfixes de cookies

De par leur conception, les cookies n'ont pas la capacité de garantir l'intégrité et la confidentialité des informations qui y sont stockées. Ces limitations empêchent un serveur d'avoir confiance dans la façon dont les attributs d'un cookie donné ont été définis lors de la création. Afin de donner aux serveurs de telles fonctionnalités d'une manière rétrocompatible, l'industrie a introduit le concept de [`Cookie Name Prefixes`](https://tools.ietf.org/html/draft-ietf-httpbis-cookie- préfixes-00) pour faciliter la transmission de ces détails intégrés dans le nom du cookie.

#### Préfixe d'hôte

Le préfixe [`__Host-`](https://developer.mozilla.org/en-US/docs/Web/HTTP/Cookies#Cookie_prefixes) attend des cookies qu'ils remplissent les conditions suivantes :

  1. Le cookie doit être défini avec l'attribut [`Secure`](#secure-attribute).
  2. Le cookie doit être défini à partir d'un URI considéré comme sécurisé par l'agent utilisateur.
  3. Envoyé uniquement à l'hôte qui a défini le cookie et NE DOIT PAS inclure d'[attribut `Domain`](#attribut-domaine).
  4. Le cookie doit être défini avec l'attribut [`Path`](#path-attribute) avec une valeur de `/` afin qu'il soit envoyé avec chaque demande à l'hôte.

Pour cette raison, le cookie `Set-Cookie: __Host-SID=12345; Secure; Path=/` serait accepté tandis que l'un des suivants serait toujours rejeté :
`Set-Cookie: __Host-SID=12345`
`Set-Cookie: __Host-SID=12345; Secure`
`Set-Cookie: __Host-SID=12345; Domain=site.exemple`
`Set-Cookie: __Host-SID=12345; Domain=site.exemple; Path=/`
`Set-Cookie: __Host-SID=12345; Secure; Domain=site.exemple; Path=/`

#### Préfixe sécurisé

Le préfixe [`__Secure-`](https://developer.mozilla.org/en-US/docs/Web/HTTP/Cookies#Cookie_prefixes) est moins restrictif et peut être introduit en ajoutant la chaîne sensible à la casse `__Secure- ` au nom du cookie. Tout cookie correspondant au préfixe `__Secure-` devrait remplir les conditions suivantes :

   1. Le cookie doit être défini avec l'attribut "Secure".
   2. Le cookie doit être défini à partir d'un URI considéré comme sécurisé par l'agent utilisateur.

### Pratiques fortes

En fonction des besoins de l'application et du fonctionnement du cookie, les attributs et les préfixes doivent être appliqués. Plus le cookie est verrouillé, mieux c'est.

En rassemblant tout cela, nous pouvons définir la configuration d'attribut de cookie la plus sécurisée comme suit : `Set-Cookie: __Host-SID=<session token>; path=/; Secure; HttpOnly; SameSite=Strict`.

## Outils

### Proxy d'interception

- [Projet OWASP Zed Attack Proxy] (https://www.zaproxy.org)
- [Suite Web Proxy Burp] (https://portswigger.net)

### Plug-in de navigateur

- [Tamper Data for FF Quantum] (https://addons.mozilla.org/en-US/firefox/addon/tamper-data-for-ff-quantum/)
- ["FireSheep" pour FireFox](https://github.com/codebutler/firesheep)
- ["EditThisCookie" pour Chrome](https://chrome.google.com/webstore/detail/editthiscookie/fngmhnnpilhplaeedifhccceomclgfbg?hl=en)
- ["Cookiebro - Gestionnaire de cookies" pour FireFox](https://addons.mozilla.org/en-US/firefox/addon/cookiebro/)

## Références

- [RFC 2965 - Mécanisme de gestion d'état HTTP](https://tools.ietf.org/html/rfc2965)
- [RFC 2616 – Protocole de transfert hypertexte – HTTP 1.1](https://tools.ietf.org/html/rfc2616)
- [Cookies du même site - draft-ietf-httpbis-cookie-same-site-00](https://tools.ietf.org/html/draft-ietf-httpbis-cookie-same-site-00)
- [L'important attribut "expire" de Set-Cookie](https://seckb.yehg.net/2012/02/important-expires-attribute-of-set.html)
- [ID de session HttpOnly dans l'URL et le corps de la page] (https://seckb.yehg.net/2012/06/httponly-session-id-in-url-and-page.html)

