# Test des fonctionnalités de changement ou de réinitialisation de mot de passe faibles

|ID          |
|------------|
|WSTG-ATHN-09|

## Sommaire

Pour toute application qui demande à l'utilisateur de s'authentifier avec un mot de passe, il doit y avoir un mécanisme par lequel l'utilisateur peut retrouver l'accès à son compte s'il oublie son mot de passe. Bien que cela puisse parfois être un processus manuel qui implique de contacter le propriétaire du site Web ou une équipe d'assistance, les utilisateurs sont souvent autorisés à effectuer une réinitialisation de mot de passe en libre-service et à retrouver l'accès à leur compte en fournissant d'autres preuves de leur identité. .

Étant donné que cette fonctionnalité fournit un moyen direct de compromettre le compte de l'utilisateur, il est crucial qu'elle soit implémentée en toute sécurité.

## Objectifs des tests

- Déterminez si la fonctionnalité de modification et de réinitialisation du mot de passe permet de compromettre les comptes.

## Comment tester

### La collecte d'informations

La première étape consiste à recueillir des informations sur les mécanismes disponibles pour permettre à l'utilisateur de réinitialiser son mot de passe sur l'application. S'il existe plusieurs interfaces sur le même site (telles qu'une interface Web, une application mobile et une API), elles doivent toutes être examinées, au cas où elles fourniraient des fonctionnalités différentes.

