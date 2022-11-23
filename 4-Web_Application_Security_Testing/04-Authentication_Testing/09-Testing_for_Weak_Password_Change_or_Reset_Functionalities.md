# Test des fonctionnalit�s de changement ou de r�initialisation de mot de passe faibles

|ID          |
|------------|
|WSTG-ATHN-09|

## Sommaire

Pour toute application qui demande � l'utilisateur de s'authentifier avec un mot de passe, il doit y avoir un m�canisme par lequel l'utilisateur peut retrouver l'acc�s � son compte s'il oublie son mot de passe. Bien que cela puisse parfois �tre un processus manuel qui implique de contacter le propri�taire du site Web ou une �quipe d'assistance, les utilisateurs sont souvent autoris�s � effectuer une r�initialisation de mot de passe en libre-service et � retrouver l'acc�s � leur compte en fournissant d'autres preuves de leur identit�. .

�tant donn� que cette fonctionnalit� fournit un moyen direct de compromettre le compte de l'utilisateur, il est crucial qu'elle soit impl�ment�e en toute s�curit�.

## Objectifs des tests

- D�terminez si la fonctionnalit� de modification et de r�initialisation du mot de passe permet de compromettre les comptes.

## Comment tester

### La collecte d'informations

La premi�re �tape consiste � recueillir des informations sur les m�canismes disponibles pour permettre � l'utilisateur de r�initialiser son mot de passe sur l'application. S'il existe plusieurs interfaces sur le m�me site (telles qu'une interface Web, une application mobile et une API), elles doivent toutes �tre examin�es, au cas o� elles fourniraient des fonctionnalit�s diff�rentes.

