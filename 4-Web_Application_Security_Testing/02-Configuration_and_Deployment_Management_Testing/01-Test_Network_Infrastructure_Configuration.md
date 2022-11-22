# Tester la configuration de l'infrastructure r�seau

|ID          |
|------------|
|WSTG-CONF-01|

## Sommaire

La complexit� intrins�que de l'infrastructure de serveurs Web interconnect�s et h�t�rog�nes, qui peut inclure des centaines d'applications Web, fait de la gestion et de l'examen de la configuration une �tape fondamentale dans le test et le d�ploiement de chaque application. Il suffit d'une seule vuln�rabilit� pour saper la s�curit� de l'ensemble de l'infrastructure, et m�me des probl�mes mineurs et apparemment sans importance peuvent �voluer en risques graves pour une autre application sur le m�me serveur. Afin de r�soudre ces probl�mes, il est de la plus haute importance d'effectuer un examen approfondi de la configuration et des probl�mes de s�curit� connus, apr�s avoir cartographi� l'ensemble de l'architecture.

Une bonne gestion de la configuration de l'infrastructure du serveur Web est tr�s importante afin de pr�server la s�curit� de l'application elle-m�me. Si des �l�ments tels que le logiciel de serveur Web, les serveurs de base de donn�es principaux ou les serveurs d'authentification ne sont pas correctement examin�s et s�curis�s, ils peuvent introduire des risques ind�sirables ou introduire de nouvelles vuln�rabilit�s susceptibles de compromettre l'application elle-m�me.

Par exemple, une vuln�rabilit� de serveur Web qui permettrait � un attaquant distant de divulguer le code source de l'application elle-m�me (une vuln�rabilit� qui s'est produite � plusieurs reprises dans les serveurs Web ou les serveurs d'applications) pourrait compromettre l'application, car des utilisateurs anonymes pourraient utiliser les informations divulgu�es dans le code source pour tirer parti des attaques contre l'application ou ses utilisateurs.

Les �tapes suivantes doivent �tre suivies pour tester l'infrastructure de gestion de la configuration�:

- Les diff�rents �l�ments qui composent l'infrastructure doivent �tre d�termin�s afin de comprendre comment ils interagissent avec une application web et comment ils affectent sa s�curit�.
- Tous les �l�ments de l'infrastructure doivent �tre revus afin de s'assurer qu'ils ne contiennent pas de vuln�rabilit�s connues.
- Une r�vision doit �tre faite des outils administratifs utilis�s pour maintenir tous les diff�rents �l�ments.
- Les syst�mes d'authentification doivent �tre revus afin de s'assurer qu'ils r�pondent aux besoins de l'application et qu'ils ne peuvent pas �tre manipul�s par des utilisateurs externes pour tirer parti de l'acc�s.
- Une liste des ports d�finis qui sont requis pour l'application doit �tre maintenue et maintenue sous contr�le des modifications.

Apr�s avoir cartographi� les diff�rents �l�ments qui composent l'infrastructure (voir [Cartographier l'architecture du r�seau et des applications](../01-Information_Gathering/10-Map_Application_Architecture.md)) il est possible de revoir la configuration de chaque �l�ment fond� et de tester vuln�rabilit�s connues.

## Objectifs des tests

- Examinez les configurations des applications d�finies sur le r�seau et confirmez qu'elles ne sont pas vuln�rables.
- Valider que les cadres et les syst�mes utilis�s sont s�curis�s et non sensibles aux vuln�rabilit�s connues en raison de logiciels non entretenus ou de param�tres et d'informations d'identification par d�faut.

## Comment tester

### Vuln�rabilit�s de serveur connues

Les vuln�rabilit�s trouv�es dans les diff�rents domaines de l'architecture de l'application, que ce soit dans le serveur Web ou dans la base de donn�es principale, peuvent gravement compromettre l'application elle-m�me. Par exemple, consid�rez une vuln�rabilit� de serveur qui permet � un utilisateur distant non authentifi� de t�l�charger des fichiers sur le serveur Web ou m�me de remplacer des fichiers. Cette vuln�rabilit� pourrait compromettre l'application, car un utilisateur malhonn�te pourrait �tre en mesure de remplacer l'application elle-m�me ou d'introduire du code qui affecterait les serveurs principaux, car son code d'application serait ex�cut� comme n'importe quelle autre application.

