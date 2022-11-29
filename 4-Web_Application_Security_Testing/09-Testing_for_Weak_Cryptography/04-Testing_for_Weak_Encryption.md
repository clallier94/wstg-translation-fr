# Test de cryptage faible

|ID          |
|------------|
|WSTG-CRYP-04|

## Sommaire

Une utilisation incorrecte des algorithmes de chiffrement peut entraîner une exposition de données sensibles, une fuite de clé, une authentification brisée, une session non sécurisée et des attaques par usurpation d'identité. Certains algorithmes de cryptage ou de hachage sont connus pour être faibles et ne sont pas recommandés pour une utilisation tels que MD5 et RC4.

Outre les bons choix d'algorithmes de cryptage sécurisé ou de hachage, les bonnes utilisations des paramètres sont également importantes pour le niveau de sécurité. Par exemple, le mode ECB (Electronic Code Book) n'est pas recommandé pour une utilisation dans le chiffrement asymétrique.

## Objectifs des tests

- Fournir une ligne directrice pour l'identification des utilisations et des implémentations de cryptage ou de hachage faibles.

## Comment tester

### Liste de contrôle de sécurité de base

- Lorsque vous utilisez AES128 ou AES256, l'IV (Initialization Vector) doit être aléatoire et imprévisible. Reportez-vous à [FIPS 140-2, Exigences de sécurité pour les modules cryptographiques](https://csrc.nist.gov/publications/detail/fips/140/2/final), section 4.9.1. tests de générateur de nombres aléatoires. Par exemple, en Java, `java.util.Random` est considéré comme un générateur de nombres aléatoires faible. `java.security.SecureRandom` doit être utilisé à la place de `java.util.Random`.
- Pour le cryptage asymétrique, utilisez la cryptographie à courbe elliptique (ECC) avec une courbe sécurisée telle que `Curve25519` préférée.
    - Si ECC ne peut pas être utilisé, utilisez le cryptage RSA avec une clé minimale de 2048 bits.
- Lors de l'utilisation de RSA dans la signature, le rembourrage PSS est recommandé.
- Les algorithmes de hachage/cryptage faibles ne doivent pas être utilisés tels que MD5, RC4, DES, Blowfish, SHA1. RSA ou DSA 1024 bits, ECDSA 160 bits (courbes elliptiques), 2TDEA 80/112 bits (triple DES à deux clés)
- Exigences minimales de longueur de clé :

```texte
Échange de clé : échange de clé Diffie-Hellman avec minimum 2 048 bits
Intégrité des messages : HMAC-SHA2
Hachage du message : SHA2 256 bits
Chiffrement asymétrique : RSA 2 048 bits
Algorithme à clé symétrique : AES 128 bits
Hachage du mot de passe : PBKDF2, Scrypt, Bcrypt
ECDH, ECDSA : 256 bits
```

- Les utilisations de SSH, le mode CBC ne doivent pas être utilisés.
- Lorsque l'algorithme de cryptage symétrique est utilisé, le mode ECB (Electronic Code Book) ne doit pas être utilisé.
- Lorsque PBKDF2 est utilisé pour hacher le mot de passe, il est recommandé que le paramètre d'itération soit supérieur à 10 000. [NIST](https://pages.nist.gov/800-63-3/sp800-63b.html#sec5) suggère également au moins 10 000 itérations de la fonction de hachage. De plus, il est interdit d'utiliser la fonction de hachage MD5 avec PBKDF2 tel que PBKDF2WithHmacMD5.

### Examen du code source

- Recherchez les mots clés suivants pour identifier l'utilisation d'algorithmes faibles : `MD4, MD5, RC4, RC2, DES, Blowfish, SHA-1, ECB`

- Pour les implémentations Java, l'API suivante est liée au chiffrement. Passez en revue les paramètres de l'implémentation du chiffrement. Par exemple,

```java
SecretKeyFactory(SecretKeyFactorySpi keyFacSpi, Provider provider, String algorithm)
SecretKeySpec(byte[] key, int offset, int len, String algorithm)
Cipher c = Cipher.getInstance("DES/CBC/PKCS5Padding");
```

- Pour le cryptage RSA, les modes de remplissage suivants sont suggérés.

```texte
RSA/ECB/OAEPWithSHA-1AndMGF1Padding (2048)
RSA/ECB/OAEPWithSHA-256AndMGF1Padding (2048)
```

- Recherchez `ECB`, il n'est pas autorisé à l'utiliser dans le rembourrage.
- Vérifiez si un IV différent (vecteur initial) est utilisé.

```java
// Use a different IV value for every encryption
byte[] newIv = ...;
s = new GCMParameterSpec(s.getTLen(), newIv);
cipher.init(..., s);
...
```

- Recherchez `IvParameterSpec`, vérifiez si la valeur IV est générée différemment et de manière aléatoire.

```java
 IvParameterSpec iv = new IvParameterSpec(randBytes);
 SecretKeySpec skey = new SecretKeySpec(key.getBytes(), "AES");
 Cipher cipher = Cipher.getInstance("AES/CBC/PKCS5Padding");
 cipher.init(Cipher.ENCRYPT_MODE, skey, iv);
```

