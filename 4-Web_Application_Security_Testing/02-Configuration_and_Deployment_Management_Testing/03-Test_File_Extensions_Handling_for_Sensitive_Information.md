# Testez la gestion des extensions de fichiers pour les informations sensibles

|ID          |
|------------|
|WSTG-CONF-03|

## Sommaire

Les extensions de fichiers sont couramment utilisées dans les serveurs Web pour déterminer facilement quelles technologies, langues et plug-ins doivent être utilisés pour répondre à la demande Web. Bien que ce comportement soit conforme aux RFC et aux normes Web, l'utilisation d'extensions de fichier standard fournit au testeur d'intrusion des informations utiles sur les technologies sous-jacentes utilisées dans une "appliance" Web et simplifie considérablement la tâche de détermination du scénario d'attaque à utiliser sur des technologies particulières. De plus, une mauvaise configuration des serveurs Web pourrait facilement révéler des informations confidentielles sur les identifiants d'accès.

La vérification des extensions est souvent utilisée pour valider les fichiers à télécharger, ce qui peut entraîner des résultats inattendus car le contenu n'est pas celui attendu ou en raison d'une gestion inattendue des noms de fichiers du système d'exploitation.

Déterminer comment les serveurs Web gèrent les requêtes correspondant aux fichiers ayant des extensions différentes peut aider à comprendre le comportement du serveur Web en fonction du type de fichiers auxquels on accède. Par exemple, cela peut aider à comprendre quelles extensions de fichier sont renvoyées sous forme de texte ou en clair par rapport à celles qui provoquent une exécution côté serveur. Ces derniers indiquent les technologies, les langages ou les plug-ins utilisés par les serveurs Web ou les serveurs d'applications, et peuvent fournir des informations supplémentaires sur la manière dont l'application Web est conçue. Par exemple, une extension ".pl" est généralement associée à la prise en charge de Perl côté serveur. Cependant, l'extension de fichier seule peut être trompeuse et pas totalement concluante. Par exemple, les ressources côté serveur Perl peuvent être renommées pour dissimuler le fait qu'elles sont effectivement liées à Perl. Consultez la section suivante sur les "composants de serveur Web" pour en savoir plus sur l'identification des technologies et des composants côté serveur.

## Objectifs des tests

- Extensions de fichiers sensibles à Dirbust ou pouvant contenir des données brutes (*par exemple* des scripts, des données brutes, des informations d'identification, etc.).
- Valider qu'aucun contournement de framework système n'existe sur l'ensemble de règles.

## Comment tester

### Navigation forcée

Soumettez des demandes avec différentes extensions de fichiers et vérifiez comment elles sont traitées. La vérification doit être effectuée par répertoire Web. Vérifiez les répertoires qui permettent l'exécution de scripts. Les répertoires de serveurs Web peuvent être identifiés par des outils d'analyse qui recherchent la présence de répertoires bien connus. De plus, la mise en miroir de la structure du site Web permet au testeur de reconstruire l'arborescence des répertoires Web servis par l'application.

Si l'architecture de l'application Web est à charge équilibrée, il est important d'évaluer tous les serveurs Web. Cela peut être facile ou non, selon la configuration de l'infrastructure d'équilibrage. Dans une infrastructure avec des composants redondants, il peut y avoir de légères variations dans la configuration des serveurs Web ou d'applications individuels. Cela peut se produire si l'architecture Web utilise des technologies hétérogènes (pensez à un ensemble de serveurs Web IIS et Apache dans une configuration d'équilibrage de charge, qui peut introduire un léger comportement asymétrique entre eux, et éventuellement des vulnérabilités différentes).

#### Exemple

Le testeur a identifié l'existence d'un fichier nommé `connection.inc`. Essayer d'y accéder directement restitue son contenu, qui sont :

```php
<?
    mysql_connect("127.0.0.1", "root", "password")
        or die("Could not connect");
?>
```

