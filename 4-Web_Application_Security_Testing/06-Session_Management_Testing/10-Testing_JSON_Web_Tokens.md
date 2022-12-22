# Test des jetons Web JSON

|ID          |
|------------|
|WSTG-SESS-10|

## Sommaire

Les jetons Web JSON (JWT) sont des jetons JSON signés cryptographiquement, destinés à partager les revendications entre les systèmes. Ils sont fréquemment utilisés comme jetons d'authentification ou de session, en particulier sur les API REST.

Les JWT sont une source courante de vulnérabilités, à la fois dans la manière dont ils sont implémentés dans les applications et dans les bibliothèques sous-jacentes. Comme ils sont utilisés pour l'authentification, une vulnérabilité peut facilement entraîner une compromission complète de l'application.

## Objectifs des tests

- Déterminer si les JWT exposent des informations sensibles.
- Déterminer si les JWT peuvent être falsifiés ou modifiés.

## Comment tester

### Aperçu

Les JWT sont composés de trois composants :

- L'en-tête
- La charge utile (ou carrosserie)
- La signature

Chaque composant est encodé en Base64, et ils sont séparés par des points (`.`). Notez que l'encodage Base64 utilisé dans un JWT supprime les signes égal (`=`), vous devrez donc peut-être les rajouter pour décoder les sections.

### Analyser le contenu

#### Entête

L'en-tête définit le type de jeton (généralement "JWT") et l'algorithme utilisé pour la signature. Un exemple d'en-tête décodé est illustré ci-dessous :

```json
{
  "alg": "HS256",
  "typ": "JWT"
}
```

Il existe trois principaux types d'algorithmes utilisés pour calculer les signatures :

| Algorithme | Descriptif |
|-----------|---------------------|
| HSxxx | HMAC utilisant une clé secrète et SHA-xxx. |
| RSxxx et PSxxx | Signature de clé publique utilisant RSA. |
| ESxxx | Signature à clé publique avec ECDSA. |

Il existe également un large éventail d'[autres algorithmes](https://www.iana.org/assignments/jose/jose.xhtml#web-signature-encryption-algorithms) qui peuvent être utilisés pour les jetons chiffrés (JWE), bien que ceux-ci sont moins courants.

#### Charge utile

La charge utile du JWT contient les données réelles. Un exemple de charge utile est illustré ci-dessous :

```json
{
  "username": "admininistrator",
  "is_admin": true,
  "iat": 1516239022,
  "exp": 1516242622
}
```

La charge utile n'est généralement pas cryptée, alors examinez-la pour déterminer si des données sensibles ou potentiellement inappropriées y sont incluses.

