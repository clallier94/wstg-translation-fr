# Test de sécurité de la couche de transport faible

|ID          |
|------------|
|WSTG-CRYP-01|

## Sommaire

Lorsque des informations sont transmises entre le client et le serveur, elles doivent être chiffrées et protégées afin d'empêcher qu'un attaquant ne puisse les lire ou les modifier. Cela se fait le plus souvent à l'aide de HTTPS, qui utilise le protocole [Transport Layer Security (TLS)](https://en.wikipedia.org/wiki/Transport_Layer_Security), qui remplace l'ancien protocole Secure Socket Layer (SSL). TLS permet également au serveur de démontrer au client qu'il s'est connecté au bon serveur, en présentant un certificat numérique de confiance.

Au fil des ans, un grand nombre de faiblesses cryptographiques ont été identifiées dans les protocoles SSL et TLS, ainsi que dans les chiffrements qu'ils utilisent. De plus, de nombreuses implémentations de ces protocoles présentent également de graves vulnérabilités. En tant que tel, il est important de tester que les sites implémentent non seulement TLS, mais qu'ils le font de manière sécurisée.

## Objectifs des tests

- Valider la configuration du service.
- Vérifiez la force cryptographique et la validité du certificat numérique.
- Assurez-vous que la sécurité TLS ne peut pas être contournée et qu'elle est correctement implémentée dans l'application.

## Comment tester

Les problèmes liés à la sécurité de la couche de transport peuvent être largement répartis dans les domaines suivants :

### Configuration du serveur

Il existe un grand nombre de versions de protocole, de chiffrements et d'extensions pris en charge par TLS. Beaucoup d'entre eux sont considérés comme hérités et présentent des faiblesses cryptographiques, telles que celles répertoriées ci-dessous. Notez que de nouvelles faiblesses sont susceptibles d'être identifiées au fil du temps, cette liste peut donc être incomplète.

