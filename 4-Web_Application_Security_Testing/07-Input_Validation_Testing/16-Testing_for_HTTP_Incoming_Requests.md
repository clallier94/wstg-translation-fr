# Test des requêtes HTTP entrantes

|ID          |
|------------|
|WSTG-INPV-16|

## Sommaire

Cette section décrit comment surveiller toutes les requêtes HTTP entrantes/sortantes côté client ou côté serveur. Le but de ce test est de vérifier s'il y a un envoi de requête HTTP inutile ou suspect en arrière-plan.

La plupart des outils de test de sécurité Web (c'est-à-dire AppScan, BurpSuite, ZAP) agissent comme proxy HTTP. Cela nécessitera des changements de proxy sur l'application ou le navigateur côté client. Les techniques de test répertoriées ci-dessous sont principalement axées sur la manière dont nous pouvons surveiller les requêtes HTTP sans modification du côté client, ce qui sera plus proche du scénario d'utilisation en production.

## Objectifs des tests

- Surveillez toutes les requêtes HTTP entrantes et sortantes vers le serveur Web pour inspecter toutes les requêtes suspectes.
- Surveillez le trafic HTTP sans modification du proxy du navigateur de l'utilisateur final ou de l'application côté client.

## Comment tester

### Proxy inverse

Il existe une situation dans laquelle nous aimerions surveiller toutes les requêtes HTTP entrantes sur le serveur Web, mais nous ne pouvons pas modifier la configuration côté navigateur ou côté client de l'application. Dans ce scénario, nous pouvons configurer un proxy inverse sur le serveur Web pour surveiller toutes les demandes entrantes/sortantes sur le serveur Web.

Pour la plate-forme Windows, Fiddler est recommandé. Il fournit non seulement un moniteur, mais peut également modifier/répondre aux requêtes HTTP. Reportez-vous à [cette référence pour savoir comment configurer Fiddler en tant que proxy inverse](http://docs.telerik.com/fiddler/Configure-Fiddler/Tasks/UseFiddlerAsReverseProxy)

Pour la plate-forme Linux, Charles Web Debugging Proxy peut être utilisé.

Les étapes de test :

1. Installez Fiddler ou Charles sur le serveur Web
2. Configurez le Fiddler ou Charles en tant que proxy inverse
3. Capturez le trafic HTTP
4. Inspectez le trafic HTTP
5. Modifier les requêtes HTTP et rejouer les requêtes modifiées pour les tests

### Redirection de port

Port forwarding is another way to allow us intercept HTTP requests without changes of client-side. You can also use Charles as a SOCKS proxy to act as port forwarding or uses of Port Forwarding tools. It will allow us to forward all coming client-side captured traffic to web server port.

Le flux de test sera :

1. Installez le Charles ou la redirection de port sur une autre machine ou un serveur Web
2. Configurez le proxy Charles en tant que chaussettes en tant que redirection de port.

### Capture du trafic réseau au niveau TCP

Cette technique surveille tout le trafic réseau au niveau TCP. Les outils TCPDump ou WireShark peuvent être utilisés. Cependant, ces outils ne nous permettent pas de modifier le trafic capturé et d'envoyer des requêtes HTTP modifiées à des fins de test. Pour rejouer les paquets de trafic capturés (PCAP), Ostinato peut être utilisé.

Les étapes de test seront :

1. Activez TCPDump ou WireShark sur le serveur Web pour capturer le trafic réseau
2. Surveiller les fichiers capturés (PCAP)
3. Modifier les fichiers PCAP par l'outil Ostinato en fonction des besoins
4. Répondre aux requêtes HTTP

Fiddler ou Charles sont recommandés car ces outils peuvent capturer le trafic HTTP et également modifier/répondre facilement aux requêtes HTTP modifiées. De plus, si le trafic Web est HTTPS, le wireshark devra importer la clé privée du serveur Web pour inspecter le corps du message HTTPS. Sinon, le corps du message HTTPS du trafic capturé sera entièrement chiffré.

## Outils

- [Fiddler](https://www.telerik.com/fiddler/)
- [TCPProxy](http://grinder.sourceforge.net/g3/tcpproxy.html)
- [Proxy de débogage Web Charles] (https://www.charlesproxy.com/)
- [WireShark](https://www.wireshark.org/)
- [PowerEdit-Pcap](https://sourceforge.net/projects/powereditpcap/)
- [pcapteller](https://github.com/BlackArch/pcapteller)
- [replayproxy](https://github.com/sparrowt/replayproxy)
- [Ostinato](https://ostinato.org/)

## Références

- [Proxy de débogage Web Charles] (https://www.charlesproxy.com/)
- [Fiddler](https://www.telerik.com/fiddler/)
- [TCPDUMP](https://www.tcpdump.org/)
- [Ostinato](https://ostinato.org/)
