# Test des jetons Web JSON

|ID          |
|------------|
|WSTG-SESS-10|

## Sommaire

Les jetons Web JSON (JWT) sont des jetons JSON sign�s cryptographiquement, destin�s � partager les revendications entre les syst�mes. Ils sont fr�quemment utilis�s comme jetons d'authentification ou de session, en particulier sur les API REST.

Les JWT sont une source courante de vuln�rabilit�s, � la fois dans la mani�re dont ils sont impl�ment�s dans les applications et dans les biblioth�ques sous-jacentes. Comme ils sont utilis�s pour l'authentification, une vuln�rabilit� peut facilement entra�ner une compromission compl�te de l'application.

## Objectifs des tests

- D�terminer si les JWT exposent des informations sensibles.
- D�terminer si les JWT peuvent �tre falsifi�s ou modifi�s.

## Comment tester

### Aper�u

Les JWT sont compos�s de trois composants�:

- L'en-t�te
- La charge utile (ou carrosserie)
- La signature

Chaque composant est encod� en Base64, et ils sont s�par�s par des points (`.`). Notez que l'encodage Base64 utilis� dans un JWT supprime les signes �gal (`=`), vous devrez donc peut-�tre les rajouter pour d�coder les sections.

### Analyser le contenu

#### Ent�te

L'en-t�te d�finit le type de jeton (g�n�ralement "JWT") et l'algorithme utilis� pour la signature. Un exemple d'en-t�te d�cod� est illustr� ci-dessous�:

```json
{
  "alg": "HS256",
  "typ": "JWT"
}
```

Il existe trois principaux types d'algorithmes utilis�s pour calculer les signatures�:

| Algorithme | Descriptif |
|-----------|---------------------|
| HSxxx | HMAC utilisant une cl� secr�te et SHA-xxx. |
| RSxxx et PSxxx | Signature de cl� publique utilisant RSA. |
| ESxxx | Signature � cl� publique avec ECDSA. |

Il existe �galement un large �ventail d'[autres algorithmes](https://www.iana.org/assignments/jose/jose.xhtml#web-signature-encryption-algorithms) qui peuvent �tre utilis�s pour les jetons chiffr�s (JWE), bien que ceux-ci sont moins courants.

#### Charge utile

La charge utile du JWT contient les donn�es r�elles. Un exemple de charge utile est illustr� ci-dessous�:

```json
{
  "username": "admininistrator",
  "is_admin": true,
  "iat": 1516239022,
  "exp": 1516242622
}
```

La charge utile n'est g�n�ralement pas crypt�e, alors examinez-la pour d�terminer si des donn�es sensibles ou potentiellement inappropri�es y sont incluses.

