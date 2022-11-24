# Test des faiblesses d'OAuth

|ID          |
|------------|
|WSTG-ATHZ-05|

## Sommaire

[OAuth2.0](https://oauth.net/2/) (ci-après dénommé OAuth) est un cadre d'autorisation qui permet à un client d'accéder à des ressources au nom de son utilisateur.

Pour ce faire, OAuth s'appuie fortement sur les jetons pour communiquer entre les différentes entités, chaque entité ayant un [rôle] différent (https://datatracker.ietf.org/doc/html/rfc6749#section-1.1) :

- **Propriétaire de la ressource :** L'entité qui accorde l'accès à une ressource, le propriétaire et, dans la plupart des cas, l'utilisateur lui-même
- **Client :** L'application qui demande l'accès à une ressource au nom du propriétaire de la ressource. Ces clients sont disponibles en deux [types](https://oauth.net/2/client-types/) :
    - **Public :** clients qui ne peuvent pas protéger un secret (*par exemple* applications axées sur le front-end, telles que les SPA, les applications mobiles, etc.)
    - **Confidentiel :** clients capables de s'authentifier en toute sécurité auprès du serveur d'autorisation en protégeant leurs secrets enregistrés (*par exemple* services back-end)
- **Serveur d'autorisation :** Le serveur qui détient les informations d'autorisation et accorde l'accès
- **Serveur de ressources :** L'application qui sert le contenu auquel accède le client

Étant donné que la responsabilité d'OAuth est de déléguer les droits d'accès du propriétaire au client, il s'agit d'une cible très attrayante pour les attaquants, et de mauvaises implémentations conduisent à un accès non autorisé aux ressources et aux informations des utilisateurs.

Afin de fournir un accès à une application cliente, OAuth s'appuie sur plusieurs [types d'octroi d'autorisation](https://oauth.net/2/grant-types/) pour générer un jeton d'accès :

- [Code d'autorisation](https://oauth.net/2/grant-types/authorization-code/) : utilisé par les clients confidentiels et publics pour échanger un code d'autorisation contre un jeton d'accès, mais recommandé uniquement pour les clients confidentiels
- [Clé de preuve pour l'échange de code (PKCE)](https://oauth.net/2/pkce/) : PKCE s'appuie sur la subvention du code d'autorisation, offrant une sécurité renforcée pour son utilisation par les clients publics et améliorant la posture des confidentiels
- [Client Credentials](https://oauth.net/2/grant-types/client-credentials/) : utilisé pour la communication de machine à machine, où "l'utilisateur" est ici la machine qui demande l'accès à ses propres ressources depuis le Serveur de ressources
- [Device Code](https://oauth.net/2/grant-types/device-code/) : utilisé pour les appareils avec des capacités d'entrée limitées.
- [Refresh Token](https://oauth.net/2/grant-types/refresh-token/) : jetons fournis par le serveur d'autorisation pour permettre aux clients d'actualiser les jetons d'accès des utilisateurs une fois qu'ils deviennent invalides ou expirent. Ce type de subvention est utilisé conjointement avec un autre type de subvention.

Deux flux seront obsolètes dans la version [OAuth2.1](https://oauth.net/2.1/), et leur utilisation n'est pas recommandée :

- [Flux implicite*](https://oauth.net/2/grant-types/implicit/) : l'implémentation sécurisée de PKCE rend ce flux obsolète. Avant PKCE, le flux implicite était utilisé par les applications côté client telles que les [applications à page unique](https://en.wikipedia.org/wiki/Single-page_application) depuis [CORS](https://developer.mozilla .org/en-US/docs/Web/HTTP/CORS) a assoupli la [politique d'origine identique](https://developer.mozilla.org/en-US/docs/Web/Security/Same-origin_policy) pour les sites Web pour communiquer entre eux. Pour plus d'informations sur les raisons pour lesquelles l'octroi implicite n'est pas recommandé, consultez cette [section](https://datatracker.ietf.org/doc/html/draft-ietf-oauth-security-topics#section-2.1.2).
- [Resource Owner Password Credentials](https://oauth.net/2/grant-types/password/) : utilisé pour échanger les identifiants des utilisateurs directement avec le client, qui les envoie ensuite à l'autorisation pour les échanger contre un accès jeton. Pour savoir pourquoi ce flux n'est pas recommandé, consultez cette [section](https://datatracker.ietf.org/doc/html/draft-ietf-oauth-security-topics#section-2.4).

* : Le flux implicite dans OAuth uniquement est obsolète, mais reste une solution viable au sein d'Open ID Connect (OIDC) pour récupérer les "id_tokens". Veillez à comprendre comment le flux implicite est utilisé, qui peut être identifié si seul le point de terminaison `/authorization` est utilisé pour obtenir un jeton d'accès, sans compter sur le point de terminaison `/token` de quelque manière que ce soit. Un exemple à ce sujet peut être trouvé [ici](https://auth0.com/docs/get-started/authentication-and-authorization-flow/implicit-flow-with-form-post).

*Veuillez noter que les flux OAuth sont un sujet complexe et que ce qui précède ne comprend qu'un résumé des domaines clés. Les références en ligne contiennent des informations supplémentaires sur les flux spécifiques.*

## Objectifs des tests

- Déterminez si l'implémentation OAuth2 est vulnérable ou utilise une implémentation obsolète ou personnalisée.

## Comment tester

### Test pour les types de subventions obsolètes

Les types de subventions obsolètes sont devenus obsolètes pour des raisons de sécurité et de fonctionnalité. Identifier s'ils sont utilisés nous permet de vérifier rapidement s'ils sont sensibles à l'une des menaces liées à leur utilisation. Certains peuvent être hors de portée de l'attaquant, comme la façon dont un client peut utiliser les informations d'identification des utilisateurs. Cela doit être documenté et transmis aux équipes d'ingénierie internes.

Pour les clients publics, il est généralement possible d'identifier le type d'octroi dans la demande au point de terminaison `/token`. Il est indiqué dans l'échange de jeton avec le paramètre `grant_type`.

L'exemple suivant montre l'attribution du code d'autorisation avec PKCE.

```http
POST /oauth/token HTTP/1.1
Host: as.example.com
[...]

{
  "client_id":"example-client",
  "code_verifier":"example",
  "grant_type":"authorization_code",
  "code":"example",
  "redirect_uri":"http://client.example.com"
}
```

Les valeurs du paramètre "grant_type" et le type de subvention qu'elles indiquent sont :

- `password` : indique l'octroi du ROPC.
- `client_credentials` : indique l'octroi des informations d'identification du client.
- `authorization_code` : indique l'attribution du code d'autorisation.

Le type de flux implicite n'est pas indiqué par le paramètre `grant_type` puisque le jeton est présenté dans la réponse à la demande de point de terminaison `/authorization` et peut à la place être identifié via le `response_type`. Ci-dessous un exemple.

```http
GET /authorize
  ?client_id=<some_client_id>
  &response_type=token 
  &redirect_uri=https%3A%2F%2Fclient.example.com%2F
  &scope=openid%20profile%20email
  &state=<random_state>
```

Les paramètres d'URL suivants indiquent le flux OAuth utilisé :

- `response_type=token` : indique un flux implicite, car le client demande directement au serveur d'autorisation de renvoyer un jeton.
- `response_type=code` : indique le flux de code d'autorisation, car le client demande au serveur d'autorisation de renvoyer un code, qui sera ensuite échangé avec un jeton.
- `code_challenge=sha256(xyz)` : indique l'extension PKCE, car aucun autre flux n'utilise ce paramètre.

Voici un exemple de demande d'autorisation pour le flux de code d'autorisation avec PKCE :

```http
GET /authorize
    ?redirect_uri=https%3A%2F%2Fclient.example.com%2F
    &client_id=<some_client_id>
    &scope=openid%20profile%20email
    &response_type=code
    &response_mode=query
    &state=<random_state>
    &nonce=<random_nonce>
    &code_challenge=<random_code_challenge>
    &code_challenge_method=S256 HTTP/1.1
Host: as.example.com
[...]
```

#### Clients publics

L'attribution du code d'autorisation avec l'extension PKCE est recommandée pour les clients publics. Une demande d'autorisation pour le flux de code d'autorisation avec PKCE doit contenir `response_type=code` et `code_challenge=sha256(xyz)`.

L'échange de jetons doit contenir le type d'autorisation "authorization_code" et un "code_verifier".

Les types de subventions inappropriés pour les clients publics sont :

- Octroi de code d'autorisation sans l'extension PKCE
- Identifiants clients
- Flux implicite
- ROPC

#### Clients confidentiels

L'attribution du code d'autorisation est recommandée pour les clients confidentiels. L'extension PKCE peut également être utilisée.

Les types de subvention inappropriés pour les clients confidentiels sont :

- Informations d'identification du client (sauf pour le machine-to-machine -- voir ci-dessous)
- Flux implicite
- ROPC

##### Machine à machine

Dans les situations où aucune interaction de l'utilisateur ne se produit et où les clients ne sont que des clients confidentiels, l'octroi des informations d'identification du client peut être utilisé.

Si vous connaissez le `client_id` et `client_secret`, il est possible d'obtenir un jeton en passant le type de subvention `client_credentials`.

```bash
$ curl --request POST \
  --url https://as.example.com/oauth/token \
  --header 'content-type: application/json' \
  --data '{"client_id":"<some_client_id>","client_secret":"<some_client_secret>","grant_type":"client_credentials"}' --proxy http://localhost:8080/ -k
```

### Fuite d'informations d'identification

Selon le flux, OAuth transporte plusieurs types d'informations d'identification sous forme de paramètres d'URL.

Les jetons suivants peuvent être considérés comme des informations d'identification divulguées :

- jeton d'accès
- jeton de rafraîchissement
- Code d'autorisation
- Défi de code PKCE / vérificateur de code

En raison du fonctionnement d'OAuth, le "code" d'autorisation ainsi que le "code_challenge" et le "code_verifier" peuvent faire partie de l'URL. Le flux implicite transporte le jeton d'autorisation dans le cadre de l'URL si le `response_mode` n'est pas défini sur [`form_post`](https://openid.net/specs/oauth-v2-form-post-response-mode-1_0 .html). Cela peut entraîner une fuite du jeton ou du code demandé dans l'en-tête du référent, dans les fichiers journaux et les proxys en raison de la transmission de ces paramètres dans la requête ou le fragment.

Le risque porté par le flux implicite de fuite des jetons est bien plus élevé que celui de la fuite du `code` ou de tout autre paramètre `code_*`, car ils sont liés à des clients spécifiques et sont plus difficiles à abuser en cas de fuite.

Afin de tester ce scénario, utilisez un proxy d'interception HTTP tel que OWASP ZAP et interceptez le trafic OAuth.

- Parcourez le processus d'autorisation et identifiez toutes les informations d'identification présentes dans l'URL.
- Si des ressources externes sont incluses dans une page concernée par le flux OAuth, analysez la demande qui leur est faite. Les informations d'identification pourraient être divulguées dans l'en-tête du référent.

Après avoir parcouru le flux OAuth et utilisé l'application, quelques requêtes sont capturées dans l'historique des requêtes d'un proxy d'interception HTTP. Recherchez l'en-tête du référent HTTP (par exemple, `Referer : https://idp.exemple.com/`) contenant le serveur d'autorisation et l'URL du client dans l'historique des requêtes.

Passer en revue les balises méta HTML (bien que cette balise ne soit [pas prise en charge](https://caniuse.com/mdn-html_elements_meta_name_referrer) sur tous les navigateurs), ou la [Referrer-Policy](https://developer.mozilla.org/ en-US/docs/Web/HTTP/Headers/Referrer-Policy) pourrait aider à évaluer si une fuite d'informations d'identification se produit via l'en-tête du référent.

## Cas de test associés

- [Test des jetons Web JSON](../06-Session_Management_Testing/10-Testing_JSON_Web_Tokens.md)

## Correction

- Lors de la mise en œuvre d'OAuth, considérez toujours la technologie utilisée et si l'application est une application côté serveur qui peut éviter de révéler des secrets, ou une application côté client qui ne le peut pas.
- Dans presque tous les cas, utilisez le flux de code d'autorisation avec PKCE. Une exception peut être les flux de machine à machine.
- Utilisez les paramètres POST ou les valeurs d'en-tête pour transporter les secrets.
- Lorsqu'aucune autre possibilité n'existe (par exemple, dans les applications héritées qui ne peuvent pas être migrées), implémentez des en-têtes de sécurité supplémentaires tels qu'un "Referrer-Policy".

## Outils

- [BurpSuite](https://portswigger.net/burp/releases)
- [EsPReSSO](https://github.com/portswigger/espresso)
- [OWASP ZAP](https://www.zaproxy.org/)

## Références

- [Authentification de l'utilisateur avec OAuth 2.0](https://oauth.net/articles/authentication/)
- [Le ??cadre d'autorisation OAuth 2.0](https://datatracker.ietf.org/doc/html/rfc6749)
- [Cadre d'autorisation OAuth 2.0 : utilisation du jeton porteur](https://datatracker.ietf.org/doc/html/rfc6750)
- [Modèle de menace OAuth 2.0 et considérations de sécurité](https://datatracker.ietf.org/doc/html/rfc6819)
- [Meilleures pratiques actuelles en matière de sécurité OAuth 2.0](https://datatracker.ietf.org/doc/html/draft-ietf-oauth-security-topics-16)
- [Flux de code d'autorisation avec clé de preuve pour l'échange de code] (https://auth0.com/docs/authorization/flows/authorization-code-flow-with-proof-key-for-code-exchange-pkce)
