# Ressource d'outils de test

## Introduction

Cette annexe est destinée à fournir une liste des outils courants utilisés pour les tests d'applications Web. Il ne vise pas à être une référence d'outil complète, et l'inclusion d'un outil ici ne doit pas être considérée comme une approbation spécifique de cet outil par l'OWASP.

La liste ne contient que des outils librement disponibles au téléchargement et à l'utilisation (bien qu'ils puissent avoir des licences limitant leur utilisation à des fins commerciales).

## Tests Web généraux

### Proxy Web

- [OWASP ZAP] (https://www.zaproxy.org)
    - Le Zed Attack Proxy (ZAP) est un outil de test de pénétration intégré facile à utiliser pour trouver des vulnérabilités dans les applications Web. Il est conçu pour être utilisé par des personnes ayant une large expérience en matière de sécurité et, en tant que tel, il est idéal pour les développeurs et les testeurs fonctionnels qui découvrent les tests d'intrusion.
    - ZAP fournit des scanners automatisés ainsi qu'un ensemble d'outils qui vous permettent de trouver manuellement les vulnérabilités de sécurité.
- [Burp Suite Community Edition](https://portswigger.net/burp/communitydownload)
    - Burp Suite est un proxy d'interception pour les tests de sécurité. Il permet d'intercepter et de modifier tout le trafic HTTP(S) passant dans les deux sens, il peut fonctionner avec des certificats TLS personnalisés et des clients non sensibles au proxy.
- [Telerik Fiddler](https://www.telerik.com/fiddler)
    - Fiddler un proxy Web d'interception qui s'adresse principalement aux développeurs plutôt qu'aux testeurs d'intrusion, mais fournit toujours des fonctionnalités utiles. Il se connecte également directement aux API HTTP de Windows, ce qui lui permet d'intercepter le trafic de certains logiciels qui ne permettent pas de définir des proxys personnalisés.

### Extensions Firefox

- [En-tête HTTP Firefox Live] (https://addons.mozilla.org/en-US/firefox/addon/http-header-live)
    - Afficher les en-têtes HTTP d'une page et lors de la navigation.
- [Conteneurs multi-comptes Firefox](https://addons.mozilla.org/en-GB/firefox/addon/multi-account-containers/)
    - Créez plusieurs conteneurs, chacun ayant ses propres cookies et sessions isolés. Utile pour tester le contrôle d'accès entre différents utilisateurs.
- [Firefox Tamper Data](https://addons.mozilla.org/en-US/firefox/addon/tamper-data-for-ff-quantum/)
    - Utilisez Tamper Data pour afficher et modifier les en-têtes HTTP/HTTPS et les paramètres de publication
- [Développeur Web Firefox](https://addons.mozilla.org/en-US/firefox/addon/web-developer/)
    - L'extension Web Developer ajoute divers outils de développement Web au navigateur.

### Extensions Chrome

- [Développeur Web Chrome] (https://chrome.google.com/webstore/detail/bfbameneiokkgbdmiekhjnmfkcnldhhm)
    - L'extension Web Developer ajoute un bouton de barre d'outils au navigateur avec divers outils de développement Web. Il s'agit du portage officiel de l'extension Web Developer pour Chrome.
- [Fabricant de requêtes HTTP](https://chrome.google.com/webstore/detail/kajfghlhfkcocafkcjlajldicbikpgnp?hl=en-US)
    - Request Maker est un outil de test d'intrusion. Avec lui, vous pouvez facilement capturer les requêtes faites par les pages Web, falsifier l'URL, les en-têtes et les données POST et, bien sûr, faire de nouvelles requêtes
- [Éditeur de cookies](https://chrome.google.com/webstore/detail/fngmhnnpilhplaeedifhccceomclgfbg?hl=en-US)
    - Modifier Ce cookie est un gestionnaire de cookies. Vous pouvez ajouter, supprimer, modifier, rechercher, protéger et bloquer les cookies

### Test de vulnérabilités spécifiques

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

##### Force brute à distance

- [OWASP ZAP] (https://www.zaproxy.org)
- [Patator](https://github.com/lanjelot/patator)
- [THC Hydra](https://github.com/vanhauser-thc/thc-hydra)
- [Burp Suite Community Edition (Intrus)](https://portswigger.net/burp/communitydownload)

#### Fuzzers

- [Ffuf](https://github.com/ffuf/ffuf)
- [Wfuzz](https://github.com/xmendez/wfuzz)
- [Jdam](https://gitlab.com/michenriksen/jdam)

#### Piratage de Google

- [Base de données de piratage Google] (https://www.exploit-db.com/google-hacking-database/)

#### HTTP lent

- [Slowloris](https://github.com/gkbrk/slowloris)
- [slowhttptest](https://github.com/shekyan/slowhttptest)

### Mise en miroir du site

- [wget](https://www.gnu.org/software/wget/)
- [wget pour windows](http://gnuwin32.sourceforge.net/packages/wget.htm)
- [curl](https://curl.haxx.se)

### Découverte de contenu

- [Gobuster](https://github.com/OJ/gobuster)

### Découverte de ports et de services

- [Nmap](https://nmap.org/)

## Analyseurs de vulnérabilité

- [OWASP ZAP] (https://www.zaproxy.org)
- [Nikto](https://cirt.net/Nikto2)
- [Nuclei](https://nuclei.projectdiscovery.io/)

## Cadres d'exploitation

- [Metasploit](https://github.com/rapid7/metasploit-framework)
- [BeEF](https://github.com/beefproject/beef/)

## distributions Linux

- [Kali](https://www.kali.org)
- [Perroquet](https://www.parrotsec.org)
- [Samouraï](https://github.com/SamuraiWTF/samuraiwtf)
- [Santoku](https://sourceforge.net/projects/santoku/)
- [BlackArch](https://blackarch.org/downloads.html)

## Analyseurs de code source

- [Spotbugs](https://spotbugs.github.io)
- [Trouver les bogues de sécurité] (https://find-sec-bugs.github.io)
- [audit de sécurité phpcs](https://github.com/squizlabs/PHP_CodeSniffer)
- [PMD](https://pmd.github.io)
- [Analyseurs .NET de Microsoft](https://docs.microsoft.com/en-us/visualstudio/code-quality/install-net-analyzers)
- [Édition communautaire SonarQube] (https://www.sonarqube.org)

## Outils d'automatisation du navigateur

Les outils d'automatisation du navigateur sont utilisés pour valider la fonctionnalité des applications Web. Certains suivent une approche scriptée et utilisent généralement un cadre de test unitaire pour construire des suites de tests et des cas de test. La plupart, sinon tous, peuvent être adaptés pour effectuer des tests spécifiques à la sécurité en plus des tests fonctionnels.

### Outils Open Source

- [HtmlUnit](http://htmlunit.sourceforge.net)
     - Un framework basé sur Java et JUnit qui utilise Apache HttpClient comme transport.
     - Très robuste et configurable et est utilisé comme moteur pour un certain nombre d'autres outils de test.
- [Sélénium](https://www.selenium.dev)
     - Framework de test basé sur JavaScript, multiplateforme et fournit une interface graphique pour créer des tests.