L'examen des vuln�rabilit�s du serveur peut �tre difficile � faire si le test doit �tre effectu� via un test de p�n�tration � l'aveugle. Dans ces cas, les vuln�rabilit�s doivent �tre test�es � partir d'un site distant, g�n�ralement � l'aide d'un outil automatis�. Cependant, le test de certaines vuln�rabilit�s peut avoir des r�sultats impr�visibles sur le serveur Web, et le test d'autres (comme ceux directement impliqu�s dans les attaques par d�ni de service) peut ne pas �tre possible en raison du temps d'arr�t du service impliqu� si le test r�ussit.

Certains outils automatis�s signaleront les vuln�rabilit�s en fonction de la version du serveur Web r�cup�r�e. Cela conduit � la fois � des faux positifs et � des faux n�gatifs. D'une part, si la version du serveur Web a �t� supprim�e ou masqu�e par l'administrateur du site local, l'outil d'analyse ne signalera pas le serveur comme vuln�rable, m�me s'il l'est. D'autre part, si le fournisseur fournissant le logiciel ne met pas � jour la version du serveur Web lorsque les vuln�rabilit�s sont corrig�es, l'outil d'analyse signalera les vuln�rabilit�s qui n'existent pas. Ce dernier cas est en fait tr�s courant car certains fournisseurs de syst�mes d'exploitation renvoient des correctifs de vuln�rabilit�s de s�curit� vers le logiciel qu'ils fournissent dans le syst�me d'exploitation, mais ne font pas de t�l�chargement complet vers la derni�re version du logiciel. Cela se produit dans la plupart des distributions GNU/Linux telles que Debian, Red Hat ou SuSE. Dans la plupart des cas, l'analyse des vuln�rabilit�s d'une architecture d'application ne trouvera que les vuln�rabilit�s associ�es aux �l�ments "expos�s" de l'architecture (tels que le serveur Web) et sera g�n�ralement incapable de trouver les vuln�rabilit�s associ�es aux �l�ments qui ne sont pas directement expos�s, tels que les back-ends d'authentification, la base de donn�es back-end ou les proxys inverses [1] utilis�s.