Le testeur détermine l'existence d'un back-end de SGBD MySQL et les informations d'identification (faibles) utilisées par l'application Web pour y accéder.

Les extensions de fichier suivantes ne doivent jamais être renvoyées par un serveur Web, car elles sont liées à des fichiers pouvant contenir des informations sensibles ou à des fichiers pour lesquels il n'y a aucune raison d'être servis.

- `.asa`
- `.inc`
- `.config`

Les extensions de fichier suivantes sont liées à des fichiers qui, lorsqu'ils sont consultés, sont soit affichés, soit téléchargés par le navigateur. Par conséquent, les fichiers avec ces extensions doivent être vérifiés pour vérifier qu'ils sont bien censés être servis (et ne sont pas des restes), et qu'ils ne contiennent pas d'informations sensibles.

- `.zip`, `.tar`, `.gz`, `.tgz`, `.rar`, etc. : fichiers d'archive (compressés)
- `.java` : aucune raison de fournir l'accès aux fichiers source Java
- `.txt` : Fichiers texte
- `.pdf` : document PDF
- `.docx`, `.rtf`, `.xlsx`, `.pptx`, etc. : Documents Office
- `.bak`, `.old` et autres extensions indiquant des fichiers de sauvegarde (par exemple : `~` pour les fichiers de sauvegarde Emacs)

La liste donnée ci-dessus ne détaille que quelques exemples, car les extensions de fichiers sont trop nombreuses pour être traitées ici de manière exhaustive. Reportez-vous à [FILExt](https://filext.com/) pour une base de données plus complète des extensions.

Pour identifier les fichiers ayant une extension donnée, un mélange de techniques peut être utilisé. Ces techniques peuvent inclure des scanners de vulnérabilité, des outils d'exploration et de mise en miroir, l'inspection manuelle de l'application (cela surmonte les limitations de l'exploration automatique), l'interrogation des moteurs de recherche (voir [Effectuer une reconnaissance de découverte de moteur de recherche pour les fuites d'informations](../01-Information_Gathering/01-Conduct_Search_Engine_Discovery_Reconnaissance_for_Information_Leakage.md)). Voir aussi [Examiner les anciennes sauvegardes et les fichiers non référencés pour les informations sensibles](04-Review_Old_Backup_and_Unreferenced_Files_for_Sensitive_Information.md) qui traite des problèmes de sécurité liés aux fichiers "oubliés".

### Téléchargement de fichiers

La gestion des fichiers hérités de Windows 8.3 peut parfois être utilisée pour contourner les filtres de téléchargement de fichiers.

Exemples d'utilisation :

1. `file.phtml` est traité comme du code PHP.
2. `FILE~1.PHT` est servi, mais pas traité par le gestionnaire PHP ISAPI.
3. `shell.phPWND` peut être téléchargé.
4. `SHELL~1.PHP` sera développé et renvoyé par le shell du système d'exploitation, puis traité par le gestionnaire PHP ISAPI.

### Test de la boîte grise

Effectuer des tests en boîte blanche sur la gestion des extensions de fichiers revient à vérifier les configurations des serveurs Web ou des serveurs d'applications participant à l'architecture de l'application Web et à vérifier comment ils sont chargés de servir différentes extensions de fichiers.

Si l'application Web s'appuie sur une infrastructure hétérogène à charge équilibrée, déterminez si cela peut introduire un comportement différent.

## Outils

Les scanners de vulnérabilité, tels que Nessus et Nikto, vérifient l'existence de répertoires Web bien connus. Ils peuvent permettre au testeur de télécharger la structure du site Web, ce qui est utile pour déterminer la configuration des répertoires Web et la manière dont les extensions de fichiers individuelles sont servies. Parmi les autres outils pouvant être utilisés à cette fin, citons :

- [wget](https://www.gnu.org/software/wget)
- [curl](https://curl.haxx.se)
- google pour "outils de mise en miroir Web".
