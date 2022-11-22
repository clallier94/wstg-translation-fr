# Testez la gestion des extensions de fichiers pour les informations sensibles

|ID          |
|------------|
|WSTG-CONF-03|

## Sommaire

Les extensions de fichiers sont couramment utilis�es dans les serveurs Web pour d�terminer facilement quelles technologies, langues et plug-ins doivent �tre utilis�s pour r�pondre � la demande Web. Bien que ce comportement soit conforme aux RFC et aux normes Web, l'utilisation d'extensions de fichier standard fournit au testeur d'intrusion des informations utiles sur les technologies sous-jacentes utilis�es dans une appliance Web et simplifie consid�rablement la t�che de d�termination du sc�nario d'attaque � utiliser sur des technologies particuli�res. De plus, une mauvaise configuration des serveurs Web pourrait facilement r�v�ler des informations confidentielles sur les identifiants d'acc�s.

La v�rification des extensions est souvent utilis�e pour valider les fichiers � t�l�charger, ce qui peut entra�ner des r�sultats inattendus car le contenu n'est pas celui attendu ou en raison d'une gestion inattendue des noms de fichiers du syst�me d'exploitation.

D�terminer comment les serveurs Web g�rent les requ�tes correspondant aux fichiers ayant des extensions diff�rentes peut aider � comprendre le comportement du serveur Web en fonction du type de fichiers auxquels on acc�de. Par exemple, cela peut aider � comprendre quelles extensions de fichier sont renvoy�es sous forme de texte ou en clair par rapport � celles qui provoquent une ex�cution c�t� serveur. Ces derniers indiquent les technologies, les langages ou les plug-ins utilis�s par les serveurs Web ou les serveurs d'applications, et peuvent fournir des informations suppl�mentaires sur la mani�re dont l'application Web est con�ue. Par exemple, une extension ".pl" est g�n�ralement associ�e � la prise en charge de Perl c�t� serveur. Cependant, l'extension de fichier seule peut �tre trompeuse et pas totalement concluante. Par exemple, les ressources c�t� serveur Perl peuvent �tre renomm�es pour dissimuler le fait qu'elles sont effectivement li�es � Perl. Consultez la section suivante sur les "composants de serveur Web" pour en savoir plus sur l'identification des technologies et des composants c�t� serveur.

## Objectifs des tests

- Extensions de fichiers sensibles � Dirbust ou pouvant contenir des donn�es brutes (*par exemple* des scripts, des donn�es brutes, des informations d'identification, etc.).
- Valider qu'aucun contournement de framework syst�me n'existe sur l'ensemble de r�gles.

## Comment tester

###�Navigation forc�e

Soumettez des demandes avec diff�rentes extensions de fichiers et v�rifiez comment elles sont trait�es. La v�rification doit �tre effectu�e par r�pertoire Web. V�rifiez les r�pertoires qui permettent l'ex�cution de scripts. Les r�pertoires de serveurs Web peuvent �tre identifi�s par des outils d'analyse qui recherchent la pr�sence de r�pertoires bien connus. De plus, la mise en miroir de la structure du site Web permet au testeur de reconstruire l'arborescence des r�pertoires Web servis par l'application.

Si l'architecture de l'application Web est � charge �quilibr�e, il est important d'�valuer tous les serveurs Web. Cela peut �tre facile ou non, selon la configuration de l'infrastructure d'�quilibrage. Dans une infrastructure avec des composants redondants, il peut y avoir de l�g�res variations dans la configuration des serveurs Web ou d'applications individuels. Cela peut se produire si l'architecture Web utilise des technologies h�t�rog�nes (pensez � un ensemble de serveurs Web IIS et Apache dans une configuration d'�quilibrage de charge, qui peut introduire un l�ger comportement asym�trique entre eux, et �ventuellement des vuln�rabilit�s diff�rentes).

#### Exemple

Le testeur a identifi� l'existence d'un fichier nomm� `connection.inc`. Essayer d'y acc�der directement restitue son contenu, qui sont :

```php
<?
    mysql_connect("127.0.0.1", "root", "password")
        or die("Could not connect");
?>
```

