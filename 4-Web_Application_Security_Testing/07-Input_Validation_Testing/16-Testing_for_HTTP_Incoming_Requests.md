# Test des requ�tes HTTP entrantes

|ID          |
|------------|
|WSTG-INPV-16|

## Sommaire

Cette section d�crit comment surveiller toutes les requ�tes HTTP entrantes/sortantes c�t� client ou c�t� serveur. Le but de ce test est de v�rifier s'il y a un envoi de requ�te HTTP inutile ou suspect en arri�re-plan.

La plupart des outils de test de s�curit� Web (c'est-�-dire AppScan, BurpSuite, ZAP) agissent comme proxy HTTP. Cela n�cessitera des changements de proxy sur l'application ou le navigateur c�t� client. Les techniques de test r�pertori�es ci-dessous sont principalement ax�es sur la mani�re dont nous pouvons surveiller les requ�tes HTTP sans modification du c�t� client, ce qui sera plus proche du sc�nario d'utilisation en production.

## Objectifs des tests

- Surveillez toutes les requ�tes HTTP entrantes et sortantes vers le serveur Web pour inspecter toutes les requ�tes suspectes.
- Surveillez le trafic HTTP sans modification du proxy du navigateur de l'utilisateur final ou de l'application c�t� client.

## Comment tester

### Proxy inverse

Il existe une situation dans laquelle nous aimerions surveiller toutes les requ�tes HTTP entrantes sur le serveur Web, mais nous ne pouvons pas modifier la configuration c�t� navigateur ou c�t� client de l'application. Dans ce sc�nario, nous pouvons configurer un proxy inverse sur le serveur Web pour surveiller toutes les demandes entrantes/sortantes sur le serveur Web.

Pour la plate-forme Windows, Fiddler est recommand�. Il fournit non seulement un moniteur, mais peut �galement modifier/r�pondre aux requ�tes HTTP. Reportez-vous � [cette r�f�rence pour savoir comment configurer Fiddler en tant que proxy inverse](http://docs.telerik.com/fiddler/Configure-Fiddler/Tasks/UseFiddlerAsReverseProxy)

Pour la plate-forme Linux, Charles Web Debugging Proxy peut �tre utilis�.

Les �tapes de test :

1. Installez Fiddler ou Charles sur le serveur Web
2. Configurez le Fiddler ou Charles en tant que proxy inverse
3. Capturez le trafic HTTP
4. Inspectez le trafic HTTP
5. Modifier les requ�tes HTTP et rejouer les requ�tes modifi�es pour les tests

### Redirection de port

Port forwarding is another way to allow us intercept HTTP requests without changes of client-side. You can also use Charles as a SOCKS proxy to act as port forwarding or uses of Port Forwarding tools. It will allow us to forward all coming client-side captured traffic to web server port.

Le flux de test sera�:

1. Installez le Charles ou la redirection de port sur une autre machine ou un serveur Web
2. Configurez le proxy Charles en tant que chaussettes en tant que redirection de port.

### Capture du trafic r�seau au niveau TCP

Cette technique surveille tout le trafic r�seau au niveau TCP. Les outils TCPDump ou WireShark peuvent �tre utilis�s. Cependant, ces outils ne nous permettent pas de modifier le trafic captur� et d'envoyer des requ�tes HTTP modifi�es � des fins de test. Pour rejouer les paquets de trafic captur�s (PCAP), Ostinato peut �tre utilis�.

Les �tapes de test seront :

1. Activez TCPDump ou WireShark sur le serveur Web pour capturer le trafic r�seau
2. Surveiller les fichiers captur�s (PCAP)
3. Modifier les fichiers PCAP par l'outil Ostinato en fonction des besoins
4. R�pondre aux requ�tes HTTP

Fiddler ou Charles sont recommand�s car ces outils peuvent capturer le trafic HTTP et �galement modifier/r�pondre facilement aux requ�tes HTTP modifi�es. De plus, si le trafic Web est HTTPS, le wireshark devra importer la cl� priv�e du serveur Web pour inspecter le corps du message HTTPS. Sinon, le corps du message HTTPS du trafic captur� sera enti�rement chiffr�.

## Outils

- [Fiddler](https://www.telerik.com/fiddler/)
- [TCPProxy](http://grinder.sourceforge.net/g3/tcpproxy.html)
- [Proxy de d�bogage Web Charles] (https://www.charlesproxy.com/)
- [WireShark](https://www.wireshark.org/)
- [PowerEdit-Pcap](https://sourceforge.net/projects/powereditpcap/)
- [pcapteller](https://github.com/BlackArch/pcapteller)
- [replayproxy](https://github.com/sparrowt/replayproxy)
- [Ostinato](https://ostinato.org/)

## R�f�rences

- [Proxy de d�bogage Web Charles] (https://www.charlesproxy.com/)
- [Fiddler](https://www.telerik.com/fiddler/)
- [TCPDUMP](https://www.tcpdump.org/)
- [Ostinato](https://ostinato.org/)