Ce JWT inclut le nom d'utilisateur et le statut administratif de l'utilisateur, ainsi que deux revendications standard (`iat` et `exp`). Ces revendications sont d�finies dans la [RFC 5719](https://tools.ietf.org/html/rfc7519#section-4.1), un bref r�sum� en est donn� dans le tableau ci-dessous�:

| R�clamation | Nom complet | Descriptif |
|-------|-----------|-------------|
| `iss` | �metteur | L'identit� de la partie qui a �mis le jeton. |
| `iat` | �mis � | L'horodatage Unix du moment o� le jeton a �t� �mis. |
| `nbf` | Pas avant | L'horodatage Unix de la date la plus ancienne � laquelle le jeton peut �tre utilis�. |
| `exp` | Expire | L'horodatage Unix de l'expiration du jeton. |

#### Signature

La signature est calcul�e � l'aide de l'algorithme d�fini dans l'en-t�te JWT, puis encod�e en Base64 et ajout�e au jeton. La modification d'une partie du JWT devrait entra�ner l'invalidit� de la signature et le rejet du jeton par le serveur.

###�Examiner l'utilisation

En plus d'�tre lui-m�me cryptographiquement s�curis�, le JWT doit �galement �tre stock� et envoy� de mani�re s�curis�e. Cela devrait inclure des v�rifications qui�:

- Il est toujours [envoy� via des connexions chiffr�es (HTTPS)](../09-Testing_for_Weak_Cryptography/03-Testing_for_Sensitive_Information_Sent_via_Unencrypted_Channels.md).
- S'il est stock� dans un cookie, il doit �tre [marqu� avec les attributs appropri�s](../06-Session_Management_Testing/02-Testing_for_Cookies_Attributes.md).

La validit� du JWT doit �galement �tre examin�e, sur la base des revendications "iat", "nbf" et "exp", pour d�terminer que�:

- Le JWT a une dur�e de vie raisonnable pour l'application.
- Les jetons expir�s sont rejet�s par l'application.

### V�rification de la signature

L'une des vuln�rabilit�s les plus graves rencontr�es avec les JWT est lorsque l'application ne parvient pas � valider que la signature est correcte. Cela se produit g�n�ralement lorsqu'un d�veloppeur utilise une fonction telle que la fonction NodeJS `jwt.decode()`, qui d�code simplement le corps du JWT, plut�t que `jwt.verify()`, qui v�rifie la signature avant de d�coder le JWT.

Cela peut �tre facilement test� en modifiant le corps du JWT sans rien changer dans l'en-t�te ou la signature, en le soumettant dans une requ�te pour voir si l'application l'accepte.

#### L'algorithme Aucun

Outre la cl� publique et les algorithmes bas�s sur HMAC, la sp�cification JWT d�finit �galement un algorithme de signature appel� "none". Comme son nom l'indique, cela signifie qu'il n'y a pas de signature pour le JWT, ce qui lui permet d'�tre modifi�.

Cela peut �tre test� en modifiant l'algorithme de signature (`alg`) dans l'en-t�te JWT en `none`, comme indiqu� dans l'exemple ci-dessous�:

```json
{
        "alg": "none",
        "typ": "JWT"
}
```

L'en-t�te et la charge utile sont ensuite r�encod�s avec Base64, et la signature est supprim�e (en laissant le point final). En utilisant l'en-t�te ci-dessus et la charge utile r�pertori�e dans la section [payload](#payload), cela donnerait le JWT suivant�:

```txt
eyJhbGciOiAibm9uZSIsICJ0eXAiOiAiSldUIn0K.eyJ1c2VybmFtZSI6ImFkbWluaW5pc3RyYXRvciIsImlzX2FkbWluIjp0cnVlLCJpYXQiOjE1MTYyMzkwMjIsImV4cCI6MTUxNjI0MjYyMn0.
```

Certaines impl�mentations tentent d'�viter cela en bloquant explicitement l'utilisation de l'algorithme "none". Si cela est fait d'une mani�re insensible � la casse, il peut �tre possible de contourner en sp�cifiant un algorithme tel que "NoNe".

#### ECDSA "Signatures psychiques"

Une vuln�rabilit� a �t� identifi�e dans Java version 15 � 18 o� ils ne validaient pas correctement les signatures ECDSA dans certaines circonstances ([CVE-2022-21449](https://neilmadden.blog/2022/04/19/psychic-signatures-in- java/), appel�es "signatures psychiques"). Si l'une de ces versions vuln�rables est utilis�e pour analyser un JWT � l'aide de l'algorithme "ES256", cela peut �tre utilis� pour contourner compl�tement la v�rification de la signature en falsifiant le corps, puis en rempla�ant la signature par la valeur suivante�:

```txt
MAYCAQACAQA
```

R�sultant en un JWT qui ressemble � ceci�:

```txt
eyJhbGciOiJFUzI1NiIsInR5cCI6IkpXVCJ9.eyJhZG1pbiI6InRydWUifQ.MAYCAQACAQA
```

### Cl�s HMAC faibles

Si le JWT est sign� � l'aide d'un algorithme bas� sur HMAC (tel que HS256), la s�curit� de la signature d�pend enti�rement de la force de la cl� secr�te utilis�e dans le HMAC.

Si l'application utilise un logiciel standard ou open source, la premi�re �tape doit �tre d'�tudier le code et de voir s'il existe une cl� de signature HMAC par d�faut utilis�e.

S'il n'y a pas de valeur par d�faut, il peut �tre possible de deviner ou de forcer brutalement leur cl�. Pour ce faire, le moyen le plus simple consiste � utiliser le script [crackjwt.py](https://github.com/Sjord/jwtcrack), qui n�cessite simplement le JWT et un fichier de dictionnaire.

Une option plus puissante consiste � convertir le JWT dans un format pouvant �tre utilis� par [John the Ripper](https://github.com/openwall/john) en utilisant [jwt2john.py](https://github.com /Sjord/jwtcrack/blob/master/jwt2john.py). John peut alors �tre utilis� pour mener des attaques beaucoup plus pouss�es contre la cl�.

Si le JWT est volumineux, il peut d�passer la taille maximale prise en charge par John. Cela peut �tre contourn� en augmentant la valeur de la variable `SALT_LIMBS` dans `/src/hmacSHA256_fmt_plug.c` (ou le fichier �quivalent pour d'autres formats HMAC) et en recompilant John, comme indiqu� dans le [probl�me GitHub](https�: //github.com/openwall/john/issues/1904).

Si cette cl� peut �tre obtenue, il est alors possible de cr�er et de signer des JWT arbitraires, ce qui entra�ne g�n�ralement une compromission compl�te de l'application.

### Confusion entre HMAC et cl� publique

Si l'application utilise des JWT avec des signatures bas�es sur une cl� publique, mais ne v�rifie pas que l'algorithme est correct, cela peut potentiellement exploiter cela dans une attaque de confusion de type de signature. Pour que cela r�ussisse, les conditions suivantes doivent �tre remplies :

1. L'application doit s'attendre � ce que le JWT soit sign� avec un algorithme bas� sur une cl� publique (c'est-�-dire `RSxxx` ou `ESxxx`).
2. L'application ne doit pas v�rifier quel algorithme le JWT utilise r�ellement pour la signature.
3. La cl� publique utilis�e pour v�rifier le JWT doit �tre disponible pour l'attaquant.

Si toutes ces conditions sont remplies, un attaquant peut utiliser la cl� publique pour signer le JWT � l'aide d'un algorithme bas� sur HMAC (tel que "HS256"). Par exemple, la biblioth�que [Node.JS jsonwebtoken](https://www.npmjs.com/package/jsonwebtoken) utilise la m�me fonction pour les cl�s publiques et les jetons bas�s sur HMAC, comme illustr� dans l'exemple ci-dessous�:

```javascript
// Verify a JWT signed using RS256
jwt.verify(token, publicKey);

// Verify a JWT signed using HS256
jwt.verify(token, secretKey);
```

Cela signifie que si le JWT est sign� en utilisant `publicKey` comme cl� secr�te pour l'algorithme `HS256`, la signature sera consid�r�e comme valide.

Afin d'exploiter ce probl�me, la cl� publique doit �tre obtenue. Cela peut se produire le plus souvent si l'application r�utilise la m�me cl� pour la signature des jetons JWT et dans le cadre du certificat TLS. Dans ce cas, la cl� peut �tre t�l�charg�e depuis le serveur � l'aide d'une commande telle que la suivante�:

```sh
openssl s_client -connect exemple.org:443 | openssl x509 -pubkey -noout
```

Alternativement, la cl� peut �tre disponible � partir d'un fichier public sur le site � un emplacement commun tel que `/.well-known/jwks.json`.

Afin de tester cela, modifiez le contenu du JWT, puis utilisez la cl� publique pr�c�demment obtenue pour signer le JWT � l'aide de l'algorithme "HS256". Ceci est souvent difficile � r�aliser lors de tests sans acc�s au code source ou aux d�tails d'impl�mentation, car le format de la cl� doit �tre identique � celui utilis� par le serveur, de sorte que des probl�mes tels que l'espace vide ou le codage CRLF peuvent entra�ner le non-respect des cl�s. correspondant �.

### Cl� publique fournie par l'attaquant

La [norme JSON Web Signature (JWS)](https://tools.ietf.org/html/rfc7515) (qui d�finit l'en-t�te et les signatures utilis�es par les JWT) permet d'int�grer la cl� utilis�e pour signer le jeton dans l'en-t�te . Si la biblioth�que utilis�e pour valider le jeton le prend en charge et ne v�rifie pas la cl� par rapport � une liste de cl�s approuv�es, cela permet � un attaquant de signer un JWT avec une cl� arbitraire qu'il fournit.

Il existe une vari�t� de scripts qui peuvent �tre utilis�s pour ce faire, tels que [jwk-node-jose.py](https://github.com/zi0Black/POC-CVE-2018-0114) ou [jwt_tool](https ://github.com/ticarpi/jwt_tool).

## Cas de test associ�s

- [Test des informations sensibles envoy�es via des canaux non crypt�s] (../09-Testing_for_Weak_Cryptography/03-Testing_for_Sensitive_Information_Sent_via_Unencrypted_Channels.md).
- [Test des attributs des cookies](../06-Session_Management_Testing/02-Testing_for_Cookies_Attributes.md).
- [Test du stockage du navigateur](../11-Client-side_Testing/12-Testing_Browser_Storage.md).

## Correction

- Utilisez une biblioth�que s�curis�e et � jour pour g�rer les JWT.
- Assurez-vous que la signature est valide et qu'elle utilise l'algorithme attendu.
- Utilisez une cl� HMAC forte ou une cl� priv�e unique pour les signer.
- Assurez-vous qu'aucune information sensible n'est expos�e dans la charge utile.
- Assurez-vous que les JWT sont stock�s et transmis en toute s�curit�.
- Voir la [fiche de triche des jetons Web JSON OWASP] (https://cheatsheetseries.owasp.org/cheatsheets/JSON_Web_Token_for_Java_Cheat_Sheet.html).

## Outils

- [John The Ripper](https://github.com/openwall/john)
- [jwt2john](https://github.com/Sjord/jwtcrack)
- [jwt-cracker](https://github.com/brendan-rius/c-jwt-cracker)
- [Extension JSON Web Tokens Burp] (https://portswigger.net/bappstore/f923cbf91698420890354c1d8958fee6)
- [Module compl�mentaire ZAP JWT] (https://github.com/SasanLabs/owasp-zap-jwt-addon)

## R�f�rences

- [RFC 7515 JSON Web Signature (JWS)](https://tools.ietf.org/html/rfc7515)
- [Jeton Web JSON RFC 7519 (JWT)](https://tools.ietf.org/html/rfc7519)
- [Feuille de triche du jeton Web JSON OWASP] (https://cheatsheetseries.owasp.org/cheatsheets/JSON_Web_Token_for_Java_Cheat_Sheet.html)
