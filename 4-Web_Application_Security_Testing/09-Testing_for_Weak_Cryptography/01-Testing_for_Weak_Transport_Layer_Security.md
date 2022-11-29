# Test de s�curit� de la couche de transport faible

|ID          |
|------------|
|WSTG-CRYP-01|

## Sommaire

Lorsque des informations sont transmises entre le client et le serveur, elles doivent �tre chiffr�es et prot�g�es afin d'emp�cher qu'un attaquant ne puisse les lire ou les modifier. Cela se fait le plus souvent � l'aide de HTTPS, qui utilise le protocole [Transport Layer Security (TLS)](https://en.wikipedia.org/wiki/Transport_Layer_Security), qui remplace l'ancien protocole Secure Socket Layer (SSL). TLS permet �galement au serveur de d�montrer au client qu'il s'est connect� au bon serveur, en pr�sentant un certificat num�rique de confiance.

Au fil des ans, un grand nombre de faiblesses cryptographiques ont �t� identifi�es dans les protocoles SSL et TLS, ainsi que dans les chiffrements qu'ils utilisent. De plus, de nombreuses impl�mentations de ces protocoles pr�sentent �galement de graves vuln�rabilit�s. En tant que tel, il est important de tester que les sites impl�mentent non seulement TLS, mais qu'ils le font de mani�re s�curis�e.

## Objectifs des tests

- Valider la configuration du service.
- V�rifiez la force cryptographique et la validit� du certificat num�rique.
- Assurez-vous que la s�curit� TLS ne peut pas �tre contourn�e et qu'elle est correctement impl�ment�e dans l'application.

## Comment tester

Les probl�mes li�s � la s�curit� de la couche de transport peuvent �tre largement r�partis dans les domaines suivants�:

###�Configuration du serveur

Il existe un grand nombre de versions de protocole, de chiffrements et d'extensions pris en charge par TLS. Beaucoup d'entre eux sont consid�r�s comme h�rit�s et pr�sentent des faiblesses cryptographiques, telles que celles r�pertori�es ci-dessous. Notez que de nouvelles faiblesses sont susceptibles d'�tre identifi�es au fil du temps, cette liste peut donc �tre incompl�te.