Enfin, tous les �diteurs de logiciels ne divulguent pas les vuln�rabilit�s de mani�re publique et, par cons�quent, ces faiblesses ne sont pas enregistr�es dans les bases de donn�es de vuln�rabilit�s connues publiquement [2]. Ces informations ne sont divulgu�es qu'aux clients ou publi�es via des correctifs qui ne sont pas accompagn�s d'avis. Cela r�duit l'utilit� des outils d'analyse des vuln�rabilit�s. En r�gle g�n�rale, la couverture des vuln�rabilit�s de ces outils sera tr�s bonne pour les produits courants (tels que le serveur Web Apache, Internet Information Server de Microsoft ou Lotus Domino d'IBM), mais sera insuffisante pour les produits moins connus.

C'est pourquoi l'examen des vuln�rabilit�s est mieux effectu� lorsque le testeur dispose d'informations internes sur le logiciel utilis�, y compris les versions et les versions utilis�es et les correctifs appliqu�s au logiciel. Avec ces informations, le testeur peut r�cup�rer les informations aupr�s du fournisseur lui-m�me et analyser quelles vuln�rabilit�s pourraient �tre pr�sentes dans l'architecture et comment elles peuvent affecter l'application elle-m�me. Lorsque cela est possible, ces vuln�rabilit�s peuvent �tre test�es pour d�terminer leurs effets r�els et d�tecter s'il existe des �l�ments externes (tels que des syst�mes de d�tection ou de pr�vention des intrusions) susceptibles de r�duire ou d'annuler la possibilit� d'une exploitation r�ussie. Les testeurs pourraient m�me d�terminer, par le biais d'un examen de la configuration, que la vuln�rabilit� n'est m�me pas pr�sente, car elle affecte un composant logiciel qui n'est pas utilis�.

Il convient �galement de noter que les fournisseurs corrigent parfois silencieusement les vuln�rabilit�s et rendent les correctifs disponibles avec les nouvelles versions logicielles. Diff�rents fournisseurs auront des cycles de publication diff�rents qui d�termineront la prise en charge qu'ils pourraient fournir pour les versions plus anciennes. Un testeur disposant d'informations d�taill�es sur les versions logicielles utilis�es par l'architecture peut analyser le risque associ� � l'utilisation d'anciennes versions logicielles qui pourraient ne pas �tre prises en charge � court terme ou qui ne sont d�j� pas prises en charge. Ceci est essentiel, car si une vuln�rabilit� devait appara�tre dans une ancienne version du logiciel qui n'est plus prise en charge, le personnel syst�me pourrait ne pas en �tre directement conscient. Aucun correctif ne sera jamais mis � disposition pour cela et les avis pourraient ne pas r�pertorier cette version comme vuln�rable car elle n'est plus prise en charge. M�me dans le cas o� ils sont conscients que la vuln�rabilit� est pr�sente et que le syst�me est vuln�rable, ils devront effectuer une mise � niveau compl�te vers une nouvelle version du logiciel, ce qui pourrait introduire des temps d'arr�t importants dans l'architecture de l'application ou pourrait forcer l'application � �tre -cod� en raison d'incompatibilit�s avec la derni�re version du logiciel.

### Outils administratifs

Toute infrastructure de serveur Web n�cessite l'existence d'outils d'administration pour maintenir et mettre � jour les informations utilis�es par l'application. Ces informations incluent le contenu statique (pages Web, fichiers graphiques), le code source de l'application, les bases de donn�es d'authentification des utilisateurs, etc. Les outils d'administration seront diff�rents selon le site, la technologie ou le logiciel utilis�. Par exemple, certains serveurs Web seront g�r�s � l'aide d'interfaces d'administration qui sont, elles-m�mes, des serveurs Web (comme le serveur Web iPlanet) ou seront administr�s par des fichiers de configuration en texte brut (dans le cas d'Apache [3]) ou utiliseront des syst�mes d'exploitation Outils GUI (lors de l'utilisation du serveur IIS de Microsoft ou ASP.Net).

Dans la plupart des cas, la configuration du serveur sera g�r�e � l'aide de diff�rents outils de maintenance de fichiers utilis�s par le serveur Web, qui sont g�r�s via des serveurs FTP, WebDAV, des syst�mes de fichiers r�seau (NFS, CIFS) ou d'autres m�canismes. �videmment, le syst�me d'exploitation des �l�ments qui composent l'architecture de l'application sera �galement g�r� � l'aide d'autres outils. Les applications peuvent �galement avoir des interfaces administratives int�gr�es qui sont utilis�es pour g�rer les donn�es de l'application elle-m�me (utilisateurs, contenu, etc.).

Apr�s avoir cartographi� les interfaces d'administration utilis�es pour g�rer les diff�rentes parties de l'architecture, il est important de les revoir car si un attaquant parvient � acc�der � l'une d'entre elles, il peut alors compromettre ou endommager l'architecture de l'application. Pour ce faire, il est important de :

- D�terminer les m�canismes qui contr�lent l'acc�s � ces interfaces et leurs susceptibilit�s associ�es. Ces informations peuvent �tre disponibles en ligne.
- Modifiez le nom d'utilisateur et le mot de passe par d�faut.

Certaines entreprises choisissent de ne pas g�rer tous les aspects de leurs applications de serveur Web, mais peuvent confier � d'autres parties la gestion du contenu fourni par l'application Web. Cette soci�t� externe peut soit ne fournir qu'une partie du contenu (actualit�s ou promotions), soit g�rer enti�rement le serveur Web (y compris le contenu et le code). Il est courant de trouver des interfaces administratives disponibles sur Internet dans ces situations, car l'utilisation d'Internet est moins ch�re que la fourniture d'une ligne d�di�e qui connectera l'entreprise externe � l'infrastructure applicative via une interface de gestion uniquement. Dans cette situation, il est tr�s important de tester si les interfaces administratives peuvent �tre vuln�rables aux attaques.

## R�f�rences

- [1] WebSEAL, �galement connu sous le nom de Tivoli Authentication Manager, est un proxy inverse d'IBM qui fait partie du framework Tivoli.
- [2] Tels que Bugtraq de Symantec, X-Force d'ISS ou la base de donn�es nationale des vuln�rabilit�s (NVD) du NIST.
- [3] Il existe des outils d'administration bas�s sur une interface graphique pour Apache (comme NetLoony) mais ils ne sont pas encore largement utilis�s.
