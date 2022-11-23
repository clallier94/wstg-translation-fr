# Test de l'authentification multifacteur (MFA)

|ID          |
|------------|
|WSTG-AUTH-11|

## Sommaire

De nombreuses applications implémentent l'authentification multifacteur (MFA) comme couche de sécurité supplémentaire pour protéger le processus de connexion. Ceci est également connu sous le nom d'authentification à deux facteurs (2FA) ou de vérification en deux étapes (2SV) - bien que ce ne soit pas strictement la même chose. MFA signifie demander à l'utilisateur de fournir *au moins* deux [facteurs d'authentification](#types-of-MFA) différents lors de la connexion.

La MFA ajoute une complexité supplémentaire à la fois à la fonctionnalité d'authentification, ainsi qu'à d'autres domaines liés à la sécurité (tels que la gestion des informations d'identification et la récupération des mots de passe), ce qui signifie qu'il est essentiel qu'elle soit mise en œuvre de manière correcte et robuste.

## Objectifs des tests

- Identifier le type de MFA utilisé par l'application.
- Déterminer si la mise en œuvre MFA est robuste et sécurisée.
- Essayez de contourner le MFA.

## Comment tester

### Types d'AMF

MFA signifie qu'*au moins* deux des facteurs suivants sont requis pour l'authentification :

| Facteur | Exemples |
|--------|----------|
| Quelque chose que vous savez | Mots de passe, codes PIN et questions de sécurité. |
| Quelque chose que vous avez | Jetons matériels ou logiciels, certificats, e-mails*, SMS et appels téléphoniques. |
| Quelque chose que vous êtes | Empreintes digitales, reconnaissance faciale, scans d'iris, scans d'empreintes digitales et facteurs comportementaux. |
| Localisation | Plages d'adresses IP sources et géolocalisation. |

\* Le courrier électronique ne constitue vraiment "quelque chose que vous avez" que si le compte de messagerie lui-même est protégé par MFA. En tant que tel, il doit être considéré comme plus faible que d'autres alternatives telles que les certificats ou le TOTP, et peut ne pas être accepté comme MFA selon certaines définitions.

Notez que le fait d'exiger plusieurs exemples d'un seul facteur (par exemple, avoir besoin à la fois d'un mot de passe et d'un code PIN) ** ne constitue pas une authentification multifacteur **, bien qu'il puisse offrir certains avantages en matière de sécurité par rapport à un simple mot de passe, et peut être considéré comme une vérification en deux étapes ( 2SV).

En raison de la complexité de la mise en œuvre de la biométrie dans un environnement basé sur un navigateur, "Something You Are" est rarement utilisé pour les applications Web, bien qu'il commence à être adopté à l'aide de normes telles que WebAuthn. Le deuxième facteur le plus courant est "Quelque chose que vous avez".

### Vérifier les contournements MFA

La première étape du test MFA consiste à identifier toutes les fonctionnalités d'authentification dans l'application, qui peuvent inclure :