Le testeur d�termine l'existence d'un back-end de SGBD MySQL et les informations d'identification (faibles) utilis�es par l'application Web pour y acc�der.

Les extensions de fichier suivantes ne doivent jamais �tre renvoy�es par un serveur Web, car elles sont li�es � des fichiers pouvant contenir des informations sensibles ou � des fichiers pour lesquels il n'y a aucune raison d'�tre servis.

- `.asa`
- `.inc`
- `.config`

Les extensions de fichier suivantes sont li�es � des fichiers qui, lorsqu'ils sont consult�s, sont soit affich�s, soit t�l�charg�s par le navigateur. Par cons�quent, les fichiers avec ces extensions doivent �tre v�rifi�s pour v�rifier qu'ils sont bien cens�s �tre servis (et ne sont pas des restes), et qu'ils ne contiennent pas d'informations sensibles.

- `.zip`, `.tar`, `.gz`, `.tgz`, `.rar`, etc.�: fichiers d'archive (compress�s)
- `.java`�: aucune raison de fournir l'acc�s aux fichiers source Java
- `.txt`�: Fichiers texte
- `.pdf`�: document PDF
- `.docx`, `.rtf`, `.xlsx`, `.pptx`, etc. : Documents Office
- `.bak`, `.old` et autres extensions indiquant des fichiers de sauvegarde (par exemple�: `~` pour les fichiers de sauvegarde Emacs)

La liste donn�e ci-dessus ne d�taille que quelques exemples, car les extensions de fichiers sont trop nombreuses pour �tre trait�es ici de mani�re exhaustive. Reportez-vous � [FILExt](https://filext.com/) pour une base de donn�es plus compl�te des extensions.

Pour identifier les fichiers ayant une extension donn�e, un m�lange de techniques peut �tre utilis�. Ces techniques peuvent inclure des scanners de vuln�rabilit�, des outils d'exploration et de mise en miroir, l'inspection manuelle de l'application (cela surmonte les limitations de l'exploration automatique), l'interrogation des moteurs de recherche (voir [Test�: exploration et recherche sur Google](../01-Information_Gathering/01-Conduct_Search_Engine_Discovery_Reconnaissance_for_Information_Leakage.md )). Voir aussi [Test des fichiers anciens, de sauvegarde et non r�f�renc�s](04-Review_Old_Backup_and_Unreferenced_Files_for_Sensitive_Information.md) qui traite des probl�mes de s�curit� li�s aux fichiers "oubli�s".

### T�l�chargement de fichiers

La gestion des fichiers h�rit�s de Windows 8.3 peut parfois �tre utilis�e pour contourner les filtres de t�l�chargement de fichiers.

Exemples d'utilisation�:

1. `file.phtml` est trait� comme du code PHP.
2. `FILE~1.PHT` est servi, mais pas trait� par le gestionnaire PHP ISAPI.
3. `shell.phPWND` peut �tre t�l�charg�.
4. `SHELL~1.PHP` sera d�velopp� et renvoy� par le shell du syst�me d'exploitation, puis trait� par le gestionnaire PHP ISAPI.

###�Test de la bo�te grise

Effectuer des tests en bo�te blanche sur la gestion des extensions de fichiers revient � v�rifier les configurations des serveurs Web ou des serveurs d'applications participant � l'architecture de l'application Web et � v�rifier comment ils sont charg�s de servir diff�rentes extensions de fichiers.

Si l'application Web s'appuie sur une infrastructure h�t�rog�ne � charge �quilibr�e, d�terminez si cela peut introduire un comportement diff�rent.

## Outils

Les scanners de vuln�rabilit�, tels que Nessus et Nikto, v�rifient l'existence de r�pertoires Web bien connus. Ils peuvent permettre au testeur de t�l�charger la structure du site Web, ce qui est utile pour d�terminer la configuration des r�pertoires Web et la mani�re dont les extensions de fichiers individuelles sont servies. Parmi les autres outils pouvant �tre utilis�s � cette fin, citons :

- [wget](https://www.gnu.org/software/wget)
- [curl](https://curl.haxx.se)
- google pour "outils de mise en miroir Web".