- [SSLv2 (noyer)](https://drownattack.com/)
- [SSLv3 (CANICHE)](https://en.wikipedia.org/wiki/POODLE)
- [TLSv1.0 (BEAST)](https://www.acunetix.com/blog/web-security-zone/what-is-beast-attack/)
- [TLSv1.1 (obsol�te par RFC 8996)](https://tools.ietf.org/html/rfc8996)
- [EXPORTER des suites de chiffrement (FREAK)](https://en.wikipedia.org/wiki/FREAK)
- [NULL ciphers](https://www.rapid7.com/db/vulnerabilities/ssl-null-ciphers) ([ils ne fournissent qu'une authentification](https://tools.ietf.org/html/rfc4785)).
- Chiffrements anonymes (ceux-ci peuvent �tre pris en charge sur les serveurs SMTP, comme indiqu� dans la [RFC 7672](https://tools.ietf.org/html/rfc7672#section-8.2))
- [chiffres RC4 (NOMORE)](https://www.rc4nomore.com/)
- Chiffrements en mode CBC (BEAST, [Lucky 13](https://en.wikipedia.org/wiki/Lucky_Thirteen_attack))
- [Compression TLS (CRIME)](https://en.wikipedia.org/wiki/CRIME)
- [Cl�s DHE faibles (LOGJAM)](https://weakdh.org/)

Le [Guide Mozilla Server Side TLS](https://wiki.mozilla.org/Security/Server_Side_TLS) d�taille les protocoles et les chiffrements actuellement recommand�s.

#### Exploitabilit�

Il convient de souligner que m�me si bon nombre de ces attaques ont �t� d�montr�es dans un environnement de laboratoire, elles ne sont g�n�ralement pas consid�r�es comme pratiques � exploiter dans le monde r�el, car elles n�cessitent une attaque MitM (g�n�ralement active) et des ressources importantes. En tant que tels, il est peu probable qu'ils soient exploit�s par quiconque autre que les �tats-nations.

###�Certificats num�riques

#### Faiblesses cryptographiques

D'un point de vue cryptographique, il y a deux domaines principaux qui doivent �tre examin�s sur un certificat num�rique�:

- La force de la cl� doit �tre *au moins* 2048 bits.
- L'algorithme de signature doit �tre *au moins* SHA-256. Les algorithmes h�rit�s tels que MD5 et SHA-1 ne doivent pas �tre utilis�s.

#### Validit�

En plus d'�tre cryptographiquement s�curis�, le certificat doit �galement �tre consid�r� comme valide (ou de confiance). Cela signifie qu'il doit :

- �tre dans la p�riode de validit� d�finie.
    - Tout certificat �mis apr�s le 1er septembre 2020 ne doit pas avoir une dur�e de vie maximale de plus de [398 jours](https://blog.mozilla.org/security/2020/07/09/reducing-tls-certificate-lifespans-to- 398 jours/).
- �tre sign� par une autorit� de certification (CA) de confiance.
    - Il doit s'agir soit d'une autorit� de certification publique de confiance pour les applications externes, soit d'une autorit� de certification interne pour les applications internes.
    - Ne signalez pas les applications internes comme ayant des certificats non fiables simplement parce que *votre* syst�me ne fait pas confiance � l'autorit� de certification.
- Avoir un autre nom d'objet (SAN) qui correspond au nom d'h�te du syst�me.
    - Le champ Common Name (CN) est ignor� par les navigateurs modernes, qui ne regardent que le SAN.
    - Assurez-vous que vous acc�dez au syst�me avec le nom correct (par exemple, si vous acc�dez � l'h�te par IP, tout certificat appara�tra comme non approuv�).

Certains certificats peuvent �tre �mis pour des domaines g�n�riques (tels que `*.example.org`), ce qui signifie qu'ils peuvent �tre valides pour plusieurs sous-domaines. Bien que pratique, il existe un certain nombre de probl�mes de s�curit� qui doivent �tre pris en compte. Ceux-ci sont discut�s dans le [OWASP Transport Layer Security Cheat Sheet](https://cheatsheetseries.owasp.org/cheatsheets/Transport_Layer_Protection_Cheat_Sheet.html#carefully-consider-the-use-of-wildcard-certificates).

Les certificats peuvent �galement divulguer des informations sur les syst�mes internes ou les noms de domaine dans les champs �metteur et SAN, ce qui peut �tre utile lorsque vous essayez de cr�er une image du r�seau interne ou de mener des activit�s d'ing�nierie sociale.

### Vuln�rabilit�s d'impl�mentation

Au fil des ans, il y a eu des vuln�rabilit�s dans les diff�rentes impl�mentations TLS. Il y en a trop pour les �num�rer ici, mais certains des exemples cl�s sont :

- [G�n�rateur de nombres al�atoires pr�visibles Debian OpenSSL](https://www.debian.org/security/2008/dsa-1571) (CVE-2008-0166)
- [Ren�gociation non s�curis�e OpenSSL](https://www.openssl.org/news/secadv/20091111.txt) (CVE-2009-3555)
- [OpenSSL Heartbleed](https://heartbleed.com) (CVE-2014-0160)
- [F5 TLS CANICHE](https://support.f5.com/csp/article/K15882) (CVE-2014-8730)
- [D�ni de service Microsoft Schannel] (https://docs.microsoft.com/en-us/security-updates/securitybulletins/2014/ms14-066) (MS14-066 / CVE-2014-6321)

### Vuln�rabilit�s des applications

Outre la configuration s�curis�e de la configuration TLS sous-jacente, l'application doit �galement l'utiliser de mani�re s�curis�e. Certains de ces points sont trait�s ailleurs dans ce guide :

- [Ne pas envoyer de donn�es sensibles sur des canaux non chiffr�s (WSTG-CRYP-03)](03-Testing_for_Sensitive_Information_Sent_via_Unencrypted_Channels.md)
- [D�finition de l'en-t�te HTTP Strict-Transport-Security (WSTG-CONF-07)](../02-Configuration_and_Deployment_Management_Testing/07-Test_HTTP_Strict_Transport_Security.md)
- [D�finition de l'indicateur de s�curit� sur les cookies (WSTG-SESS-02)](../06-Session_Management_Testing/02-Testing_for_Cookies_Attributes.md)

#### Contenu actif mixte

Le contenu actif mixte se produit lorsque des ressources actives (telles que des scripts vers CSS) sont charg�es via HTTP non chiffr� et incluses dans une page s�curis�e (HTTPS). Ceci est dangereux car cela permettrait � un attaquant de modifier ces fichiers (car ils sont envoy�s en clair), ce qui pourrait lui permettre d'ex�cuter du code arbitraire (JavaScript ou CSS) dans la page. Le contenu passif (tel que les images) charg� via une connexion non s�curis�e peut �galement divulguer des informations ou permettre � un attaquant de d�figurer la page, bien qu'il soit moins susceptible de conduire � un compromis complet.

> Remarque : les navigateurs modernes bloqueront le chargement du contenu actif � partir de sources non s�curis�es vers des pages s�curis�es.

#### Redirection de HTTP vers HTTPS

De nombreux sites acceptent les connexions via HTTP non crypt�, puis redirigent imm�diatement l'utilisateur vers la version s�curis�e (HTTPS) du site avec une redirection "301 Moved Permanently". La version HTTPS du site d�finit ensuite l'en-t�te "Strict-Transport-Security" pour demander au navigateur de toujours utiliser HTTPS � l'avenir.

Cependant, si un attaquant parvient � intercepter cette requ�te initiale, il pourrait rediriger l'utilisateur vers un site malveillant ou utiliser un outil tel que [sslstrip](https://github.com/moxie0/sslstrip) pour intercepter les requ�tes suivantes.

Afin de se d�fendre contre ce type d'attaque, le site doit �tre ajout� � la [liste de pr�chargement] (https://hstspreload.org).

##�Tests automatis�s

Il existe un grand nombre d'outils d'analyse qui peuvent �tre utilis�s pour identifier les faiblesses de la configuration SSL/TLS d'un service, y compris des outils d�di�s et des analyseurs de vuln�rabilit� � usage g�n�ral. Certains des plus populaires sont :

- [Nmap](https://nmap.org) (divers scripts)
- [OWASP O-Saft](https://owasp.org/www-project-o-saft/)
- [sslscan](https://github.com/rbsec/sslscan)
- [sslyze](https://github.com/nabla-c0d3/sslyze)
- [Laboratoires SSL](https://www.ssllabs.com/ssltest/)
- [testssl.sh](https://github.com/drwetter/testssl.sh)

###�Tests manuels

Il est �galement possible d'effectuer la plupart des v�rifications manuellement, en utilisant des lignes de commande telles que `openssl s_client` ou `gnutls-cli` pour se connecter avec des protocoles, des chiffrements ou des options sp�cifiques.

Lors de tests de ce type, sachez que la version d'OpenSSL ou de GnuTLS fournie avec la plupart des syst�mes modernes peut ne pas prendre en charge certains protocoles obsol�tes et non s�curis�s tels que les chiffrements SSLv2 ou EXPORT. Assurez-vous que votre version prend en charge les versions obsol�tes avant de l'utiliser pour les tests, sinon vous vous retrouverez avec de faux n�gatifs.

Il peut �galement �tre possible d'effectuer des tests limit�s � l'aide d'un navigateur Web, car les navigateurs modernes fourniront des d�tails sur les protocoles et les chiffrements utilis�s dans leurs outils de d�veloppement. Ils fournissent �galement un moyen simple de tester si un certificat est consid�r� comme fiable, en acc�dant au service et en voyant si un avertissement de certificat vous est pr�sent�.

## R�f�rences

- [Aide-m�moire sur la protection de la couche de transport OWASP] (https://cheatsheetseries.owasp.org/cheatsheets/Transport_Layer_Protection_Cheat_Sheet.html)
- [Guide TLS c�t� serveur Mozilla] (https://wiki.mozilla.org/Security/Server_Side_TLS)