- En Java, recherchez MessageDigest pour vérifier si un algorithme de hachage faible (MD5 ou CRC) est utilisé. Par exemple:

`MessageDigest md5 = MessageDigest.getInstance("MD5");`

- Pour la signature, SHA1 et MD5 ne doivent pas être utilisés. Par exemple:

`Signature sig = Signature.getInstance("SHA1withRSA");`

- Recherchez `PBKDF2`. Pour générer la valeur de hachage du mot de passe, il est suggéré d'utiliser `PBKDF2`. Passez en revue les paramètres pour générer la valeur `PBKDF2`.

Les itérations doivent être supérieures à **10 000** et la valeur **salt** doit être générée en tant que **valeur aléatoire**.

```java
private static byte[] pbkdf2(char[] password, byte[] salt, int iterations, int bytes)
    throws NoSuchAlgorithmException, InvalidKeySpecException
  {
       PBEKeySpec spec = new PBEKeySpec(password, salt, iterations, bytes * 8);
       SecretKeyFactory skf = SecretKeyFactory.getInstance(PBKDF2_ALGORITHM);
       return skf.generateSecret(spec).getEncoded();
   }
```

- Informations sensibles codées en dur :

```texte
Mots clés liés à l'utilisateur : nom, racine, su, sudo, admin, superutilisateur, login, nom d'utilisateur, uid
Mots-clés associés à la clé : clé publique, AK, SK, clé secrète, clé privée, passwd, mot de passe, pwd, clé partagée, clé partagée, cryto, base64
Autres mots clés sensibles courants : sysadmin, root, privilege, pass, key, code, master, admin, uname, session, token, Oauth, privatekey, shared secret
```

## Outils

- Les scanners de vulnérabilité tels que Nessus, NMAP (scripts) ou OpenVAS peuvent rechercher l'utilisation ou l'acceptation d'un cryptage faible par rapport à des protocoles tels que SNMP, TLS, SSH, SMTP, etc.
- Utilisez un outil d'analyse de code statique pour effectuer une révision du code source tel que klocwork, Fortify, Coverity, CheckMark dans les cas suivants.

```texte
CWE-261 : Cryptographie faible pour les mots de passe
CWE-323 : Réutiliser un Nonce, Paire de clés dans Le chiffrement
CWE-326 : Puissance de chiffrement inadaptée
CWE-327 : Utilisation d'un algorithme cryptographique cassé ou risqué 
CWE-328 : Hachage unidirectionnel réversible
CWE-329 : ne pas utiliser un VI aléatoire avec le mode CBC
CWE-330 : Utilisation de valeurs insuffisamment aléatoires
CWE-347 : Vérification incorrecte de la signature cryptographique 
CWE-354 : Validation incorrecte de la valeur de vérification d'intégrité 
CWE-547 : Utilisation de constantes codées en dur, relatives à la sécurité 
CWE-780 : Utilisation de l'algorithme RSA sans OAEP
```

## Références

- [Normes NIST FIPS] (https://csrc.nist.gov/publications/fips)
- [Wikipédia : vecteur d'initialisation] (https://en.wikipedia.org/wiki/Initialization_vector)
- [Codage sécurisé - Génération de nombres aléatoires forts](https://www.securecoding.cert.org/confluence/display/java/MSC02-J.+Generate+strong+random+numbers)
- [Remplissage de chiffrement asymétrique optimal] (https://en.wikipedia.org/wiki/Optimal_asymmetric_encryption_padding)
- [Aide-mémoire sur le stockage cryptographique] (https://cheatsheetseries.owasp.org/cheatsheets/Cryptographic_Storage_Cheat_Sheet.html)
- [Fiche de triche de stockage de mot de passe] (https://cheatsheetseries.owasp.org/cheatsheets/Password_Storage_Cheat_Sheet.html)
- [Codage sécurisé - N'utilisez pas d'algorithmes cryptographiques non sécurisés ou faibles](https://www.securecoding.cert.org/confluence/display/java/MSC61-J.+Do+not+use+insecure+or+weak+ cryptographiques+algorithmes)
- [Aléatoire non sécurisé] (https://owasp.org/www-community/vulnerabilities/Insecure_Randomness)
- [Entropie insuffisante] (https://owasp.org/www-community/vulnerabilities/Insufficient_Entropy)
- [Longueur d'ID de session insuffisante] (https://owasp.org/www-community/vulnerabilities/Insufficient_Session-ID_Length)
- [Utilisation d'un algorithme cryptographique défectueux ou risqué](https://owasp.org/www-community/vulnerabilities/Using_a_broken_or_risky_cryptographic_algorithm)
- [API Javax.crypto.cipher](https://docs.oracle.com/javase/8/docs/api/javax/crypto/Cipher.html)
- ISO 18033-1:2015 – Algorithmes de chiffrement
- ISO 18033-2:2015 – Chiffrements asymétriques
- ISO 18033-3:2015 – Chiffrements par blocs

