# Test de l'authentification multifacteur (MFA)

|ID          |
|------------|
|WSTG-AUTH-11|

## Sommaire

De nombreuses applications impl�mentent l'authentification multifacteur (MFA) comme couche de s�curit� suppl�mentaire pour prot�ger le processus de connexion. Ceci est �galement connu sous le nom d'authentification � deux facteurs (2FA) ou de v�rification en deux �tapes (2SV) - bien que ce ne soit pas strictement la m�me chose. MFA signifie demander � l'utilisateur de fournir *au moins* deux [facteurs d'authentification](#types-of-MFA) diff�rents lors de la connexion.

La MFA ajoute une complexit� suppl�mentaire � la fois � la fonctionnalit� d'authentification, ainsi qu'� d'autres domaines li�s � la s�curit� (tels que la gestion des informations d'identification et la r�cup�ration des mots de passe), ce qui signifie qu'il est essentiel qu'elle soit mise en �uvre de mani�re correcte et robuste.

## Objectifs des tests

- Identifier le type de MFA utilis� par l'application.
- D�terminer si la mise en �uvre MFA est robuste et s�curis�e.
- Essayez de contourner le MFA.

## Comment tester

### Types d'AMF

MFA signifie qu'*au moins* deux des facteurs suivants sont requis pour l'authentification�:

| Facteur | Exemples |
|--------|----------|
| Quelque chose que vous savez | Mots de passe, codes PIN et questions de s�curit�. |
| Quelque chose que vous avez | Jetons mat�riels ou logiciels, certificats, e-mails*, SMS et appels t�l�phoniques. |
| Quelque chose que vous �tes | Empreintes digitales, reconnaissance faciale, scans d'iris, scans d'empreintes digitales et facteurs comportementaux. |
| Localisation | Plages d'adresses�IP sources et g�olocalisation. |

\* Le courrier �lectronique ne constitue vraiment "quelque chose que vous avez" que si le compte de messagerie lui-m�me est prot�g� par MFA. En tant que tel, il doit �tre consid�r� comme plus faible que d'autres alternatives telles que les certificats ou le TOTP, et peut ne pas �tre accept� comme MFA selon certaines d�finitions.

Notez que le fait d'exiger plusieurs exemples d'un seul facteur (par exemple, avoir besoin � la fois d'un mot de passe et d'un code PIN) ** ne constitue pas une authentification multifacteur **, bien qu'il puisse offrir certains avantages en mati�re de s�curit� par rapport � un simple mot de passe, et peut �tre consid�r� comme une v�rification en deux �tapes ( 2SV).

En raison de la complexit� de la mise en �uvre de la biom�trie dans un environnement bas� sur un navigateur, "Something You Are" est rarement utilis� pour les applications Web, bien qu'il commence � �tre adopt� � l'aide de normes telles que WebAuthn. Le deuxi�me facteur le plus courant est "Quelque chose que vous avez".

### V�rifier les contournements MFA

La premi�re �tape du test MFA consiste � identifier toutes les fonctionnalit�s d'authentification dans l'application, qui peuvent inclure�:

