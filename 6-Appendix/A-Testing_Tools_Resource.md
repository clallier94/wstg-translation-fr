# Ressource d'outils de test

## Introduction

Cette annexe est destin�e � fournir une liste des outils courants utilis�s pour les tests d'applications Web. Il ne vise pas � �tre une r�f�rence d'outil compl�te, et l'inclusion d'un outil ici ne doit pas �tre consid�r�e comme une approbation sp�cifique de cet outil par l'OWASP.

La liste ne contient que des outils librement disponibles au t�l�chargement et � l'utilisation (bien qu'ils puissent avoir des licences limitant leur utilisation � des fins commerciales).

##�Tests Web g�n�raux

### Proxy Web

- [OWASP ZAP] (https://www.zaproxy.org)
    - Le Zed Attack Proxy (ZAP) est un outil de test de p�n�tration int�gr� facile � utiliser pour trouver des vuln�rabilit�s dans les applications Web. Il est con�u pour �tre utilis� par des personnes ayant une large exp�rience en mati�re de s�curit� et, en tant que tel, il est id�al pour les d�veloppeurs et les testeurs fonctionnels qui d�couvrent les tests d'intrusion.
    - ZAP fournit des scanners automatis�s ainsi qu'un ensemble d'outils qui vous permettent de trouver manuellement les vuln�rabilit�s de s�curit�.
- [Burp Suite Community Edition](https://portswigger.net/burp/communitydownload)
    - Burp Suite est un proxy d'interception pour les tests de s�curit�. Il permet d'intercepter et de modifier tout le trafic HTTP(S) passant dans les deux sens, il peut fonctionner avec des certificats TLS personnalis�s et des clients non sensibles au proxy.
- [Telerik Fiddler](https://www.telerik.com/fiddler)
    - Fiddler un proxy Web d'interception qui s'adresse principalement aux d�veloppeurs plut�t qu'aux testeurs d'intrusion, mais fournit toujours des fonctionnalit�s utiles. Il se connecte �galement directement aux API HTTP de Windows, ce qui lui permet d'intercepter le trafic de certains logiciels qui ne permettent pas de d�finir des proxys personnalis�s.

### Extensions Firefox

- [En-t�te HTTP Firefox Live] (https://addons.mozilla.org/en-US/firefox/addon/http-header-live)
    - Afficher les en-t�tes HTTP d'une page et lors de la navigation.
- [Conteneurs multi-comptes Firefox](https://addons.mozilla.org/en-GB/firefox/addon/multi-account-containers/)
    - Cr�ez plusieurs conteneurs, chacun ayant ses propres cookies et sessions isol�s. Utile pour tester le contr�le d'acc�s entre diff�rents utilisateurs.
- [Firefox Tamper Data](https://addons.mozilla.org/en-US/firefox/addon/tamper-data-for-ff-quantum/)
    - Utilisez Tamper Data pour afficher et modifier les en-t�tes HTTP/HTTPS et les param�tres de publication
- [D�veloppeur Web Firefox](https://addons.mozilla.org/en-US/firefox/addon/web-developer/)
    - L'extension Web Developer ajoute divers outils de d�veloppement Web au navigateur.

### Extensions Chrome

- [D�veloppeur Web Chrome] (https://chrome.google.com/webstore/detail/bfbameneiokkgbdmiekhjnmfkcnldhhm)
    - L'extension Web Developer ajoute un bouton de barre d'outils au navigateur avec divers outils de d�veloppement Web. Il s'agit du portage officiel de l'extension Web Developer pour Chrome.
- [Fabricant de requ�tes HTTP](https://chrome.google.com/webstore/detail/kajfghlhfkcocafkcjlajldicbikpgnp?hl=en-US)
    - Request Maker est un outil de test d'intrusion. Avec lui, vous pouvez facilement capturer les requ�tes faites par les pages Web, falsifier l'URL, les en-t�tes et les donn�es POST et, bien s�r, faire de nouvelles requ�tes
- [�diteur de cookies](https://chrome.google.com/webstore/detail/fngmhnnpilhplaeedifhccceomclgfbg?hl=en-US)
    - Modifier Ce cookie est un gestionnaire de cookies. Vous pouvez ajouter, supprimer, modifier, rechercher, prot�ger et bloquer les cookies

### Test de vuln�rabilit�s sp�cifiques

#### Test d'injection SQL

- [sqlmap](http://sqlmap.org)

#### Test TLS

- [OWASP O-Saft](https://owasp.org/www-project-o-saft/)
- [sslyze](https://github.com/nabla-c0d3/sslyze)
- [testssl.sh](https://github.com/drwetter/testssl.sh)
- [SSLScan](https://github.com/rbsec/sslscan)
- [SSLLabs](https://www.ssllabs.com/ssltest/)

#### Test des attaques par force brute

##### Hash Crackers

- [John the Ripper](https://github.com/openwall/john)
- [hashcat](https://hashcat.net/hashcat/)

##### Force brute � distance

- [OWASP ZAP] (https://www.zaproxy.org)
- [Patator](https://github.com/lanjelot/patator)
- [THC Hydra](https://github.com/vanhauser-thc/thc-hydra)
- [Burp Suite Community Edition (Intrus)](https://portswigger.net/burp/communitydownload)

#### Fuzzers

- [Ffuf](https://github.com/ffuf/ffuf)
- [Wfuzz](https://github.com/xmendez/wfuzz)
- [Jdam](https://gitlab.com/michenriksen/jdam)

#### Piratage de Google

- [Base de donn�es de piratage Google] (https://www.exploit-db.com/google-hacking-database/)

#### HTTP lent

- [Slowloris](https://github.com/gkbrk/slowloris)
- [slowhttptest](https://github.com/shekyan/slowhttptest)

### Mise en miroir du site

- [wget](https://www.gnu.org/software/wget/)
- [wget pour windows](http://gnuwin32.sourceforge.net/packages/wget.htm)
- [curl](https://curl.haxx.se)

### D�couverte de contenu

- [Gobuster](https://github.com/OJ/gobuster)

### D�couverte de ports et de services

- [Nmap](https://nmap.org/)

## Analyseurs de vuln�rabilit�

- [OWASP ZAP] (https://www.zaproxy.org)
- [Nikto](https://cirt.net/Nikto2)
- [Nuclei](https://nuclei.projectdiscovery.io/)

## Cadres d'exploitation

- [Metasploit](https://github.com/rapid7/metasploit-framework)
- [BeEF](https://github.com/beefproject/beef/)

## distributions Linux

- [Kali](https://www.kali.org)
- [Perroquet](https://www.parrotsec.org)
- [Samoura�](https://github.com/SamuraiWTF/samuraiwtf)
- [Santoku](https://sourceforge.net/projects/santoku/)
- [BlackArch](https://blackarch.org/downloads.html)

## Analyseurs de code source

- [Spotbugs](https://spotbugs.github.io)
- [Trouver les bogues de s�curit�] (https://find-sec-bugs.github.io)
- [audit de s�curit� phpcs](https://github.com/squizlabs/PHP_CodeSniffer)
- [PMD](https://pmd.github.io)
- [Analyseurs .NET de Microsoft](https://docs.microsoft.com/en-us/visualstudio/code-quality/install-net-analyzers)
- [�dition communautaire SonarQube] (https://www.sonarqube.org)

## Outils d'automatisation du navigateur

Les outils d'automatisation du navigateur sont utilis�s pour valider la fonctionnalit� des applications Web. Certains suivent une approche script�e et utilisent g�n�ralement un cadre de test unitaire pour construire des suites de tests et des cas de test. La plupart, sinon tous, peuvent �tre adapt�s pour effectuer des tests sp�cifiques � la s�curit� en plus des tests fonctionnels.

### Outils Open�Source

- [HtmlUnit](http://htmlunit.sourceforge.net)
     - Un framework bas� sur Java et JUnit qui utilise Apache HttpClient comme transport.
     - Tr�s robuste et configurable et est utilis� comme moteur pour un certain nombre d'autres outils de test.
- [S�l�nium](https://www.selenium.dev)
     - Framework de test bas� sur JavaScript, multiplateforme et fournit une interface graphique pour cr�er des tests.