- [SSLv2 (noyer)](https://drownattack.com/)
- [SSLv3 (CANICHE)](https://en.wikipedia.org/wiki/POODLE)
- [TLSv1.0 (BEAST)](https://www.acunetix.com/blog/web-security-zone/what-is-beast-attack/)
- [TLSv1.1 (obsolète par RFC 8996)](https://tools.ietf.org/html/rfc8996)
- [EXPORTER des suites de chiffrement (FREAK)](https://en.wikipedia.org/wiki/FREAK)
- [NULL ciphers](https://www.rapid7.com/db/vulnerabilities/ssl-null-ciphers) ([ils ne fournissent qu'une authentification](https://tools.ietf.org/html/rfc4785)).
- Chiffrements anonymes (ceux-ci peuvent être pris en charge sur les serveurs SMTP, comme indiqué dans la [RFC 7672](https://tools.ietf.org/html/rfc7672#section-8.2))
- [chiffres RC4 (NOMORE)](https://www.rc4nomore.com/)
- Chiffrements en mode CBC (BEAST, [Lucky 13](https://en.wikipedia.org/wiki/Lucky_Thirteen_attack))
- [Compression TLS (CRIME)](https://en.wikipedia.org/wiki/CRIME)
- [Clés DHE faibles (LOGJAM)](https://weakdh.org/)

Le [Guide Mozilla Server Side TLS](https://wiki.mozilla.org/Security/Server_Side_TLS) détaille les protocoles et les chiffrements actuellement recommandés.

#### Exploitabilité

Il convient de souligner que même si bon nombre de ces attaques ont été démontrées dans un environnement de laboratoire, elles ne sont généralement pas considérées comme pratiques à exploiter dans le monde réel, car elles nécessitent une attaque MitM (généralement active) et des ressources importantes. En tant que tels, il est peu probable qu'ils soient exploités par quiconque autre que les États-nations.

### Certificats numériques

#### Faiblesses cryptographiques

D'un point de vue cryptographique, il y a deux domaines principaux qui doivent être examinés sur un certificat numérique :

- La force de la clé doit être *au moins* 2048 bits.
- L'algorithme de signature doit être *au moins* SHA-256. Les algorithmes hérités tels que MD5 et SHA-1 ne doivent pas être utilisés.

#### Validité

En plus d'être cryptographiquement sécurisé, le certificat doit également être considéré comme valide (ou de confiance). Cela signifie qu'il doit :

- Être dans la période de validité définie.
    - Tout certificat émis après le 1er septembre 2020 ne doit pas avoir une durée de vie maximale de plus de [398 jours](https://blog.mozilla.org/security/2020/07/09/reducing-tls-certificate-lifespans-to- 398 jours/).
- Être signé par une autorité de certification (CA) de confiance.
    - Il doit s'agir soit d'une autorité de certification publique de confiance pour les applications externes, soit d'une autorité de certification interne pour les applications internes.
    - Ne signalez pas les applications internes comme ayant des certificats non fiables simplement parce que *votre* système ne fait pas confiance à l'autorité de certification.
- Avoir un autre nom d'objet (SAN) qui correspond au nom d'hôte du système.
    - Le champ Common Name (CN) est ignoré par les navigateurs modernes, qui ne regardent que le SAN.
    - Assurez-vous que vous accédez au système avec le nom correct (par exemple, si vous accédez à l'hôte par IP, tout certificat apparaîtra comme non approuvé).

Certains certificats peuvent être émis pour des domaines génériques (tels que `*.example.org`), ce qui signifie qu'ils peuvent être valides pour plusieurs sous-domaines. Bien que pratique, il existe un certain nombre de problèmes de sécurité qui doivent être pris en compte. Ceux-ci sont discutés dans le [OWASP Transport Layer Security Cheat Sheet](https://cheatsheetseries.owasp.org/cheatsheets/Transport_Layer_Protection_Cheat_Sheet.html#carefully-consider-the-use-of-wildcard-certificates).

Les certificats peuvent également divulguer des informations sur les systèmes internes ou les noms de domaine dans les champs Émetteur et SAN, ce qui peut être utile lorsque vous essayez de créer une image du réseau interne ou de mener des activités d'ingénierie sociale.

### Vulnérabilités d'implémentation

Au fil des ans, il y a eu des vulnérabilités dans les différentes implémentations TLS. Il y en a trop pour les énumérer ici, mais certains des exemples clés sont :

- [Générateur de nombres aléatoires prévisibles Debian OpenSSL](https://www.debian.org/security/2008/dsa-1571) (CVE-2008-0166)
- [Renégociation non sécurisée OpenSSL](https://www.openssl.org/news/secadv/20091111.txt) (CVE-2009-3555)
- [OpenSSL Heartbleed](https://heartbleed.com) (CVE-2014-0160)
- [F5 TLS CANICHE](https://support.f5.com/csp/article/K15882) (CVE-2014-8730)
- [Déni de service Microsoft Schannel] (https://docs.microsoft.com/en-us/security-updates/securitybulletins/2014/ms14-066) (MS14-066 / CVE-2014-6321)

### Vulnérabilités des applications

Outre la configuration sécurisée de la configuration TLS sous-jacente, l'application doit également l'utiliser de manière sécurisée. Certains de ces points sont traités ailleurs dans ce guide :

- [Ne pas envoyer de données sensibles sur des canaux non chiffrés (WSTG-CRYP-03)](03-Testing_for_Sensitive_Information_Sent_via_Unencrypted_Channels.md)
- [Définition de l'en-tête HTTP Strict-Transport-Security (WSTG-CONF-07)](../02-Configuration_and_Deployment_Management_Testing/07-Test_HTTP_Strict_Transport_Security.md)
- [Définition de l'indicateur de sécurité sur les cookies (WSTG-SESS-02)](../06-Session_Management_Testing/02-Testing_for_Cookies_Attributes.md)

#### Contenu actif mixte

Le contenu actif mixte se produit lorsque des ressources actives (telles que des scripts vers CSS) sont chargées via HTTP non chiffré et incluses dans une page sécurisée (HTTPS). Ceci est dangereux car cela permettrait à un attaquant de modifier ces fichiers (car ils sont envoyés en clair), ce qui pourrait lui permettre d'exécuter du code arbitraire (JavaScript ou CSS) dans la page. Le contenu passif (tel que les images) chargé via une connexion non sécurisée peut également divulguer des informations ou permettre à un attaquant de défigurer la page, bien qu'il soit moins susceptible de conduire à un compromis complet.

> Remarque : les navigateurs modernes bloqueront le chargement du contenu actif à partir de sources non sécurisées vers des pages sécurisées.

#### Redirection de HTTP vers HTTPS

De nombreux sites acceptent les connexions via HTTP non crypté, puis redirigent immédiatement l'utilisateur vers la version sécurisée (HTTPS) du site avec une redirection "301 Moved Permanently". La version HTTPS du site définit ensuite l'en-tête "Strict-Transport-Security" pour demander au navigateur de toujours utiliser HTTPS à l'avenir.

Cependant, si un attaquant parvient à intercepter cette requête initiale, il pourrait rediriger l'utilisateur vers un site malveillant ou utiliser un outil tel que [sslstrip](https://github.com/moxie0/sslstrip) pour intercepter les requêtes suivantes.

Afin de se défendre contre ce type d'attaque, le site doit être ajouté à la [liste de préchargement] (https://hstspreload.org).

## Tests automatisés

Il existe un grand nombre d'outils d'analyse qui peuvent être utilisés pour identifier les faiblesses de la configuration SSL/TLS d'un service, y compris des outils dédiés et des analyseurs de vulnérabilité à usage général. Certains des plus populaires sont :

- [Nmap](https://nmap.org) (divers scripts)
- [OWASP O-Saft](https://owasp.org/www-project-o-saft/)
- [sslscan](https://github.com/rbsec/sslscan)
- [sslyze](https://github.com/nabla-c0d3/sslyze)
- [Laboratoires SSL](https://www.ssllabs.com/ssltest/)
- [testssl.sh](https://github.com/drwetter/testssl.sh)

### Tests manuels

Il est également possible d'effectuer la plupart des vérifications manuellement, en utilisant des lignes de commande telles que `openssl s_client` ou `gnutls-cli` pour se connecter avec des protocoles, des chiffrements ou des options spécifiques.

Lors de tests de ce type, sachez que la version d'OpenSSL ou de GnuTLS fournie avec la plupart des systèmes modernes peut ne pas prendre en charge certains protocoles obsolètes et non sécurisés tels que les chiffrements SSLv2 ou EXPORT. Assurez-vous que votre version prend en charge les versions obsolètes avant de l'utiliser pour les tests, sinon vous vous retrouverez avec de faux négatifs.

Il peut également être possible d'effectuer des tests limités à l'aide d'un navigateur Web, car les navigateurs modernes fourniront des détails sur les protocoles et les chiffrements utilisés dans leurs outils de développement. Ils fournissent également un moyen simple de tester si un certificat est considéré comme fiable, en accédant au service et en voyant si un avertissement de certificat vous est présenté.

## Références

- [Aide-mémoire sur la protection de la couche de transport OWASP] (https://cheatsheetseries.owasp.org/cheatsheets/Transport_Layer_Protection_Cheat_Sheet.html)
- [Guide TLS côté serveur Mozilla] (https://wiki.mozilla.org/Security/Server_Side_TLS)