- La page de connexion principale.
- Fonctionnalit� critique pour la s�curit� (telle que la d�sactivation de MFA ou la modification d'un mot de passe).
- Fournisseurs de connexion f�d�r�s.
- Points de terminaison API (� partir de l'interface Web principale et des applications mobiles).
- Protocoles alternatifs (non HTTP).
- Fonctionnalit� de test ou de d�bogage.

Toutes les diff�rentes m�thodes de connexion doivent �tre examin�es afin de garantir que l'authentification MFA est appliqu�e de mani�re coh�rente. Si certaines m�thodes ne n�cessitent pas de MFA, elles peuvent fournir une m�thode simple pour les contourner.

Si l'authentification est effectu�e en plusieurs �tapes, il peut �tre possible de la contourner en compl�tant la premi�re �tape du processus d'authentification (en saisissant le nom d'utilisateur et le mot de passe), puis en for�ant la navigation vers l'application ou en effectuant des requ�tes API directes sans terminer la seconde. �tape (saisie du code MFA).

Dans certains cas, des contournements MFA intentionnels peuvent �galement �tre mis en �uvre, comme ne pas n�cessiter de MFA�:

- � partir d'adresses IP sp�cifiques (qui peuvent �tre usurp�es � l'aide de l'en-t�te HTTP `X-Forwarded-For`).
- Lorsqu'un en-t�te HTTP sp�cifique est d�fini (comme un en-t�te non standard comme `X-Debug`).
- Pour un compte sp�cifique cod� en dur (comme un compte "root" ou "breakglass").

Lorsqu'une application prend en charge � la fois les connexions locales et f�d�r�es, il peut �tre possible de contourner le MFA s'il n'y a pas de s�paration forte entre ces deux types de comptes. Par exemple, si un utilisateur enregistre un compte local et configure MFA pour celui-ci, mais n'a pas configur� MFA sur son compte sur le fournisseur de connexion f�d�r�e, il peut �tre possible pour un attaquant de r�enregistrer (ou de lier) un compte f�d�r� sur l'application cible avec la m�me adresse e-mail en compromettant le compte de l'utilisateur sur le fournisseur de connexion f�d�r�e.

Enfin, si le MFA est impl�ment� sur un syst�me diff�rent de l'application principale (comme sur un proxy inverse, afin de prot�ger une application h�rit�e qui ne supporte pas nativement le MFA), il peut alors �tre possible de le contourner en se connectant directement � le serveur d'applications principal, comme indiqu� dans le guide sur la fa�on de [mapper l'architecture de l'application](../01-Information_Gathering/10-Map_Application_Architecture.md#content-delivery-network-cdn).

### V�rifier la gestion MFA

La fonctionnalit� utilis�e pour g�rer MFA depuis le compte de l'utilisateur doit �tre test�e pour d�tecter les vuln�rabilit�s, notamment�:

- L'utilisateur doit-il se r�-authentifier pour supprimer ou modifier les param�tres MFA�?
- La fonctionnalit� de gestion MFA est-elle vuln�rable � [falsification de demande intersite](../06-Session_Management_Testing/05-Testing_for_Cross_Site_Request_Forgery.md)�?
- Le param�tre MFA d'autres utilisateurs peut-il �tre modifi� via [Vuln�rabilit�s IDOR](../05-Authorization_Testing/04-Testing_for_Insecure_Direct_Object_References.md)�?

### V�rifier les options de r�cup�ration MFA

De nombreuses applications fourniront aux utilisateurs un moyen de retrouver l'acc�s � leur compte s'ils ne peuvent pas s'authentifier avec leur deuxi�me facteur (par exemple s'ils ont perdu leur t�l�phone). Ces m�canismes peuvent souvent repr�senter une faiblesse importante de l'application, car ils permettent effectivement de contourner le deuxi�me facteur d'authentification.

#### Codes de r�cup�ration

Certaines applications fourniront � l'utilisateur une liste de codes de r�cup�ration ou de sauvegarde lorsqu'elles activent MFA, qui peut �tre utilis�e pour se connecter. Ceux-ci doivent �tre v�rifi�s pour s'assurer :

- Ils sont suffisamment longs et complexes pour se prot�ger des attaques par force brute.
- Ils sont g�n�r�s de mani�re s�curis�e.
- Ils ne peuvent �tre utilis�s qu'une seule fois.
- Une protection contre la force brute est en place (comme le verrouillage du compte).
- L'utilisateur est averti (par e-mail, SMS, etc.) lorsqu'un code est utilis�.

Consultez la section ["Codes de sauvegarde" dans l'aide-m�moire sur les mots de passe oubli�s](https://cheatsheetseries.owasp.org/cheatsheets/Forgot_Password_Cheat_Sheet.html#backup-codes) pour plus de d�tails.

#### Processus de r�initialisation MFA

Si l'application impl�mente un processus de r�initialisation MFA, celui-ci doit �tre test� de la m�me mani�re que le [processus de r�initialisation du mot de passe] (09-Testing_for_Weak_Password_Change_or_Reset_Functionalities.md) est test�. Il est important que ce processus soit *au moins* aussi solide que l'impl�mentation MFA pour l'application.

#### Authentification alternative

Certaines applications permettront � l'utilisateur de prouver son identit� par d'autres moyens, comme l'utilisation de [questions de s�curit�](08-Testing_for_Weak_Security_Question_Answer.md). Cela repr�sente g�n�ralement une faiblesse importante, car les questions de s�curit� offrent un niveau de s�curit� bien inf�rieur � celui de MFA.

### Mots de passe � usage unique

La forme la plus courante de MFA est celle des mots de passe � usage unique (OTP), qui sont g�n�ralement des codes num�riques � six chiffres (bien qu'ils puissent �tre plus longs ou plus courts). Ceux-ci peuvent soit �tre g�n�r�s � la fois par le serveur et l'utilisateur (par exemple, avec une application d'authentification), soit �tre g�n�r�s sur le serveur et envoy�s � l'utilisateur. Cet OTP peut �tre fourni � l'utilisateur de diff�rentes mani�res, notamment�:

| Taper | Descriptif |
|------|-------------|
| Mot de passe � usage unique HMAC (HOPT) | G�n�re un code bas� sur le HMAC d'un secret et d'un compteur partag�. |
| Mot de passe � usage unique bas� sur le temps (TOTP) | G�n�re un code bas� sur le HMAC d'un secret et l'heure actuelle. |
| E-mail | Envoie un code par e-mail. |
| SMS | Envoie un code par SMS. |
| T�l�phone | Envoie un code via un appel vocal � un num�ro de t�l�phone. |

L'OTP est g�n�ralement entr� apr�s que l'utilisateur a fourni son nom d'utilisateur et son mot de passe. Diverses v�rifications doivent �tre effectu�es, notamment :

- Le compte est-il verrouill� apr�s plusieurs tentatives MFA infructueuses�?
- L'adresse IP de l'utilisateur est-elle bloqu�e apr�s plusieurs tentatives MFA infructueuses sur diff�rents comptes�?
- Les tentatives MFA infructueuses sont-elles enregistr�es�?
- Le formulaire est-il vuln�rable aux attaques par injection, y compris [l'injection de caract�res g�n�riques SQL](../07-Input_Validation_Testing/05-Testing_for_SQL_Injection.md#sql-wildcard-injection)�?

Selon le type d'OTP utilis�, il existe �galement d'autres contr�les sp�cifiques � effectuer�:

- Comment les OTP sont-ils envoy�s � l'utilisateur (e-mail, SMS, t�l�phone, etc.)
    - Existe-t-il une limitation du d�bit pour �viter que les spams par SMS/t�l�phone ne co�tent de l'argent�?
- Quelle est la force des OTP (longueur et keyspace)�?
- Combien de temps les OTP sont-ils valables�?
- Plusieurs OTP sont-ils valides � la fois�?
- Les OTP peuvent-ils �tre utilis�s plus d'une fois ?
- Les OTP sont-ils li�s au bon compte utilisateur ou est-il possible de s'authentifier aupr�s d'eux sur d'autres comptes�?

#### HOTP et TOTP

Les codes HOTP et TOTP sont tous deux bas�s sur un secret partag� entre le serveur et l'utilisateur. Pour les codes TOTP, cela est g�n�ralement fourni � l'utilisateur sous la forme d'un code QR qu'il scanne avec une application d'authentification (bien qu'il puisse �galement �tre fourni sous forme de texte secret � saisir manuellement).

Lorsque le secret est g�n�r� sur le serveur, il doit �tre v�rifi� pour s'assurer qu'il est suffisamment long et complexe ([RFC 4226](https://www.rfc-editor.org/rfc/rfc4226#section-4) recommande � moins 160�bits), et qu'il est g�n�r� � l'aide d'une [fonction al�atoire s�curis�e](https://cheatsheetseries.owasp.org/cheatsheets/Cryptographic_Storage_Cheat_Sheet.html#secure-random-number-generation).

Lorsque le secret peut �tre fourni par l'utilisateur, une longueur minimale appropri�e doit �tre appliqu�e et l'entr�e doit �tre v�rifi�e pour les attaques par injection habituelles.

Les codes TOTP sont g�n�ralement valides pendant 30 secondes, mais certaines applications choisissent d'accepter plusieurs codes (tels que les codes pr�c�dents, actuels et suivants) afin de g�rer les diff�rences entre l'heure syst�me sur le serveur et sur l'appareil de l'utilisateur. Certaines applications peuvent autoriser plusieurs codes de part et d'autre du code actuel, ce qui peut permettre � un attaquant de deviner ou de forcer le code plus facilement. Le tableau ci-dessous montre les chances de forcer avec succ�s un code OTP bas� sur un attaquant capable de faire 10 requ�tes par seconde, pour les applications qui acceptent soit uniquement le code actuel, soit plusieurs codes (voir [cet article](https:/ /www.codasecurity.co.uk/articles/mfa-testing#case-study---brute-forcing-totp) pour les calculs derri�re le tableau).

| Codes valides | Taux de r�ussite apr�s 1 heure | Taux de r�ussite apr�s 4 heures | Taux de r�ussite apr�s 12 heures | Taux de r�ussite apr�s 24 heures |
|------------|---------------------------|------- ---------------------|------------------------------------------ -|------------------------------------------|
| 1 | 4% | 13% | 35% | 58% |
| 3 | 10% | 35% | 72% | 92% |
| 5 | 16% | 51% | 88% | 99% |
| 7 | 22% | 63% | 95% | 99% |

#### Courriel, SMS et t�l�phone

Lorsque les codes sont g�n�r�s par le serveur et envoy�s au client, les domaines suivants doivent �tre pris en compte�:

- Le m�canisme de transport (email, SMS ou voix) est-il suffisamment s�curis� pour l'application�?
- Les codes sont-ils suffisamment longs et complexes ?
- Les codes sont-ils g�n�r�s � l'aide d'une [fonction al�atoire s�curis�e](https://cheatsheetseries.owasp.org/cheatsheets/Cryptographic_Storage_Cheat_Sheet.html#secure-random-number-generation)�?
- Combien de temps les codes sont-ils valables ?
- Plusieurs codes sont-ils valides � la fois ou est-ce que la g�n�ration d'un nouveau code invalide le pr�c�dent�?
    - Cela pourrait-il �tre utilis� pour bloquer l'acc�s � un compte en demandant � plusieurs reprises des codes�?
- Existe-t-il une limitation de d�bit suffisante pour emp�cher un attaquant de demander un grand nombre de codes�?
    - Un grand nombre de code envoy� par e-mail peut bloquer le serveur pour l'envoi de spam.
    - Un grand nombre de SMS ou d'appels vocaux peut co�ter de l'argent ou �tre utilis� pour harceler un utilisateur.

### Applications mobiles et notifications push

Une approche alternative aux codes OTP consiste � envoyer une notification push au t�l�phone mobile de l'utilisateur, qu'il peut approuver ou refuser. Cette m�thode est moins courante, car elle n�cessite que l'utilisateur installe un authentificateur sp�cifique � l'application.

Pour �valuer correctement la s�curit� de cela, il faut �tendre la port�e des tests pour couvrir � la fois l'application mobile et toutes les API ou services de support utilis�s par celle-ci�; ce qui signifie qu'il sortirait souvent du cadre d'un test d'application Web traditionnel. Cependant, il existe quelques v�rifications simples qui peuvent �tre effectu�es sans tester l'application mobile, notamment�:

- La notification fournit-elle suffisamment de contexte (adresses IP, emplacement, etc.) pour que l'utilisateur puisse d�cider en connaissance de cause s'il doit l'approuver ou la refuser�?
- Existe-t-il un type de m�canisme de d�fi et de r�ponse (comme fournir un code sur le site Web que l'utilisateur doit saisir dans l'application - souvent appel� ��correspondance de num�ros�� ou ��d�fi de num�ros��)�?
- Existe-t-il une limitation du d�bit ou des m�canismes pour emp�cher l'utilisateur d'�tre spamm� avec des notifications dans l'espoir qu'il en acceptera une aveugl�ment�?

### Filtrage des adresses IP et des emplacements

L'un des facteurs parfois utilis�s avec MFA est l'emplacement ("quelque part o� vous �tes"), bien que la question de savoir si cela constitue un facteur d'authentification appropri� est discutable. Dans le contexte d'une application Web, cela signifie g�n�ralement restreindre l'acc�s � des adresses IP sp�cifiques ou ne pas demander � l'utilisateur un deuxi�me facteur tant qu'il se connecte � partir d'une adresse IP de confiance sp�cifique. Un sc�nario courant pour cela serait d'authentifier les utilisateurs avec uniquement leur mot de passe lorsqu'ils se connectent � partir des plages IP du bureau, mais en exigeant un code OTP lorsqu'ils se connectent d'ailleurs.

Selon l'impl�mentation, il peut �tre possible pour un utilisateur d'usurper une adresse IP de confiance en d�finissant l'en-t�te "X-Forwarded-For", ce qui pourrait lui permettre de contourner cette v�rification. Notez que si l'application ne nettoie pas correctement le contenu de cet en-t�te, il peut �galement �tre possible de mener une attaque telle qu'une injection SQL ici. Si l'application prend en charge IPv6, cela doit �galement �tre v�rifi� pour s'assurer que les restrictions appropri�es sont appliqu�es � ces connexions.

De plus, les adresses IP de confiance doivent �tre examin�es pour s'assurer qu'elles ne pr�sentent aucune faiblesse, par exemple si elles incluent�:

- Adresses IP qui pourraient �tre accessibles par des utilisateurs non fiables (tels que les r�seaux sans fil invit�s dans un bureau).
- Adresse IP attribu�e dynamiquement qui pourrait changer.
- Plages de r�seaux publics o� un attaquant pourrait h�berger son propre syst�me (comme Azure ou AWS).

### Certificats et cartes � puce

Transport Layer Security (TLS) est couramment utilis� pour crypter le trafic entre le client et le serveur, et pour fournir un m�canisme permettant au client de confirmer l'identit� du serveur (en comparant le nom commun (CN) ou le nom alternatif du sujet (SAN) sur le certificat au domaine demand�). Cependant, il peut �galement fournir un m�canisme permettant au serveur de confirmer l'identit� du client, appel� authentification par certificat client ou TLS mutuel (mTLS). Une discussion compl�te de l'authentification par certificat client sort du cadre de ce guide, mais le principe cl� est que l'utilisateur pr�sente un certificat num�rique (stock� sur sa machine ou sur une carte � puce), qui est valid� par le serveur.

La premi�re �tape du test consiste � d�terminer si l'application cible restreint les autorit�s de certification (CA) approuv�es pour �mettre des certificats. Ces informations peuvent �tre obtenues � l'aide de divers outils ou en examinant manuellement la poign�e de main TLS. Le moyen le plus simple consiste � utiliser le `s_client` d'OpenSSL�:

```bash
$ openssl s_client -connect example:443
[...]
Acceptable client certificate CA names
C = US, ST = Example, L = Example, O = Example Org, CN = Example Org Root Certificate Authority
Client Certificate Types: RSA sign, DSA sign, ECDSA sign
```

S'il n'y a pas de restrictions, il peut �tre possible de s'authentifier � l'aide d'un certificat d'une autre autorit� de certification. S'il y a des restrictions mais qu'elles sont mal impl�ment�es, il peut �tre possible de cr�er une CA locale avec le nom correct ("Example Org Root Certificate Authority" dans l'exemple ci-dessus), et d'utiliser cette nouvelle CA pour signer les certificats clients.

Si un certificat valide peut �tre obtenu, il convient �galement de v�rifier que le certificat ne peut �tre utilis� que pour l'utilisateur pour lequel il est d�livr� (c'est-�-dire que vous ne pouvez pas utiliser un certificat d�livr� � Alice pour vous authentifier sur le compte de Bob). De plus, les certificats doivent �tre v�rifi�s pour s'assurer qu'ils n'ont ni expir� ni �t� r�voqu�s.

## Cas de test associ�s

- [Test du m�canisme de verrouillage faible] (03-Testing_for_Weak_Lock_Out_Mechanism.md)
- [Test des fonctionnalit�s de changement ou de r�initialisation de mot de passe faibles] (09-Testing_for_Weak_Password_Change_or_Reset_Functionalities.md)

## Correction

Veiller � ce que:

- MFA est mis en �uvre pour tous les comptes et fonctionnalit�s pertinents sur les applications.
- Les m�thodes de support MFA sont adapt�es � l'application.
- Les m�canismes utilis�s pour mettre en �uvre la MFA sont correctement s�curis�s et prot�g�s contre les attaques par force brute.
- Il existe un audit et une journalisation appropri�s pour toutes les activit�s li�es � MFA.

Consultez l'[Aide-m�moire sur l'authentification multifacteur OWASP](https://cheatsheetseries.owasp.org/cheatsheets/Multifactor_Authentication_Cheat_Sheet.html) pour plus de recommandations.

## R�f�rences

- [Aide-m�moire d'authentification multifacteur OWASP] (https://cheatsheetseries.owasp.org/cheatsheets/Multifactor_Authentication_Cheat_Sheet.html)
