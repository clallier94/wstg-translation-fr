# Effectuer une reconnaissance de d�couverte de moteur de recherche pour les fuites d'informations

|ID          |
|------------|
|WSTG-INFO-01|

## Sommaire

Pour que les moteurs de recherche fonctionnent, les programmes informatiques (ou "robots") r�cup�rent r�guli�rement des donn�es (appel�es [exploration](https://en.wikipedia.org/wiki/Web_crawler)) � partir de milliards de pages sur le Web. Ces programmes trouvent du contenu et des fonctionnalit�s Web en suivant des liens � partir d'autres pages ou en consultant des plans de site. Si un site Web utilise un fichier sp�cial appel� "robots.txt" pour r�pertorier les pages qu'il ne souhaite pas que les moteurs de recherche r�cup�rent, les pages qui y sont r�pertori�es seront ignor�es. Il s'agit d'un aper�u de base. Google propose une explication plus d�taill�e du [fonctionnement d'un moteur de recherche](https://support.google.com/webmasters/answer/70897?hl=en).

Les testeurs peuvent utiliser des moteurs de recherche pour effectuer des reconnaissances sur des sites Web et des applications Web. Il existe des �l�ments directs et indirects dans la d�couverte et la reconnaissance des moteurs de recherche�: les m�thodes directes concernent la recherche dans les index et le contenu associ� � partir des caches, tandis que les m�thodes indirectes concernent l'apprentissage d'informations de conception et de configuration sensibles en recherchant des forums, des groupes de discussion et des sites Web d'appel d'offres.

Une fois qu'un robot de moteur de recherche a termin� l'exploration, il commence � indexer le contenu Web en fonction des balises et des attributs associ�s, tels que `<TITLE>`, afin de renvoyer des r�sultats de recherche pertinents. Si le fichier `robots.txt` n'est pas mis � jour pendant la dur�e de vie du site Web et que les balises m�ta HTML en ligne indiquant aux robots de ne pas indexer le contenu n'ont pas �t� utilis�es, il est possible que les index contiennent du contenu Web non pr�vu. � inclure par les propri�taires. Les propri�taires de sites Web peuvent utiliser le "robots.txt" mentionn� pr�c�demment, les balises m�ta HTML, l'authentification et les outils fournis par les moteurs de recherche pour supprimer ce contenu.

## Objectifs des tests

- Identifiez les informations sensibles de conception et de configuration de l'application, du syst�me ou de l'organisation qui sont expos�es directement (sur le site Web de l'organisation) ou indirectement (via des services tiers).

## Comment tester

Utilisez un moteur de recherche pour rechercher des informations potentiellement sensibles. Cela peut inclure�:

- sch�mas et configurations de r�seau�;
- messages et e-mails archiv�s par des administrateurs ou d'autres membres cl�s du personnel�;
- proc�dures de connexion et formats de nom d'utilisateur�;
- noms d'utilisateur, mots de passe et cl�s priv�es�;
- fichiers de configuration de services tiers ou cloud�;
- r�v�ler le contenu du message d'erreur�; et
- applications non publiques (d�veloppement, test, test d'acceptation par l'utilisateur (UAT) et versions interm�diaires des sites).

### Moteurs de recherche

Ne limitez pas les tests � un seul fournisseur de moteur de recherche, car diff�rents moteurs de recherche peuvent g�n�rer des r�sultats diff�rents. Les r�sultats des moteurs de recherche peuvent varier de plusieurs mani�res, en fonction de la date � laquelle le moteur a explor� le contenu pour la derni�re fois et de l'algorithme utilis� par le moteur pour d�terminer les pages pertinentes. Envisagez d'utiliser les moteurs de recherche suivants (par ordre alphab�tique)�:

- [Baidu](https://www.baidu.com/), le moteur de recherche [le plus populaire](https://en.wikipedia.org/wiki/Web_search_engine#Market_share) de Chine.
- [Bing](https://www.bing.com/), un moteur de recherche d�tenu et exploit� par Microsoft, et le deuxi�me [plus populaire](https://en.wikipedia.org/wiki/Web_search_engine#Market_share) � l'�chelle mondiale. Prend en charge les [mots cl�s de recherche avanc�e] (http://help.bing.microsoft.com/#apex/18/en-US/10001/-1).
- [binsearch.info](https://binsearch.info/), un moteur de recherche pour les newsgroups Usenet binaires.
- [Common Crawl](https://commoncrawl.org/), "un r�f�rentiel ouvert de donn�es d'exploration du Web qui peut �tre consult� et analys� par n'importe qui."
- [DuckDuckGo](https://duckduckgo.com/), un moteur de recherche ax� sur la confidentialit� qui compile les r�sultats de nombreuses [sources] diff�rentes(https://help.duckduckgo.com/results/sources/). Prend en charge la [syntaxe de recherche](https://help.duckduckgo.com/duckduckgo-help-pages/results/syntax/).
- [Google](https://www.google.com/), qui propose le moteur de recherche [le plus populaire](https://en.wikipedia.org/wiki/Web_search_engine#Market_share) au monde et utilise un syst�me de classement tenter de renvoyer les r�sultats les plus pertinents. Prend en charge les [op�rateurs de recherche](https://support.google.com/websearch/answer/2466433).
- [Internet Archive Wayback Machine](https://archive.org/web/), "construire une biblioth�que num�rique de sites Internet et d'autres artefacts culturels sous forme num�rique".
- [Shodan](https://www.shodan.io/), un service de recherche d'appareils et de services connect�s � Internet. Les options d'utilisation incluent un plan gratuit limit� ainsi que des plans d'abonnement payants.

### Op�rateurs de recherche

Un op�rateur de recherche est un mot-cl� ou une syntaxe sp�ciale qui �tend les capacit�s des requ�tes de recherche r�guli�res et peut aider � obtenir des r�sultats plus sp�cifiques. Ils prennent g�n�ralement la forme `operator:query`. Voici quelques op�rateurs de recherche couramment pris en charge�:

- `site:` limitera la recherche au domaine fourni.
- `inurl:` renverra uniquement les r�sultats qui incluent le mot-cl� dans l'URL.
- `intitle:` renverra uniquement les r�sultats qui ont le mot-cl� dans le titre de la page.
- `intext:` ou `inbody:` recherchera uniquement le mot-cl� dans le corps des pages.
- `filetype:` correspondra uniquement � un type de fichier sp�cifique, c'est-�-dire `.png` ou `.php`.

Par exemple, pour trouver le contenu Web de owasp.org tel qu'index� par un moteur de recherche typique, la syntaxe requise est�:

```text
site:owasp.org
```

![Exemple de r�sultat de recherche d'op�ration de site Google](images/Google_site_Operator_Search_Results_Example_20200406.png)\
*Figure 4.1.1-1 : Exemple de r�sultat de recherche d'op�ration de site Google*

### Affichage du contenu mis en cache

Pour rechercher du contenu qui a d�j� �t� index�, utilisez l'op�rateur `cache:`. Ceci est utile pour afficher le contenu qui peut avoir chang� depuis le moment o� il a �t� index�, ou qui peut ne plus �tre disponible. Tous les moteurs de recherche ne fournissent pas de contenu mis en cache pour la recherche�; la source la plus utile au moment de la r�daction est Google.

Pour afficher `owasp.org` tel qu'il est mis en cache, la syntaxe est la suivante�:

```text
cache:owasp.org
```

![Exemple de r�sultat de recherche d'op�ration Google Cache](images/Google_cache_Operator_Search_Results_Example_20200406.png)\
*Figure 4.1.1-2 : Exemple de r�sultat de recherche d'op�ration Google Cache*

### Google Hacking, ou Dorking

La recherche avec les op�rateurs peut �tre une technique de d�couverte tr�s efficace lorsqu'elle est combin�e � la cr�ativit� du testeur. Les op�rateurs peuvent �tre encha�n�s pour d�couvrir efficacement des types sp�cifiques de fichiers et d'informations sensibles. Cette technique, appel�e [Google hacking](https://en.wikipedia.org/wiki/Google_hacking) ou Dorking, est �galement possible en utilisant d'autres moteurs de recherche, tant que les op�rateurs de recherche sont pris en charge.

Une base de donn�es de dorks, telle que [Google Hacking Database](https://www.exploit-db.com/google-hacking-database), est une ressource utile qui peut aider � d�couvrir des informations sp�cifiques. Certaines cat�gories de dorks disponibles sur cette base de donn�es incluent:

- Prises de pieds
- Fichiers contenant des noms d'utilisateur
- R�pertoires sensibles
- D�tection de serveur Web
- Fichiers vuln�rables
- Serveurs vuln�rables
- Messages d'erreur
- Fichiers contenant des informations juteuses
- Fichiers contenant des mots de passe
- Informations sensibles sur les achats en ligne

## Correction

Examinez attentivement la sensibilit� des informations de conception et de configuration avant de les publier en ligne.

Examinez p�riodiquement la sensibilit� des informations de conception et de configuration existantes publi�es en ligne.
