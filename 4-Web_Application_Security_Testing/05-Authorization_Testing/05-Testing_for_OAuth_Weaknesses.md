# Test des faiblesses d'OAuth

|ID          |
|------------|
|WSTG-ATHZ-05|

## Sommaire

[OAuth2.0](https://oauth.net/2/) (ci-apr�s d�nomm� OAuth) est un cadre d'autorisation qui permet � un client d'acc�der � des ressources au nom de son utilisateur.

Pour ce faire, OAuth s'appuie fortement sur les jetons pour communiquer entre les diff�rentes entit�s, chaque entit� ayant un [r�le] diff�rent (https://datatracker.ietf.org/doc/html/rfc6749#section-1.1)�:

- **Propri�taire de la ressource�:** L'entit� qui accorde l'acc�s � une ressource, le propri�taire et, dans la plupart des cas, l'utilisateur lui-m�me
- **Client�:** L'application qui demande l'acc�s � une ressource au nom du propri�taire de la ressource. Ces clients sont disponibles en deux [types](https://oauth.net/2/client-types/)�:
    -�**Public�:**�clients qui ne peuvent pas prot�ger un secret (*par exemple* applications ax�es sur le front-end, telles que les SPA, les applications mobiles, etc.)
    -�**Confidentiel�:**�clients capables de s'authentifier en toute s�curit� aupr�s du serveur d'autorisation en prot�geant leurs secrets enregistr�s (*par exemple* services back-end)
- **Serveur d'autorisation�:** Le serveur qui d�tient les informations d'autorisation et accorde l'acc�s
- **Serveur de ressources�:** L'application qui sert le contenu auquel acc�de le client

�tant donn� que la responsabilit� d'OAuth est de d�l�guer les droits d'acc�s du propri�taire au client, il s'agit d'une cible tr�s attrayante pour les attaquants, et de mauvaises impl�mentations conduisent � un acc�s non autoris� aux ressources et aux informations des utilisateurs.

Afin de fournir un acc�s � une application cliente, OAuth s'appuie sur plusieurs [types d'octroi d'autorisation](https://oauth.net/2/grant-types/) pour g�n�rer un jeton d'acc�s�:

- [Code d'autorisation](https://oauth.net/2/grant-types/authorization-code/)�: utilis� par les clients confidentiels et publics pour �changer un code d'autorisation contre un jeton d'acc�s, mais recommand� uniquement pour les clients confidentiels
- [Cl� de preuve pour l'�change de code (PKCE)](https://oauth.net/2/pkce/)�: PKCE s'appuie sur la subvention du code d'autorisation, offrant une s�curit� renforc�e pour son utilisation par les clients publics et am�liorant la posture des confidentiels
- [Client Credentials](https://oauth.net/2/grant-types/client-credentials/)�: utilis� pour la communication de machine � machine, o� "l'utilisateur" est ici la machine qui demande l'acc�s � ses propres ressources depuis le Serveur de ressources
- [Device Code](https://oauth.net/2/grant-types/device-code/) : utilis� pour les appareils avec des capacit�s d'entr�e limit�es.
- [Refresh Token](https://oauth.net/2/grant-types/refresh-token/)�: jetons fournis par le serveur d'autorisation pour permettre aux clients d'actualiser les jetons d'acc�s des utilisateurs une fois qu'ils deviennent invalides ou expirent. Ce type de subvention est utilis� conjointement avec un autre type de subvention.

Deux flux seront obsol�tes dans la version [OAuth2.1](https://oauth.net/2.1/), et leur utilisation n'est pas recommand�e�:

- [Flux implicite*](https://oauth.net/2/grant-types/implicit/)�: l'impl�mentation s�curis�e de PKCE rend ce flux obsol�te. Avant PKCE, le flux implicite �tait utilis� par les applications c�t� client telles que les [applications � page unique](https://en.wikipedia.org/wiki/Single-page_application) depuis [CORS](https://developer.mozilla .org/en-US/docs/Web/HTTP/CORS) a assoupli la [politique d'origine identique](https://developer.mozilla.org/en-US/docs/Web/Security/Same-origin_policy) pour les sites Web pour communiquer entre eux. Pour plus d'informations sur les raisons pour lesquelles l'octroi implicite n'est pas recommand�, consultez cette [section](https://datatracker.ietf.org/doc/html/draft-ietf-oauth-security-topics#section-2.1.2).
- [Resource Owner Password Credentials](https://oauth.net/2/grant-types/password/)�: utilis� pour �changer les identifiants des utilisateurs directement avec le client, qui les envoie ensuite � l'autorisation pour les �changer contre un acc�s jeton. Pour savoir pourquoi ce flux n'est pas recommand�, consultez cette [section](https://datatracker.ietf.org/doc/html/draft-ietf-oauth-security-topics#section-2.4).

*�: Le flux implicite dans OAuth uniquement est obsol�te, mais reste une solution viable au sein d'Open ID Connect (OIDC) pour r�cup�rer les "id_tokens". Veillez � comprendre comment le flux implicite est utilis�, qui peut �tre identifi� si seul le point de terminaison `/authorization` est utilis� pour obtenir un jeton d'acc�s, sans compter sur le point de terminaison `/token` de quelque mani�re que ce soit. Un exemple � ce sujet peut �tre trouv� [ici](https://auth0.com/docs/get-started/authentication-and-authorization-flow/implicit-flow-with-form-post).

*Veuillez noter que les flux OAuth sont un sujet complexe et que ce qui pr�c�de ne comprend qu'un r�sum� des domaines cl�s. Les r�f�rences en ligne contiennent des informations suppl�mentaires sur les flux sp�cifiques.*

## Objectifs des tests

- D�terminez si l'impl�mentation OAuth2 est vuln�rable ou utilise une impl�mentation obsol�te ou personnalis�e.

## Comment tester

###�Test pour les types de subventions obsol�tes

Les types de subventions obsol�tes sont devenus obsol�tes pour des raisons de s�curit� et de fonctionnalit�. Identifier s'ils sont utilis�s nous permet de v�rifier rapidement s'ils sont sensibles � l'une des menaces li�es � leur utilisation. Certains peuvent �tre hors de port�e de l'attaquant, comme la fa�on dont un client peut utiliser les informations d'identification des utilisateurs. Cela doit �tre document� et transmis aux �quipes d'ing�nierie internes.

Pour les clients publics, il est g�n�ralement possible d'identifier le type d'octroi dans la demande au point de terminaison `/token`. Il est indiqu� dans l'�change de jeton avec le param�tre `grant_type`.

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

Les valeurs du param�tre "grant_type" et le type de subvention qu'elles indiquent sont�:

- `password`�: indique l'octroi du ROPC.
- `client_credentials`�: indique l'octroi des informations d'identification du client.
- `authorization_code`�: indique l'attribution du code d'autorisation.

Le type de flux implicite n'est pas indiqu� par le param�tre `grant_type` puisque le jeton est pr�sent� dans la r�ponse � la demande de point de terminaison `/authorization` et peut � la place �tre identifi� via le `response_type`. Ci-dessous un exemple.

```http
GET /authorize
  ?client_id=<some_client_id>
  &response_type=token 
  &redirect_uri=https%3A%2F%2Fclient.example.com%2F
  &scope=openid%20profile%20email
  &state=<random_state>
```

Les param�tres d'URL suivants indiquent le flux OAuth utilis�:

- `response_type=token`�: indique un flux implicite, car le client demande directement au serveur d'autorisation de renvoyer un jeton.
- `response_type=code`�: indique le flux de code d'autorisation, car le client demande au serveur d'autorisation de renvoyer un code, qui sera ensuite �chang� avec un jeton.
- `code_challenge=sha256(xyz)`�: indique l'extension PKCE, car aucun autre flux n'utilise ce param�tre.

Voici un exemple de demande d'autorisation pour le flux de code d'autorisation avec PKCE�:

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

L'attribution du code d'autorisation avec l'extension PKCE est recommand�e pour les clients publics. Une demande d'autorisation pour le flux de code d'autorisation avec PKCE doit contenir `response_type=code` et `code_challenge=sha256(xyz)`.

L'�change de jetons doit contenir le type d'autorisation "authorization_code" et un "code_verifier".

Les types de subventions inappropri�s pour les clients publics sont�:

- Octroi de code d'autorisation sans l'extension PKCE
- Identifiants clients
- Flux implicite
- ROPC

#### Clients confidentiels

L'attribution du code d'autorisation est recommand�e pour les clients confidentiels. L'extension PKCE peut �galement �tre utilis�e.

Les types de subvention inappropri�s pour les clients confidentiels sont�:

- Informations d'identification du client (sauf pour le machine-to-machine -- voir ci-dessous)
- Flux implicite
- ROPC

##### Machine � machine

Dans les situations o� aucune interaction de l'utilisateur ne se produit et o� les clients ne sont que des clients confidentiels, l'octroi des informations d'identification du client peut �tre utilis�.

Si vous connaissez le `client_id` et `client_secret`, il est possible d'obtenir un jeton en passant le type de subvention `client_credentials`.

```bash
$ curl --request POST \
  --url https://as.example.com/oauth/token \
  --header 'content-type: application/json' \
  --data '{"client_id":"<some_client_id>","client_secret":"<some_client_secret>","grant_type":"client_credentials"}' --proxy http://localhost:8080/ -k
```

### Fuite d'informations d'identification

Selon le flux, OAuth transporte plusieurs types d'informations d'identification sous forme de param�tres d'URL.

Les jetons suivants peuvent �tre consid�r�s comme des informations d'identification divulgu�es�:

- jeton d'acc�s
- jeton de rafra�chissement
- Code d'autorisation
- D�fi de code PKCE / v�rificateur de code

En raison du fonctionnement d'OAuth, le "code" d'autorisation ainsi que le "code_challenge" et le "code_verifier" peuvent faire partie de l'URL. Le flux implicite transporte le jeton d'autorisation dans le cadre de l'URL si le `response_mode` n'est pas d�fini sur [`form_post`](https://openid.net/specs/oauth-v2-form-post-response-mode-1_0 .html). Cela peut entra�ner une fuite du jeton ou du code demand� dans l'en-t�te du r�f�rent, dans les fichiers journaux et les proxys en raison de la transmission de ces param�tres dans la requ�te ou le fragment.

Le risque port� par le flux implicite de fuite des jetons est bien plus �lev� que celui de la fuite du `code` ou de tout autre param�tre `code_*`, car ils sont li�s � des clients sp�cifiques et sont plus difficiles � abuser en cas de fuite.

Afin de tester ce sc�nario, utilisez un proxy d'interception HTTP tel que OWASP ZAP et interceptez le trafic OAuth.

- Parcourez le processus d'autorisation et identifiez toutes les informations d'identification pr�sentes dans l'URL.
- Si des ressources externes sont incluses dans une page concern�e par le flux OAuth, analysez la demande qui leur est faite. Les informations d'identification pourraient �tre divulgu�es dans l'en-t�te du r�f�rent.

Apr�s avoir parcouru le flux OAuth et utilis� l'application, quelques requ�tes sont captur�es dans l'historique des requ�tes d'un proxy d'interception HTTP. Recherchez l'en-t�te du r�f�rent HTTP (par exemple, `Referer�: https://idp.exemple.com/`) contenant le serveur d'autorisation et l'URL du client dans l'historique des requ�tes.

Passer en revue les balises m�ta HTML (bien que cette balise ne soit [pas prise en charge](https://caniuse.com/mdn-html_elements_meta_name_referrer) sur tous les navigateurs), ou la [Referrer-Policy](https://developer.mozilla.org/ en-US/docs/Web/HTTP/Headers/Referrer-Policy) pourrait aider � �valuer si une fuite d'informations d'identification se produit via l'en-t�te du r�f�rent.

## Cas de test associ�s

- [Test des jetons Web JSON](../06-Session_Management_Testing/10-Testing_JSON_Web_Tokens.md)

## Correction

- Lors de la mise en �uvre d'OAuth, consid�rez toujours la technologie utilis�e et si l'application est une application c�t� serveur qui peut �viter de r�v�ler des secrets, ou une application c�t� client qui ne le peut pas.
- Dans presque tous les cas, utilisez le flux de code d'autorisation avec PKCE. Une exception peut �tre les flux de machine � machine.
- Utilisez les param�tres POST ou les valeurs d'en-t�te pour transporter les secrets.
- Lorsqu'aucune autre possibilit� n'existe (par exemple, dans les applications h�rit�es qui ne peuvent pas �tre migr�es), impl�mentez des en-t�tes de s�curit� suppl�mentaires tels qu'un "Referrer-Policy".

## Outils

- [BurpSuite](https://portswigger.net/burp/releases)
- [EsPReSSO](https://github.com/portswigger/espresso)
- [OWASP ZAP](https://www.zaproxy.org/)

## R�f�rences

- [Authentification de l'utilisateur avec OAuth 2.0](https://oauth.net/articles/authentication/)
- [Le ??cadre d'autorisation OAuth 2.0](https://datatracker.ietf.org/doc/html/rfc6749)
- [Cadre d'autorisation OAuth�2.0�: utilisation du jeton porteur](https://datatracker.ietf.org/doc/html/rfc6750)
- [Mod�le de menace OAuth 2.0 et consid�rations de s�curit�](https://datatracker.ietf.org/doc/html/rfc6819)
- [Meilleures pratiques actuelles en mati�re de s�curit� OAuth 2.0](https://datatracker.ietf.org/doc/html/draft-ietf-oauth-security-topics-16)
- [Flux de code d'autorisation avec cl� de preuve pour l'�change de code] (https://auth0.com/docs/authorization/flows/authorization-code-flow-with-proof-key-for-code-exchange-pkce)