Une fois que cela a �t� �tabli, d�terminez quelles informations sont requises pour qu'un utilisateur lance une r�initialisation de mot de passe. Il peut s'agir du nom d'utilisateur ou de l'adresse e-mail (qui peuvent tous deux �tre obtenus � partir d'informations publiques), mais il peut �galement s'agir d'un identifiant d'utilisateur g�n�r� en interne.

### Pr�occupations g�n�rales

Quelles que soient les m�thodes sp�cifiques utilis�es pour r�initialiser les mots de passe, il existe un certain nombre de domaines communs qui doivent �tre pris en compte�:

- Le processus de r�initialisation du mot de passe est-il plus faible que le processus d'authentification�?

  Le processus de r�initialisation du mot de passe fournit un m�canisme alternatif pour acc�der au compte d'un utilisateur et doit donc �tre au moins aussi s�curis� que le processus d'authentification habituel. Cependant, cela peut fournir un moyen plus simple de compromettre le compte, en particulier s'il utilise des facteurs d'authentification plus faibles tels que des questions de s�curit�.

  De plus, le processus de r�initialisation du mot de passe peut contourner l'obligation d'utiliser l'authentification multifacteur (MFA), ce qui peut r�duire consid�rablement la s�curit� de l'application.

- Existe-t-il une limitation du d�bit ou une autre protection contre les attaques automatis�es�?

  Comme pour tout m�canisme d'authentification, le processus de r�initialisation du mot de passe doit �tre prot�g� contre les attaques automatis�es ou par force brute. Il existe une vari�t� de m�thodes diff�rentes qui peuvent �tre utilis�es pour y parvenir, telles que la limitation du d�bit ou l'utilisation de CAPTCHA. Celles-ci sont particuli�rement importantes pour les fonctionnalit�s qui d�clenchent des actions externes (telles que l'envoi d'un e-mail ou d'un SMS), ou lorsque l'utilisateur saisit un jeton de r�initialisation de mot de passe.

  Il est �galement possible de se prot�ger contre les attaques par force brute en excluant le compte du processus de r�initialisation du mot de passe apr�s un certain nombre de tentatives cons�cutives. Cependant, cela pourrait �galement emp�cher un utilisateur l�gitime de r�initialiser son mot de passe et de retrouver l'acc�s � son compte.

- Est-il vuln�rable aux attaques courantes ?

  Outre les domaines sp�cifiques abord�s dans ce guide, il est �galement important de rechercher d'autres vuln�rabilit�s courantes telles que l'injection SQL ou les scripts intersites.

- Le processus de r�initialisation permet-il l'�num�ration des utilisateurs�?

  Consultez le guide [Test d'�num�ration de compte](../03-Identity_Management_Testing/04-Testing_for_Account_Enumeration_and_Guessable_User_Account.md) pour plus d'informations.

### E-mail - Nouveau mot de passe envoy�

Dans ce mod�le, l'utilisateur re�oit un nouveau mot de passe par e-mail une fois qu'il a prouv� son identit�. Ceci est consid�r� comme moins s�r pour deux raisons principales�:

- Le mot de passe est envoy� � l'utilisateur sous une forme non crypt�e.
- Le mot de passe du compte est modifi� lorsque la demande est faite, bloquant ainsi l'acc�s de l'utilisateur � son compte jusqu'� ce qu'il re�oive l'e-mail. En faisant des demandes r�p�t�es, il est possible d'emp�cher un utilisateur de pouvoir acc�der � son compte.

Lorsque cette approche est utilis�e, les domaines suivants doivent �tre examin�s�:

- L'utilisateur est-il oblig� de changer de mot de passe lors de la premi�re connexion�?

  Le nouveau mot de passe est envoy� par e-mail non crypt� et peut rester ind�finiment dans la bo�te de r�ception de l'utilisateur s'il ne supprime pas l'e-mail. En tant que tel, l'utilisateur devrait �tre tenu de changer le mot de passe d�s qu'il se connecte pour la premi�re fois.

- Le mot de passe est-il g�n�r� de mani�re s�curis�e�?

  Le mot de passe doit �tre g�n�r� � l'aide d'un g�n�rateur de nombres pseudo-al�atoires cryptographiquement s�curis�s (CSPRNG) et doit �tre suffisamment long pour emp�cher la devinette du mot de passe ou les attaques par force brute. Pour une exp�rience utilisateur s�curis�e et conviviale, il doit �tre g�n�r� � l'aide d'une approche de type phrase de passe s�curis�e (c'est-�-dire en combinant plusieurs mots), plut�t qu'une cha�ne de caract�res al�atoires.

- Le mot de passe existant de l'utilisateur lui est-il envoy�?

  Plut�t que de g�n�rer un nouveau mot de passe pour l'utilisateur, certaines applications enverront � l'utilisateur leur mot de passe existant. Il s'agit d'une approche tr�s peu s�curis�e, car elle expose leur mot de passe actuel sur des e-mails non crypt�s. De plus, si le site est capable de r�cup�rer le mot de passe existant, cela implique que les mots de passe sont soit stock�s � l'aide d'un cryptage r�versible, soit (plus probablement) en texte brut non crypt�, les deux repr�sentant une grave faiblesse de s�curit�.

- Les e-mails sont-ils envoy�s depuis un domaine avec une protection anti-spoofing�?

  Le domaine doit impl�menter SPF, DKIM et DMARC pour emp�cher les attaquants d'usurper ses e-mails, ce qui pourrait �tre utilis� dans le cadre d'une attaque d'ing�nierie sociale.

- Le courrier �lectronique est-il consid�r� comme suffisamment s�curis� ?

  Les e-mails sont g�n�ralement envoy�s non chiffr�s et, dans de nombreux cas, le compte de messagerie de l'utilisateur ne sera pas prot�g� par MFA. Il peut �galement �tre partag� entre plusieurs personnes, en particulier dans un environnement d'entreprise.

  D�terminez si la fonctionnalit� de r�initialisation du mot de passe par e-mail est appropri�e, en fonction du contexte de l'application test�e.

### E-mail - Lien envoy�

Dans ce mod�le, l'utilisateur re�oit par e-mail un lien contenant un jeton. Ils peuvent alors cliquer sur ce lien, et sont invit�s � entrer un nouveau mot de passe sur le site. Il s'agit de l'approche la plus couramment utilis�e pour la r�initialisation du mot de passe, mais elle est plus complexe � mettre en �uvre que l'approche d�crite pr�c�demment. Les principaux domaines � tester sont :

- Le lien utilise-t-il HTTPS�?

  Si le jeton est envoy� via HTTP non chiffr�, il peut �tre possible pour un attaquant de l'intercepter.

- Le lien peut-il �tre utilis� plusieurs fois ?

  Les liens doivent expirer apr�s leur utilisation, sinon ils fournissent une porte d�rob�e persistante pour le compte.

- Le lien expire-t-il s'il reste inutilis�?

  Les liens doivent �tre limit�s dans le temps. La dur�e exacte d�pendra du site, mais elle devrait rarement d�passer une heure.

- Le jeton est-il suffisamment long et al�atoire ?

  La s�curit� du processus d�pend enti�rement de l'incapacit� d'un attaquant � deviner ou � forcer brutalement un jeton. Les jetons doivent �tre g�n�r�s avec un g�n�rateur de nombres pseudo-al�atoires cryptographiquement s�curis�s (CSPRNG) et doivent �tre suffisamment longs pour qu'il soit impossible pour un attaquant de deviner ou de forcer brutalement. Au moins 128 bits (ou 32 caract�res hexad�cimaux) est un minimum suffisant pour rendre une telle attaque en ligne impraticable.

  Les jetons ne doivent jamais �tre g�n�r�s sur la base de valeurs connues, comme en prenant le hachage MD5 de l'e-mail de l'utilisateur avec `md5($email)`, ou en utilisant des GUID qui peuvent utiliser des fonctions PRNG non s�curis�es, ou peuvent m�me ne pas �tre al�atoires selon le type .

  Une approche alternative aux jetons al�atoires consiste � utiliser un jeton sign� cryptographiquement tel qu'un JWT. Dans ce cas, les v�rifications JWT habituelles doivent �tre effectu�es (la signature est-elle v�rifi�e, l'algorithme "nONe" peut-il �tre utilis�, la cl� HMAC peut-elle �tre brute-forc�e, etc.). Consultez le guide [Test des jetons Web JSON](../06-Session_Management_Testing/10-Testing_JSON_Web_Tokens.md) pour plus d'informations.

- Le lien contient-il un ID utilisateur�?

  Parfois, le lien de r�initialisation du mot de passe peut inclure un ID utilisateur ainsi qu'un jeton, tel que `reset.php?userid=1&token=123456`. Dans ce cas, il peut �tre possible de modifier le param�tre `userid` pour r�initialiser les mots de passe des autres utilisateurs.

- Pouvez-vous injecter un en-t�te d'h�te diff�rent�?

  Si l'application fait confiance � la valeur de l'en-t�te "Host" et l'utilise pour g�n�rer le lien de r�initialisation du mot de passe, il peut �tre possible de voler des jetons en injectant un en-t�te "Host" modifi� dans la requ�te. Consultez le guide [Test pour l'injection d'en-t�te d'h�te](../07-Input_Validation_Testing/17-Testing_for_Host_Header_Injection.md) pour plus d'informations.

- Le lien est-il expos� � des tiers ?

  Si la page vers laquelle l'utilisateur est redirig� inclut du contenu d'autres parties (comme le chargement de scripts � partir d'autres domaines), le jeton de r�initialisation dans l'URL peut �tre expos� dans l'en-t�te HTTP "Referer" envoy� dans ces requ�tes. L'en-t�te HTTP "Referrer-Policy" peut �tre utilis� pour se prot�ger contre cela, alors v�rifiez si un est d�fini pour la page.

  De plus, si la page comprend des scripts de suivi, d'analyse ou de publicit�, le jeton leur sera �galement expos�.

- Les e-mails sont-ils envoy�s depuis un domaine avec une protection anti-spoofing�?

  Le domaine doit impl�menter SPF, DKIM et DMARC pour emp�cher les attaquants d'usurper ses e-mails, ce qui pourrait �tre utilis� dans le cadre d'une attaque d'ing�nierie sociale.

- Le courrier �lectronique est-il consid�r� comme suffisamment s�curis� ?

  Les e-mails sont g�n�ralement envoy�s non chiffr�s et, dans de nombreux cas, le compte de messagerie de l'utilisateur ne sera pas prot�g� par MFA. Il peut �galement �tre partag� entre plusieurs personnes, en particulier dans un environnement d'entreprise.

  D�terminez si la fonctionnalit� de r�initialisation du mot de passe par e-mail est appropri�e, en fonction du contexte de l'application test�e.

### Jetons envoy�s par SMS ou appel t�l�phonique

Plut�t que d'envoyer un jeton dans un e-mail, une approche alternative consiste � l'envoyer via SMS ou un appel t�l�phonique automatis�, que l'utilisateur saisira ensuite sur l'application. Les principaux domaines � tester sont :

- Le jeton est-il suffisamment long et al�atoire ?

  Les jetons envoy�s de cette mani�re sont g�n�ralement plus courts, car ils sont destin�s � �tre saisis manuellement par l'utilisateur, plut�t que d'�tre int�gr�s dans un lien. Il est assez courant que les applications utilisent six chiffres num�riques, ce qui ne fournit qu'environ 20 bits de s�curit� (r�alisable pour une attaque par force brute en ligne), plut�t que le jeton d'e-mail g�n�ralement plus long.

  Il est donc beaucoup plus important que la fonctionnalit� de r�initialisation du mot de passe soit prot�g�e contre les attaques par force brute.

- Le jeton peut-il �tre utilis� plusieurs fois ?

  Les jetons doivent �tre invalid�s apr�s leur utilisation, sinon ils fournissent une porte d�rob�e persistante pour le compte.

- Le jeton expire-t-il s'il reste inutilis�?

  Comme les jetons plus courts sont plus sensibles aux attaques par force brute, un d�lai d'expiration plus court doit �tre mis en �uvre pour limiter la fen�tre disponible pour qu'un attaquant puisse mener une attaque.

- Existe-t-il des limites et des restrictions de d�bit appropri�es�?

  L'envoi d'un SMS ou le d�clenchement d'un appel t�l�phonique automatis� � un utilisateur est nettement plus perturbateur que l'envoi d'un e-mail, et pourrait �tre utilis� pour harceler un utilisateur, voire mener une attaque par d�ni de service contre son t�l�phone. L'application doit impl�menter une limitation de d�bit pour �viter cela.

  De plus, les SMS et les appels t�l�phoniques entra�nent souvent des co�ts financiers pour l'exp�diteur. Si un attaquant est capable de provoquer l'envoi d'un grand nombre de messages, cela pourrait entra�ner des co�ts importants pour l'op�rateur du site Web. Cela est particuli�rement vrai s'ils sont envoy�s vers des num�ros internationaux ou surtax�s. Cependant, l'autorisation des num�ros internationaux peut �tre une exigence de l'application.

- Un SMS ou un appel t�l�phonique est-il consid�r� comme suffisamment s�curis� ?

  [Une vari�t� d'attaques](https://www.ncsc.gov.uk/guidance/protecting-sms-messages-used-in-critical-business-processes#section_4) ont �t� d�montr�es et permettraient � un attaquant de d�tourner efficacement SMS, les points de vue divergent quant � savoir si le SMS est suffisamment s�curis� pour �tre utilis� comme facteur d'authentification.

  Il est g�n�ralement possible de r�pondre � un appel t�l�phonique automatis� avec un acc�s physique � un appareil, sans avoir besoin d'un code PIN ou d'une empreinte digitale pour d�verrouiller le t�l�phone. Dans certaines circonstances (comme un environnement de bureau partag�), cela pourrait permettre � un attaquant interne de r�initialiser de mani�re triviale le mot de passe d'un autre utilisateur en se dirigeant vers son bureau lorsqu'il n'est pas au bureau.

  D�terminez si les SMS ou les appels t�l�phoniques automatis�s sont appropri�s, en fonction du contexte de l'application test�e.

### Questions de s�curit�

Plut�t que de leur envoyer un lien ou un nouveau mot de passe, les questions de s�curit� peuvent �tre utilis�es comme m�canisme pour authentifier l'utilisateur. Cette approche est consid�r�e comme faible et ne doit pas �tre utilis�e si de meilleures options sont disponibles.

Consultez le guide [Test des questions de s�curit� faible](08-Testing_for_Weak_Security_Question_Answer.md) pour plus d'informations.

### Changements de mot de passe authentifi�s

Une fois que l'utilisateur a prouv� son identit� (soit par un lien de r�initialisation de mot de passe, un code de r�cup�ration, soit en se connectant sur l'application), il devrait pouvoir changer son mot de passe. Les principaux domaines � tester sont�:

- Lors de la d�finition du mot de passe, pouvez-vous sp�cifier l'ID utilisateur�?

  Si l'ID utilisateur est inclus dans la demande de r�initialisation du mot de passe et n'est pas valid�, il peut �tre possible de le modifier et de changer les mots de passe des autres utilisateurs.

- L'utilisateur doit-il se r�-authentifier�?

  Si un utilisateur connect� essaie de changer son mot de passe, il doit �tre invit� � se r�-authentifier avec son mot de passe actuel afin de se prot�ger contre un attaquant obtenant un acc�s temporaire � une session sans surveillance. Si l'authentification MFA est activ�e pour l'utilisateur, il se r�authentifiera g�n�ralement avec cela, plut�t qu'avec son mot de passe.

- Le formulaire de changement de mot de passe est-il vuln�rable au CSRF�?

  Si l'utilisateur n'est pas tenu de se r�-authentifier, il peut alors �tre possible d'effectuer une attaque CSRF contre le formulaire de r�initialisation du mot de passe, permettant � son compte d'�tre compromis. Pour plus d'informations, consultez le guide [Testing for Cross-Site Request Forgery](../06-Session_Management_Testing/05-Testing_for_Cross_Site_Request_Forgery.md).

- Une politique de mot de passe forte et efficace est-elle appliqu�e�?

  La politique de mot de passe doit �tre coh�rente pour l'ensemble des fonctionnalit�s d'enregistrement, de changement de mot de passe et de r�initialisation du mot de passe. Consultez le guide [Testing for Weak Password Policy](07-Testing_for_Weak_Password_Policy.md) pour plus d'informations.

## R�f�rences

- [Feuille de triche de mot de passe oubli� OWASP] (https://cheatsheetseries.owasp.org/cheatsheets/Forgot_Password_Cheat_Sheet.html)