- La page de connexion principale.
- Fonctionnalité critique pour la sécurité (telle que la désactivation de MFA ou la modification d'un mot de passe).
- Fournisseurs de connexion fédérés.
- Points de terminaison API (à partir de l'interface Web principale et des applications mobiles).
- Protocoles alternatifs (non HTTP).
- Fonctionnalité de test ou de débogage.

Toutes les différentes méthodes de connexion doivent être examinées afin de garantir que l'authentification MFA est appliquée de manière cohérente. Si certaines méthodes ne nécessitent pas de MFA, elles peuvent fournir une méthode simple pour les contourner.

Si l'authentification est effectuée en plusieurs étapes, il peut être possible de la contourner en complétant la première étape du processus d'authentification (en saisissant le nom d'utilisateur et le mot de passe), puis en forçant la navigation vers l'application ou en effectuant des requêtes API directes sans terminer la seconde. étape (saisie du code MFA).

Dans certains cas, des contournements MFA intentionnels peuvent également être mis en œuvre, comme ne pas nécessiter de MFA :

- À partir d'adresses IP spécifiques (qui peuvent être usurpées à l'aide de l'en-tête HTTP `X-Forwarded-For`).
- Lorsqu'un en-tête HTTP spécifique est défini (comme un en-tête non standard comme `X-Debug`).
- Pour un compte spécifique codé en dur (comme un compte "root" ou "breakglass").

Lorsqu'une application prend en charge à la fois les connexions locales et fédérées, il peut être possible de contourner le MFA s'il n'y a pas de séparation forte entre ces deux types de comptes. Par exemple, si un utilisateur enregistre un compte local et configure MFA pour celui-ci, mais n'a pas configuré MFA sur son compte sur le fournisseur de connexion fédérée, il peut être possible pour un attaquant de réenregistrer (ou de lier) un compte fédéré sur l'application cible avec la même adresse e-mail en compromettant le compte de l'utilisateur sur le fournisseur de connexion fédérée.

Enfin, si le MFA est implémenté sur un système différent de l'application principale (comme sur un proxy inverse, afin de protéger une application héritée qui ne supporte pas nativement le MFA), il peut alors être possible de le contourner en se connectant directement à le serveur d'applications principal, comme indiqué dans le guide sur la façon de [mapper l'architecture de l'application](../01-Information_Gathering/10-Map_Application_Architecture.md#content-delivery-network-cdn).

### Vérifier la gestion MFA

La fonctionnalité utilisée pour gérer MFA depuis le compte de l'utilisateur doit être testée pour détecter les vulnérabilités, notamment :

- L'utilisateur doit-il se ré-authentifier pour supprimer ou modifier les paramètres MFA ?
- La fonctionnalité de gestion MFA est-elle vulnérable à [falsification de demande intersite](../06-Session_Management_Testing/05-Testing_for_Cross_Site_Request_Forgery.md) ?
- Le paramètre MFA d'autres utilisateurs peut-il être modifié via [Vulnérabilités IDOR](../05-Authorization_Testing/04-Testing_for_Insecure_Direct_Object_References.md) ?

### Vérifier les options de récupération MFA

De nombreuses applications fourniront aux utilisateurs un moyen de retrouver l'accès à leur compte s'ils ne peuvent pas s'authentifier avec leur deuxième facteur (par exemple s'ils ont perdu leur téléphone). Ces mécanismes peuvent souvent représenter une faiblesse importante de l'application, car ils permettent effectivement de contourner le deuxième facteur d'authentification.

#### Codes de récupération

Certaines applications fourniront à l'utilisateur une liste de codes de récupération ou de sauvegarde lorsqu'elles activent MFA, qui peut être utilisée pour se connecter. Ceux-ci doivent être vérifiés pour s'assurer :

- Ils sont suffisamment longs et complexes pour se protéger des attaques par force brute.
- Ils sont générés de manière sécurisée.
- Ils ne peuvent être utilisés qu'une seule fois.
- Une protection contre la force brute est en place (comme le verrouillage du compte).
- L'utilisateur est averti (par e-mail, SMS, etc.) lorsqu'un code est utilisé.

Consultez la section ["Codes de sauvegarde" dans l'aide-mémoire sur les mots de passe oubliés](https://cheatsheetseries.owasp.org/cheatsheets/Forgot_Password_Cheat_Sheet.html#backup-codes) pour plus de détails.

#### Processus de réinitialisation MFA

Si l'application implémente un processus de réinitialisation MFA, celui-ci doit être testé de la même manière que le [processus de réinitialisation du mot de passe] (09-Testing_for_Weak_Password_Change_or_Reset_Functionalities.md) est testé. Il est important que ce processus soit *au moins* aussi solide que l'implémentation MFA pour l'application.

#### Authentification alternative

Certaines applications permettront à l'utilisateur de prouver son identité par d'autres moyens, comme l'utilisation de [questions de sécurité](08-Testing_for_Weak_Security_Question_Answer.md). Cela représente généralement une faiblesse importante, car les questions de sécurité offrent un niveau de sécurité bien inférieur à celui de MFA.

### Mots de passe à usage unique

La forme la plus courante de MFA est celle des mots de passe à usage unique (OTP), qui sont généralement des codes numériques à six chiffres (bien qu'ils puissent être plus longs ou plus courts). Ceux-ci peuvent soit être générés à la fois par le serveur et l'utilisateur (par exemple, avec une application d'authentification), soit être générés sur le serveur et envoyés à l'utilisateur. Cet OTP peut être fourni à l'utilisateur de différentes manières, notamment :

| Taper | Descriptif |
|------|-------------|
| Mot de passe à usage unique HMAC (HOPT) | Génère un code basé sur le HMAC d'un secret et d'un compteur partagé. |
| Mot de passe à usage unique basé sur le temps (TOTP) | Génère un code basé sur le HMAC d'un secret et l'heure actuelle. |
| E-mail | Envoie un code par e-mail. |
| SMS | Envoie un code par SMS. |
| Téléphone | Envoie un code via un appel vocal à un numéro de téléphone. |

L'OTP est généralement entré après que l'utilisateur a fourni son nom d'utilisateur et son mot de passe. Diverses vérifications doivent être effectuées, notamment :

- Le compte est-il verrouillé après plusieurs tentatives MFA infructueuses ?
- L'adresse IP de l'utilisateur est-elle bloquée après plusieurs tentatives MFA infructueuses sur différents comptes ?
- Les tentatives MFA infructueuses sont-elles enregistrées ?
- Le formulaire est-il vulnérable aux attaques par injection, y compris [l'injection de caractères génériques SQL](../07-Input_Validation_Testing/05-Testing_for_SQL_Injection.md#sql-wildcard-injection) ?

Selon le type d'OTP utilisé, il existe également d'autres contrôles spécifiques à effectuer :

- Comment les OTP sont-ils envoyés à l'utilisateur (e-mail, SMS, téléphone, etc.)
    - Existe-t-il une limitation du débit pour éviter que les spams par SMS/téléphone ne coûtent de l'argent ?
- Quelle est la force des OTP (longueur et keyspace) ?
- Combien de temps les OTP sont-ils valables ?
- Plusieurs OTP sont-ils valides à la fois ?
- Les OTP peuvent-ils être utilisés plus d'une fois ?
- Les OTP sont-ils liés au bon compte utilisateur ou est-il possible de s'authentifier auprès d'eux sur d'autres comptes ?

#### HOTP et TOTP

Les codes HOTP et TOTP sont tous deux basés sur un secret partagé entre le serveur et l'utilisateur. Pour les codes TOTP, cela est généralement fourni à l'utilisateur sous la forme d'un code QR qu'il scanne avec une application d'authentification (bien qu'il puisse également être fourni sous forme de texte secret à saisir manuellement).

Lorsque le secret est généré sur le serveur, il doit être vérifié pour s'assurer qu'il est suffisamment long et complexe ([RFC 4226](https://www.rfc-editor.org/rfc/rfc4226#section-4) recommande à moins 160 bits), et qu'il est généré à l'aide d'une [fonction aléatoire sécurisée](https://cheatsheetseries.owasp.org/cheatsheets/Cryptographic_Storage_Cheat_Sheet.html#secure-random-number-generation).

Lorsque le secret peut être fourni par l'utilisateur, une longueur minimale appropriée doit être appliquée et l'entrée doit être vérifiée pour les attaques par injection habituelles.

Les codes TOTP sont généralement valides pendant 30 secondes, mais certaines applications choisissent d'accepter plusieurs codes (tels que les codes précédents, actuels et suivants) afin de gérer les différences entre l'heure système sur le serveur et sur l'appareil de l'utilisateur. Certaines applications peuvent autoriser plusieurs codes de part et d'autre du code actuel, ce qui peut permettre à un attaquant de deviner ou de forcer le code plus facilement. Le tableau ci-dessous montre les chances de forcer avec succès un code OTP basé sur un attaquant capable de faire 10 requêtes par seconde, pour les applications qui acceptent soit uniquement le code actuel, soit plusieurs codes (voir [cet article](https:/ /www.codasecurity.co.uk/articles/mfa-testing#case-study---brute-forcing-totp) pour les calculs derrière le tableau).

| Codes valides | Taux de réussite après 1 heure | Taux de réussite après 4 heures | Taux de réussite après 12 heures | Taux de réussite après 24 heures |
|------------|---------------------------|------- ---------------------|------------------------------------------ -|------------------------------------------|
| 1 | 4% | 13% | 35% | 58% |
| 3 | 10% | 35% | 72% | 92% |
| 5 | 16% | 51% | 88% | 99% |
| 7 | 22% | 63% | 95% | 99% |

#### Courriel, SMS et téléphone

Lorsque les codes sont générés par le serveur et envoyés au client, les domaines suivants doivent être pris en compte :

- Le mécanisme de transport (email, SMS ou voix) est-il suffisamment sécurisé pour l'application ?
- Les codes sont-ils suffisamment longs et complexes ?
- Les codes sont-ils générés à l'aide d'une [fonction aléatoire sécurisée](https://cheatsheetseries.owasp.org/cheatsheets/Cryptographic_Storage_Cheat_Sheet.html#secure-random-number-generation) ?
- Combien de temps les codes sont-ils valables ?
- Plusieurs codes sont-ils valides à la fois ou est-ce que la génération d'un nouveau code invalide le précédent ?
    - Cela pourrait-il être utilisé pour bloquer l'accès à un compte en demandant à plusieurs reprises des codes ?
- Existe-t-il une limitation de débit suffisante pour empêcher un attaquant de demander un grand nombre de codes ?
    - Un grand nombre de code envoyé par e-mail peut bloquer le serveur pour l'envoi de spam.
    - Un grand nombre de SMS ou d'appels vocaux peut coûter de l'argent ou être utilisé pour harceler un utilisateur.

### Applications mobiles et notifications push

Une approche alternative aux codes OTP consiste à envoyer une notification push au téléphone mobile de l'utilisateur, qu'il peut approuver ou refuser. Cette méthode est moins courante, car elle nécessite que l'utilisateur installe un authentificateur spécifique à l'application.

Pour évaluer correctement la sécurité de cela, il faut étendre la portée des tests pour couvrir à la fois l'application mobile et toutes les API ou services de support utilisés par celle-ci ; ce qui signifie qu'il sortirait souvent du cadre d'un test d'application Web traditionnel. Cependant, il existe quelques vérifications simples qui peuvent être effectuées sans tester l'application mobile, notamment :

- La notification fournit-elle suffisamment de contexte (adresses IP, emplacement, etc.) pour que l'utilisateur puisse décider en connaissance de cause s'il doit l'approuver ou la refuser ?
- Existe-t-il un type de mécanisme de défi et de réponse (comme fournir un code sur le site Web que l'utilisateur doit saisir dans l'application - souvent appelé « correspondance de numéros » ou « défi de numéros ») ?
- Existe-t-il une limitation du débit ou des mécanismes pour empêcher l'utilisateur d'être spammé avec des notifications dans l'espoir qu'il en acceptera une aveuglément ?

### Filtrage des adresses IP et des emplacements

L'un des facteurs parfois utilisés avec MFA est l'emplacement ("quelque part où vous êtes"), bien que la question de savoir si cela constitue un facteur d'authentification approprié est discutable. Dans le contexte d'une application Web, cela signifie généralement restreindre l'accès à des adresses IP spécifiques ou ne pas demander à l'utilisateur un deuxième facteur tant qu'il se connecte à partir d'une adresse IP de confiance spécifique. Un scénario courant pour cela serait d'authentifier les utilisateurs avec uniquement leur mot de passe lorsqu'ils se connectent à partir des plages IP du bureau, mais en exigeant un code OTP lorsqu'ils se connectent d'ailleurs.

Selon l'implémentation, il peut être possible pour un utilisateur d'usurper une adresse IP de confiance en définissant l'en-tête "X-Forwarded-For", ce qui pourrait lui permettre de contourner cette vérification. Notez que si l'application ne nettoie pas correctement le contenu de cet en-tête, il peut également être possible de mener une attaque telle qu'une injection SQL ici. Si l'application prend en charge IPv6, cela doit également être vérifié pour s'assurer que les restrictions appropriées sont appliquées à ces connexions.

De plus, les adresses IP de confiance doivent être examinées pour s'assurer qu'elles ne présentent aucune faiblesse, par exemple si elles incluent :

- Adresses IP qui pourraient être accessibles par des utilisateurs non fiables (tels que les réseaux sans fil invités dans un bureau).
- Adresse IP attribuée dynamiquement qui pourrait changer.
- Plages de réseaux publics où un attaquant pourrait héberger son propre système (comme Azure ou AWS).

### Certificats et cartes à puce

Transport Layer Security (TLS) est couramment utilisé pour crypter le trafic entre le client et le serveur, et pour fournir un mécanisme permettant au client de confirmer l'identité du serveur (en comparant le nom commun (CN) ou le nom alternatif du sujet (SAN) sur le certificat au domaine demandé). Cependant, il peut également fournir un mécanisme permettant au serveur de confirmer l'identité du client, appelé authentification par certificat client ou TLS mutuel (mTLS). Une discussion complète de l'authentification par certificat client sort du cadre de ce guide, mais le principe clé est que l'utilisateur présente un certificat numérique (stocké sur sa machine ou sur une carte à puce), qui est validé par le serveur.

La première étape du test consiste à déterminer si l'application cible restreint les autorités de certification (CA) approuvées pour émettre des certificats. Ces informations peuvent être obtenues à l'aide de divers outils ou en examinant manuellement la poignée de main TLS. Le moyen le plus simple consiste à utiliser le `s_client` d'OpenSSL :

```bash
$ openssl s_client -connect example:443
[...]
Acceptable client certificate CA names
C = US, ST = Example, L = Example, O = Example Org, CN = Example Org Root Certificate Authority
Client Certificate Types: RSA sign, DSA sign, ECDSA sign
```

S'il n'y a pas de restrictions, il peut être possible de s'authentifier à l'aide d'un certificat d'une autre autorité de certification. S'il y a des restrictions mais qu'elles sont mal implémentées, il peut être possible de créer une CA locale avec le nom correct ("Example Org Root Certificate Authority" dans l'exemple ci-dessus), et d'utiliser cette nouvelle CA pour signer les certificats clients.

Si un certificat valide peut être obtenu, il convient également de vérifier que le certificat ne peut être utilisé que pour l'utilisateur pour lequel il est délivré (c'est-à-dire que vous ne pouvez pas utiliser un certificat délivré à Alice pour vous authentifier sur le compte de Bob). De plus, les certificats doivent être vérifiés pour s'assurer qu'ils n'ont ni expiré ni été révoqués.

## Cas de test associés

- [Test du mécanisme de verrouillage faible] (03-Testing_for_Weak_Lock_Out_Mechanism.md)
- [Test des fonctionnalités de changement ou de réinitialisation de mot de passe faibles] (09-Testing_for_Weak_Password_Change_or_Reset_Functionalities.md)

## Correction

Veiller à ce que:

- MFA est mis en œuvre pour tous les comptes et fonctionnalités pertinents sur les applications.
- Les méthodes de support MFA sont adaptées à l'application.
- Les mécanismes utilisés pour mettre en œuvre la MFA sont correctement sécurisés et protégés contre les attaques par force brute.
- Il existe un audit et une journalisation appropriés pour toutes les activités liées à MFA.

Consultez l'[Aide-mémoire sur l'authentification multifacteur OWASP](https://cheatsheetseries.owasp.org/cheatsheets/Multifactor_Authentication_Cheat_Sheet.html) pour plus de recommandations.

## Références

- [Aide-mémoire d'authentification multifacteur OWASP] (https://cheatsheetseries.owasp.org/cheatsheets/Multifactor_Authentication_Cheat_Sheet.html)