Ce JWT inclut le nom d'utilisateur et le statut administratif de l'utilisateur, ainsi que deux revendications standard (`iat` et `exp`). Ces revendications sont définies dans la [RFC 5719](https://tools.ietf.org/html/rfc7519#section-4.1), un bref résumé en est donné dans le tableau ci-dessous :

| Réclamation | Nom complet | Descriptif |
|-------|-----------|-------------|
| `iss` | Émetteur | L'identité de la partie qui a émis le jeton. |
| `iat` | Émis à | L'horodatage Unix du moment où le jeton a été émis. |
| `nbf` | Pas avant | L'horodatage Unix de la date la plus ancienne à laquelle le jeton peut être utilisé. |
| `exp` | Expire | L'horodatage Unix de l'expiration du jeton. |

#### Signature

La signature est calculée à l'aide de l'algorithme défini dans l'en-tête JWT, puis encodée en Base64 et ajoutée au jeton. La modification d'une partie du JWT devrait entraîner l'invalidité de la signature et le rejet du jeton par le serveur.

### Examiner l'utilisation

En plus d'être lui-même cryptographiquement sécurisé, le JWT doit également être stocké et envoyé de manière sécurisée. Cela devrait inclure des vérifications qui :

- Il est toujours [envoyé via des connexions chiffrées (HTTPS)](../09-Testing_for_Weak_Cryptography/03-Testing_for_Sensitive_Information_Sent_via_Unencrypted_Channels.md).
- S'il est stocké dans un cookie, il doit être [marqué avec les attributs appropriés](../06-Session_Management_Testing/02-Testing_for_Cookies_Attributes.md).

La validité du JWT doit également être examinée, sur la base des revendications `iat`, `nbf` et `exp`, pour déterminer que :

- Le JWT a une durée de vie raisonnable pour l'application.
- Les jetons expirés sont rejetés par l'application.

### Vérification de la signature

L'une des vulnérabilités les plus graves rencontrées avec les JWT est lorsque l'application ne parvient pas à valider que la signature est correcte. Cela se produit généralement lorsqu'un développeur utilise une fonction telle que la fonction NodeJS `jwt.decode()`, qui décode simplement le corps du JWT, plutôt que `jwt.verify()`, qui vérifie la signature avant de décoder le JWT.

Cela peut être facilement testé en modifiant le corps du JWT sans rien changer dans l'en-tête ou la signature, en le soumettant dans une requête pour voir si l'application l'accepte.

#### L'algorithme Aucun

Outre la clé publique et les algorithmes basés sur HMAC, la spécification JWT définit également un algorithme de signature appelé `none`. Comme son nom l'indique, cela signifie qu'il n'y a pas de signature pour le JWT, ce qui lui permet d'être modifié.

Cela peut être testé en modifiant l'algorithme de signature (`alg`) dans l'en-tête JWT en `none`, comme indiqué dans l'exemple ci-dessous :

```json
{
        "alg": "none",
        "typ": "JWT"
}
```

L'en-tête et la charge utile sont ensuite réencodés avec Base64, et la signature est supprimée (en laissant le point final). En utilisant l'en-tête ci-dessus et la charge utile répertoriée dans la section [Charge utile](#charge-utile), cela donnerait le JWT suivant :

```txt
eyJhbGciOiAibm9uZSIsICJ0eXAiOiAiSldUIn0K.eyJ1c2VybmFtZSI6ImFkbWluaW5pc3RyYXRvciIsImlzX2FkbWluIjp0cnVlLCJpYXQiOjE1MTYyMzkwMjIsImV4cCI6MTUxNjI0MjYyMn0.
```

Certaines implémentations tentent d'éviter cela en bloquant explicitement l'utilisation de l'algorithme "none". Si cela est fait d'une manière insensible à la casse, il peut être possible de contourner en spécifiant un algorithme tel que `NoNe`.

#### ECDSA "Signatures psychiques"

Une vulnérabilité a été identifiée dans Java version 15 à 18 où ils ne validaient pas correctement les signatures ECDSA dans certaines circonstances ([CVE-2022-21449](https://neilmadden.blog/2022/04/19/psychic-signatures-in-java/), appelées "signatures psychiques"). Si l'une de ces versions vulnérables est utilisée pour analyser un JWT à l'aide de l'algorithme "ES256", cela peut être utilisé pour contourner complètement la vérification de la signature en falsifiant le corps, puis en remplaçant la signature par la valeur suivante :

```txt
MAYCAQACAQA
```

Résultant en un JWT qui ressemble à ceci :

```txt
eyJhbGciOiJFUzI1NiIsInR5cCI6IkpXVCJ9.eyJhZG1pbiI6InRydWUifQ.MAYCAQACAQA
```

### Clés HMAC faibles

Si le JWT est signé à l'aide d'un algorithme basé sur HMAC (tel que HS256), la sécurité de la signature dépend entièrement de la force de la clé secrète utilisée dans le HMAC.

Si l'application utilise un logiciel standard ou open source, la première étape doit être d'étudier le code et de voir s'il existe une clé de signature HMAC par défaut utilisée.

S'il n'y a pas de valeur par défaut, il peut être possible de deviner ou de forcer brutalement leur clé. Pour ce faire, le moyen le plus simple consiste à utiliser le script [crackjwt.py](https://github.com/Sjord/jwtcrack), qui nécessite simplement le JWT et un fichier de dictionnaire.

Une option plus puissante consiste à convertir le JWT dans un format pouvant être utilisé par [John the Ripper](https://github.com/openwall/john) en utilisant [jwt2john.py](https://github.com /Sjord/jwtcrack/blob/master/jwt2john.py). John peut alors être utilisé pour mener des attaques beaucoup plus poussées contre la clé.

Si le JWT est volumineux, il peut dépasser la taille maximale prise en charge par John. Cela peut être contourné en augmentant la valeur de la variable `SALT_LIMBS` dans `/src/hmacSHA256_fmt_plug.c` (ou le fichier équivalent pour d'autres formats HMAC) et en recompilant John, comme indiqué dans le [problème GitHub](https://github.com/openwall/john/issues/1904).

Si cette clé peut être obtenue, il est alors possible de créer et de signer des JWT arbitraires, ce qui entraîne généralement une compromission complète de l'application.

### Confusion entre HMAC et clé publique

Si l'application utilise des JWT avec des signatures basées sur une clé publique, mais ne vérifie pas que l'algorithme est correct, cela peut potentiellement exploiter cela dans une attaque de confusion de type de signature. Pour que cela réussisse, les conditions suivantes doivent être remplies :

1. L'application doit s'attendre à ce que le JWT soit signé avec un algorithme basé sur une clé publique (c'est-à-dire `RSxxx` ou `ESxxx`).
2. L'application ne doit pas vérifier quel algorithme le JWT utilise réellement pour la signature.
3. La clé publique utilisée pour vérifier le JWT doit être disponible pour l'attaquant.

Si toutes ces conditions sont remplies, un attaquant peut utiliser la clé publique pour signer le JWT à l'aide d'un algorithme basé sur HMAC (tel que "HS256"). Par exemple, la bibliothèque [Node.JS jsonwebtoken](https://www.npmjs.com/package/jsonwebtoken) utilise la même fonction pour les clés publiques et les jetons basés sur HMAC, comme illustré dans l'exemple ci-dessous :

```javascript
// Verify a JWT signed using RS256
jwt.verify(token, publicKey);

// Verify a JWT signed using HS256
jwt.verify(token, secretKey);
```

Cela signifie que si le JWT est signé en utilisant `publicKey` comme clé secrète pour l'algorithme `HS256`, la signature sera considérée comme valide.

Afin d'exploiter ce problème, la clé publique doit être obtenue. Cela peut se produire le plus souvent si l'application réutilise la même clé pour la signature des jetons JWT et dans le cadre du certificat TLS. Dans ce cas, la clé peut être téléchargée depuis le serveur à l'aide d'une commande telle que la suivante :

```sh
openssl s_client -connect exemple.org:443 | openssl x509 -pubkey -noout
```

Alternativement, la clé peut être disponible à partir d'un fichier public sur le site à un emplacement commun tel que `/.well-known/jwks.json`.

Afin de tester cela, modifiez le contenu du JWT, puis utilisez la clé publique précédemment obtenue pour signer le JWT à l'aide de l'algorithme "HS256". Ceci est souvent difficile à réaliser lors de tests sans accès au code source ou aux détails d'implémentation, car le format de la clé doit être identique à celui utilisé par le serveur, de sorte que des problèmes tels que l'espace vide ou le codage CRLF peuvent entraîner le non-respect des clés. correspondant à.

### Clé publique fournie par l'attaquant

La [norme JSON Web Signature (JWS)](https://tools.ietf.org/html/rfc7515) (qui définit l'en-tête et les signatures utilisées par les JWT) permet d'intégrer la clé utilisée pour signer le jeton dans l'en-tête . Si la bibliothèque utilisée pour valider le jeton le prend en charge et ne vérifie pas la clé par rapport à une liste de clés approuvées, cela permet à un attaquant de signer un JWT avec une clé arbitraire qu'il fournit.

Il existe une variété de scripts qui peuvent être utilisés pour ce faire, tels que [jwk-node-jose.py](https://github.com/zi0Black/POC-CVE-2018-0114) ou [jwt_tool](https://github.com/ticarpi/jwt_tool).

## Cas de test associés

- [Test des informations sensibles envoyées via des canaux non cryptés](../09-Testing_for_Weak_Cryptography/03-Testing_for_Sensitive_Information_Sent_via_Unencrypted_Channels.md).
- [Test des attributs des cookies](../06-Session_Management_Testing/02-Testing_for_Cookies_Attributes.md).
- [Test du stockage du navigateur](../11-Client-side_Testing/12-Testing_Browser_Storage.md).

## Correction

- Utilisez une bibliothèque sécurisée et à jour pour gérer les JWT.
- Assurez-vous que la signature est valide et qu'elle utilise l'algorithme attendu.
- Utilisez une clé HMAC forte ou une clé privée unique pour les signer.
- Assurez-vous qu'aucune information sensible n'est exposée dans la charge utile.
- Assurez-vous que les JWT sont stockés et transmis en toute sécurité.
- Voir la [fiche de triche des jetons Web JSON OWASP](https://cheatsheetseries.owasp.org/cheatsheets/JSON_Web_Token_for_Java_Cheat_Sheet.html).

## Outils

- [John The Ripper](https://github.com/openwall/john)
- [jwt2john](https://github.com/Sjord/jwtcrack)
- [jwt-cracker](https://github.com/brendan-rius/c-jwt-cracker)
- [Extension JSON Web Tokens Burp](https://portswigger.net/bappstore/f923cbf91698420890354c1d8958fee6)
- [Module complémentaire ZAP JWT](https://github.com/SasanLabs/owasp-zap-jwt-addon)

## Références

- [RFC 7515 JSON Web Signature (JWS)](https://tools.ietf.org/html/rfc7515)
- [Jeton Web JSON RFC 7519 (JWT)](https://tools.ietf.org/html/rfc7519)
- [Feuille de triche du jeton Web JSON OWASP](https://cheatsheetseries.owasp.org/cheatsheets/JSON_Web_Token_for_Java_Cheat_Sheet.html)