Une fois que cela a été établi, déterminez quelles informations sont requises pour qu'un utilisateur lance une réinitialisation de mot de passe. Il peut s'agir du nom d'utilisateur ou de l'adresse e-mail (qui peuvent tous deux être obtenus à partir d'informations publiques), mais il peut également s'agir d'un identifiant d'utilisateur généré en interne.

### Préoccupations générales

Quelles que soient les méthodes spécifiques utilisées pour réinitialiser les mots de passe, il existe un certain nombre de domaines communs qui doivent être pris en compte :

- Le processus de réinitialisation du mot de passe est-il plus faible que le processus d'authentification ?

  Le processus de réinitialisation du mot de passe fournit un mécanisme alternatif pour accéder au compte d'un utilisateur et doit donc être au moins aussi sécurisé que le processus d'authentification habituel. Cependant, cela peut fournir un moyen plus simple de compromettre le compte, en particulier s'il utilise des facteurs d'authentification plus faibles tels que des questions de sécurité.

  De plus, le processus de réinitialisation du mot de passe peut contourner l'obligation d'utiliser l'authentification multifacteur (MFA), ce qui peut réduire considérablement la sécurité de l'application.

- Existe-t-il une limitation du débit ou une autre protection contre les attaques automatisées ?

  Comme pour tout mécanisme d'authentification, le processus de réinitialisation du mot de passe doit être protégé contre les attaques automatisées ou par force brute. Il existe une variété de méthodes différentes qui peuvent être utilisées pour y parvenir, telles que la limitation du débit ou l'utilisation de CAPTCHA. Celles-ci sont particulièrement importantes pour les fonctionnalités qui déclenchent des actions externes (telles que l'envoi d'un e-mail ou d'un SMS), ou lorsque l'utilisateur saisit un jeton de réinitialisation de mot de passe.

  Il est également possible de se protéger contre les attaques par force brute en excluant le compte du processus de réinitialisation du mot de passe après un certain nombre de tentatives consécutives. Cependant, cela pourrait également empêcher un utilisateur légitime de réinitialiser son mot de passe et de retrouver l'accès à son compte.

- Est-il vulnérable aux attaques courantes ?

  Outre les domaines spécifiques abordés dans ce guide, il est également important de rechercher d'autres vulnérabilités courantes telles que l'injection SQL ou les scripts intersites.

- Le processus de réinitialisation permet-il l'énumération des utilisateurs ?

  Consultez le guide [Test d'énumération de compte](../03-Identity_Management_Testing/04-Testing_for_Account_Enumeration_and_Guessable_User_Account.md) pour plus d'informations.

### E-mail - Nouveau mot de passe envoyé

Dans ce modèle, l'utilisateur reçoit un nouveau mot de passe par e-mail une fois qu'il a prouvé son identité. Ceci est considéré comme moins sûr pour deux raisons principales :

- Le mot de passe est envoyé à l'utilisateur sous une forme non cryptée.
- Le mot de passe du compte est modifié lorsque la demande est faite, bloquant ainsi l'accès de l'utilisateur à son compte jusqu'à ce qu'il reçoive l'e-mail. En faisant des demandes répétées, il est possible d'empêcher un utilisateur de pouvoir accéder à son compte.

Lorsque cette approche est utilisée, les domaines suivants doivent être examinés :

- L'utilisateur est-il obligé de changer de mot de passe lors de la première connexion ?

  Le nouveau mot de passe est envoyé par e-mail non crypté et peut rester indéfiniment dans la boîte de réception de l'utilisateur s'il ne supprime pas l'e-mail. En tant que tel, l'utilisateur devrait être tenu de changer le mot de passe dès qu'il se connecte pour la première fois.

- Le mot de passe est-il généré de manière sécurisée ?

  Le mot de passe doit être généré à l'aide d'un générateur de nombres pseudo-aléatoires cryptographiquement sécurisés (CSPRNG) et doit être suffisamment long pour empêcher la devinette du mot de passe ou les attaques par force brute. Pour une expérience utilisateur sécurisée et conviviale, il doit être généré à l'aide d'une approche de type phrase de passe sécurisée (c'est-à-dire en combinant plusieurs mots), plutôt qu'une chaîne de caractères aléatoires.

- Le mot de passe existant de l'utilisateur lui est-il envoyé ?

  Plutôt que de générer un nouveau mot de passe pour l'utilisateur, certaines applications enverront à l'utilisateur leur mot de passe existant. Il s'agit d'une approche très peu sécurisée, car elle expose leur mot de passe actuel sur des e-mails non cryptés. De plus, si le site est capable de récupérer le mot de passe existant, cela implique que les mots de passe sont soit stockés à l'aide d'un cryptage réversible, soit (plus probablement) en texte brut non crypté, les deux représentant une grave faiblesse de sécurité.

- Les e-mails sont-ils envoyés depuis un domaine avec une protection anti-spoofing ?

  Le domaine doit implémenter SPF, DKIM et DMARC pour empêcher les attaquants d'usurper ses e-mails, ce qui pourrait être utilisé dans le cadre d'une attaque d'ingénierie sociale.

- Le courrier électronique est-il considéré comme suffisamment sécurisé ?

  Les e-mails sont généralement envoyés non chiffrés et, dans de nombreux cas, le compte de messagerie de l'utilisateur ne sera pas protégé par MFA. Il peut également être partagé entre plusieurs personnes, en particulier dans un environnement d'entreprise.

  Déterminez si la fonctionnalité de réinitialisation du mot de passe par e-mail est appropriée, en fonction du contexte de l'application testée.

### E-mail - Lien envoyé

Dans ce modèle, l'utilisateur reçoit par e-mail un lien contenant un jeton. Ils peuvent alors cliquer sur ce lien, et sont invités à entrer un nouveau mot de passe sur le site. Il s'agit de l'approche la plus couramment utilisée pour la réinitialisation du mot de passe, mais elle est plus complexe à mettre en œuvre que l'approche décrite précédemment. Les principaux domaines à tester sont :

- Le lien utilise-t-il HTTPS ?

  Si le jeton est envoyé via HTTP non chiffré, il peut être possible pour un attaquant de l'intercepter.

- Le lien peut-il être utilisé plusieurs fois ?

  Les liens doivent expirer après leur utilisation, sinon ils fournissent une porte dérobée persistante pour le compte.

- Le lien expire-t-il s'il reste inutilisé ?

  Les liens doivent être limités dans le temps. La durée exacte dépendra du site, mais elle devrait rarement dépasser une heure.

- Le jeton est-il suffisamment long et aléatoire ?

  La sécurité du processus dépend entièrement de l'incapacité d'un attaquant à deviner ou à forcer brutalement un jeton. Les jetons doivent être générés avec un générateur de nombres pseudo-aléatoires cryptographiquement sécurisés (CSPRNG) et doivent être suffisamment longs pour qu'il soit impossible pour un attaquant de deviner ou de forcer brutalement. Au moins 128 bits (ou 32 caractères hexadécimaux) est un minimum suffisant pour rendre une telle attaque en ligne impraticable.

  Les jetons ne doivent jamais être générés sur la base de valeurs connues, comme en prenant le hachage MD5 de l'e-mail de l'utilisateur avec `md5($email)`, ou en utilisant des GUID qui peuvent utiliser des fonctions PRNG non sécurisées, ou peuvent même ne pas être aléatoires selon le type .

  Une approche alternative aux jetons aléatoires consiste à utiliser un jeton signé cryptographiquement tel qu'un JWT. Dans ce cas, les vérifications JWT habituelles doivent être effectuées (la signature est-elle vérifiée, l'algorithme "nONe" peut-il être utilisé, la clé HMAC peut-elle être brute-forcée, etc.). Consultez le guide [Test des jetons Web JSON](../06-Session_Management_Testing/10-Testing_JSON_Web_Tokens.md) pour plus d'informations.

- Le lien contient-il un ID utilisateur ?

  Parfois, le lien de réinitialisation du mot de passe peut inclure un ID utilisateur ainsi qu'un jeton, tel que `reset.php?userid=1&token=123456`. Dans ce cas, il peut être possible de modifier le paramètre `userid` pour réinitialiser les mots de passe des autres utilisateurs.

- Pouvez-vous injecter un en-tête d'hôte différent ?

  Si l'application fait confiance à la valeur de l'en-tête "Host" et l'utilise pour générer le lien de réinitialisation du mot de passe, il peut être possible de voler des jetons en injectant un en-tête "Host" modifié dans la requête. Consultez le guide [Test pour l'injection d'en-tête d'hôte](../07-Input_Validation_Testing/17-Testing_for_Host_Header_Injection.md) pour plus d'informations.

- Le lien est-il exposé à des tiers ?

  Si la page vers laquelle l'utilisateur est redirigé inclut du contenu d'autres parties (comme le chargement de scripts à partir d'autres domaines), le jeton de réinitialisation dans l'URL peut être exposé dans l'en-tête HTTP "Referer" envoyé dans ces requêtes. L'en-tête HTTP "Referrer-Policy" peut être utilisé pour se protéger contre cela, alors vérifiez si un est défini pour la page.

  De plus, si la page comprend des scripts de suivi, d'analyse ou de publicité, le jeton leur sera également exposé.

- Les e-mails sont-ils envoyés depuis un domaine avec une protection anti-spoofing ?

  Le domaine doit implémenter SPF, DKIM et DMARC pour empêcher les attaquants d'usurper ses e-mails, ce qui pourrait être utilisé dans le cadre d'une attaque d'ingénierie sociale.

- Le courrier électronique est-il considéré comme suffisamment sécurisé ?

  Les e-mails sont généralement envoyés non chiffrés et, dans de nombreux cas, le compte de messagerie de l'utilisateur ne sera pas protégé par MFA. Il peut également être partagé entre plusieurs personnes, en particulier dans un environnement d'entreprise.

  Déterminez si la fonctionnalité de réinitialisation du mot de passe par e-mail est appropriée, en fonction du contexte de l'application testée.

### Jetons envoyés par SMS ou appel téléphonique

Plutôt que d'envoyer un jeton dans un e-mail, une approche alternative consiste à l'envoyer via SMS ou un appel téléphonique automatisé, que l'utilisateur saisira ensuite sur l'application. Les principaux domaines à tester sont :

- Le jeton est-il suffisamment long et aléatoire ?

  Les jetons envoyés de cette manière sont généralement plus courts, car ils sont destinés à être saisis manuellement par l'utilisateur, plutôt que d'être intégrés dans un lien. Il est assez courant que les applications utilisent six chiffres numériques, ce qui ne fournit qu'environ 20 bits de sécurité (réalisable pour une attaque par force brute en ligne), plutôt que le jeton d'e-mail généralement plus long.

  Il est donc beaucoup plus important que la fonctionnalité de réinitialisation du mot de passe soit protégée contre les attaques par force brute.

- Le jeton peut-il être utilisé plusieurs fois ?

  Les jetons doivent être invalidés après leur utilisation, sinon ils fournissent une porte dérobée persistante pour le compte.

- Le jeton expire-t-il s'il reste inutilisé ?

  Comme les jetons plus courts sont plus sensibles aux attaques par force brute, un délai d'expiration plus court doit être mis en œuvre pour limiter la fenêtre disponible pour qu'un attaquant puisse mener une attaque.

- Existe-t-il des limites et des restrictions de débit appropriées ?

  L'envoi d'un SMS ou le déclenchement d'un appel téléphonique automatisé à un utilisateur est nettement plus perturbateur que l'envoi d'un e-mail, et pourrait être utilisé pour harceler un utilisateur, voire mener une attaque par déni de service contre son téléphone. L'application doit implémenter une limitation de débit pour éviter cela.

  De plus, les SMS et les appels téléphoniques entraînent souvent des coûts financiers pour l'expéditeur. Si un attaquant est capable de provoquer l'envoi d'un grand nombre de messages, cela pourrait entraîner des coûts importants pour l'opérateur du site Web. Cela est particulièrement vrai s'ils sont envoyés vers des numéros internationaux ou surtaxés. Cependant, l'autorisation des numéros internationaux peut être une exigence de l'application.

- Un SMS ou un appel téléphonique est-il considéré comme suffisamment sécurisé ?

  [Une variété d'attaques](https://www.ncsc.gov.uk/guidance/protecting-sms-messages-used-in-critical-business-processes#section_4) ont été démontrées et permettraient à un attaquant de détourner efficacement SMS, les points de vue divergent quant à savoir si le SMS est suffisamment sécurisé pour être utilisé comme facteur d'authentification.

  Il est généralement possible de répondre à un appel téléphonique automatisé avec un accès physique à un appareil, sans avoir besoin d'un code PIN ou d'une empreinte digitale pour déverrouiller le téléphone. Dans certaines circonstances (comme un environnement de bureau partagé), cela pourrait permettre à un attaquant interne de réinitialiser de manière triviale le mot de passe d'un autre utilisateur en se dirigeant vers son bureau lorsqu'il n'est pas au bureau.

  Déterminez si les SMS ou les appels téléphoniques automatisés sont appropriés, en fonction du contexte de l'application testée.

### Questions de sécurité

Plutôt que de leur envoyer un lien ou un nouveau mot de passe, les questions de sécurité peuvent être utilisées comme mécanisme pour authentifier l'utilisateur. Cette approche est considérée comme faible et ne doit pas être utilisée si de meilleures options sont disponibles.

Consultez le guide [Test des questions de sécurité faible](08-Testing_for_Weak_Security_Question_Answer.md) pour plus d'informations.

### Changements de mot de passe authentifiés

Une fois que l'utilisateur a prouvé son identité (soit par un lien de réinitialisation de mot de passe, un code de récupération, soit en se connectant sur l'application), il devrait pouvoir changer son mot de passe. Les principaux domaines à tester sont :

- Lors de la définition du mot de passe, pouvez-vous spécifier l'ID utilisateur ?

  Si l'ID utilisateur est inclus dans la demande de réinitialisation du mot de passe et n'est pas validé, il peut être possible de le modifier et de changer les mots de passe des autres utilisateurs.

- L'utilisateur doit-il se ré-authentifier ?

  Si un utilisateur connecté essaie de changer son mot de passe, il doit être invité à se ré-authentifier avec son mot de passe actuel afin de se protéger contre un attaquant obtenant un accès temporaire à une session sans surveillance. Si l'authentification MFA est activée pour l'utilisateur, il se réauthentifiera généralement avec cela, plutôt qu'avec son mot de passe.

- Le formulaire de changement de mot de passe est-il vulnérable au CSRF ?

  Si l'utilisateur n'est pas tenu de se ré-authentifier, il peut alors être possible d'effectuer une attaque CSRF contre le formulaire de réinitialisation du mot de passe, permettant à son compte d'être compromis. Pour plus d'informations, consultez le guide [Testing for Cross-Site Request Forgery](../06-Session_Management_Testing/05-Testing_for_Cross_Site_Request_Forgery.md).

- Une politique de mot de passe forte et efficace est-elle appliquée ?

  La politique de mot de passe doit être cohérente pour l'ensemble des fonctionnalités d'enregistrement, de changement de mot de passe et de réinitialisation du mot de passe. Consultez le guide [Testing for Weak Password Policy](07-Testing_for_Weak_Password_Policy.md) pour plus d'informations.

## Références

- [Feuille de triche de mot de passe oublié OWASP](https://cheatsheetseries.owasp.org/cheatsheets/Forgot_Password_Cheat_Sheet.html)
