# Tester la configuration de l'infrastructure réseau

|ID          |
|------------|
|WSTG-CONF-01|

## Sommaire

La complexité intrinsèque de l'infrastructure de serveurs Web interconnectés et hétérogènes, qui peut inclure des centaines d'applications Web, fait de la gestion et de l'examen de la configuration une étape fondamentale dans le test et le déploiement de chaque application. Il suffit d'une seule vulnérabilité pour saper la sécurité de l'ensemble de l'infrastructure, et même des problèmes mineurs et apparemment sans importance peuvent évoluer en risques graves pour une autre application sur le même serveur. Afin de résoudre ces problèmes, il est de la plus haute importance d'effectuer un examen approfondi de la configuration et des problèmes de sécurité connus, après avoir cartographié l'ensemble de l'architecture.

Une bonne gestion de la configuration de l'infrastructure du serveur Web est très importante afin de préserver la sécurité de l'application elle-même. Si des éléments tels que le logiciel de serveur Web, les serveurs de base de données principaux ou les serveurs d'authentification ne sont pas correctement examinés et sécurisés, ils peuvent introduire des risques indésirables ou introduire de nouvelles vulnérabilités susceptibles de compromettre l'application elle-même.

Par exemple, une vulnérabilité de serveur Web qui permettrait à un attaquant distant de divulguer le code source de l'application elle-même (une vulnérabilité qui s'est produite à plusieurs reprises dans les serveurs Web ou les serveurs d'applications) pourrait compromettre l'application, car des utilisateurs anonymes pourraient utiliser les informations divulguées dans le code source pour tirer parti des attaques contre l'application ou ses utilisateurs.

Les étapes suivantes doivent être suivies pour tester l'infrastructure de gestion de la configuration :

- Les différents éléments qui composent l'infrastructure doivent être déterminés afin de comprendre comment ils interagissent avec une application web et comment ils affectent sa sécurité.
- Tous les éléments de l'infrastructure doivent être revus afin de s'assurer qu'ils ne contiennent pas de vulnérabilités connues.
- Une révision doit être faite des outils administratifs utilisés pour maintenir tous les différents éléments.
- Les systèmes d'authentification doivent être revus afin de s'assurer qu'ils répondent aux besoins de l'application et qu'ils ne peuvent pas être manipulés par des utilisateurs externes pour tirer parti de l'accès.
- Une liste des ports définis qui sont requis pour l'application doit être maintenue et maintenue sous contrôle des modifications.

Après avoir cartographié les différents éléments qui composent l'infrastructure (voir [Cartographier l'architecture du réseau et des applications](../01-Information_Gathering/10-Map_Application_Architecture.md)) il est possible de revoir la configuration de chaque élément fondé et de tester vulnérabilités connues.

## Objectifs des tests

- Examinez les configurations des applications définies sur le réseau et confirmez qu'elles ne sont pas vulnérables.
- Valider que les cadres et les systèmes utilisés sont sécurisés et non sensibles aux vulnérabilités connues en raison de logiciels non entretenus ou de paramètres et d'informations d'identification par défaut.

## Comment tester

### Vulnérabilités de serveur connues

Les vulnérabilités trouvées dans les différents domaines de l'architecture de l'application, que ce soit dans le serveur Web ou dans la base de données principale, peuvent gravement compromettre l'application elle-même. Par exemple, considérez une vulnérabilité de serveur qui permet à un utilisateur distant non authentifié de télécharger des fichiers sur le serveur Web ou même de remplacer des fichiers. Cette vulnérabilité pourrait compromettre l'application, car un utilisateur malhonnête pourrait être en mesure de remplacer l'application elle-même ou d'introduire du code qui affecterait les serveurs principaux, car son code d'application serait exécuté comme n'importe quelle autre application.

L'examen des vulnérabilités du serveur peut être difficile à faire si le test doit être effectué via un test de pénétration à l'aveugle. Dans ces cas, les vulnérabilités doivent être testées à partir d'un site distant, généralement à l'aide d'un outil automatisé. Cependant, le test de certaines vulnérabilités peut avoir des résultats imprévisibles sur le serveur Web, et le test d'autres (comme ceux directement impliqués dans les attaques par déni de service) peut ne pas être possible en raison du temps d'arrêt du service impliqué si le test réussit.

Certains outils automatisés signaleront les vulnérabilités en fonction de la version du serveur Web récupérée. Cela conduit à la fois à des faux positifs et à des faux négatifs. D'une part, si la version du serveur Web a été supprimée ou masquée par l'administrateur du site local, l'outil d'analyse ne signalera pas le serveur comme vulnérable, même s'il l'est. D'autre part, si le fournisseur fournissant le logiciel ne met pas à jour la version du serveur Web lorsque les vulnérabilités sont corrigées, l'outil d'analyse signalera les vulnérabilités qui n'existent pas. Ce dernier cas est en fait très courant car certains fournisseurs de systèmes d'exploitation renvoient des correctifs de vulnérabilités de sécurité vers le logiciel qu'ils fournissent dans le système d'exploitation, mais ne font pas de téléchargement complet vers la dernière version du logiciel. Cela se produit dans la plupart des distributions GNU/Linux telles que Debian, Red Hat ou SuSE. Dans la plupart des cas, l'analyse des vulnérabilités d'une architecture d'application ne trouvera que les vulnérabilités associées aux éléments "exposés" de l'architecture (tels que le serveur Web) et sera généralement incapable de trouver les vulnérabilités associées aux éléments qui ne sont pas directement exposés, tels que les back-ends d'authentification, la base de données back-end ou les proxys inverses [1] utilisés.

Enfin, tous les éditeurs de logiciels ne divulguent pas les vulnérabilités de manière publique et, par conséquent, ces faiblesses ne sont pas enregistrées dans les bases de données de vulnérabilités connues publiquement [2]. Ces informations ne sont divulguées qu'aux clients ou publiées via des correctifs qui ne sont pas accompagnés d'avis. Cela réduit l'utilité des outils d'analyse des vulnérabilités. En règle générale, la couverture des vulnérabilités de ces outils sera très bonne pour les produits courants (tels que le serveur Web Apache, Internet Information Server de Microsoft ou Lotus Domino d'IBM), mais sera insuffisante pour les produits moins connus.

C'est pourquoi l'examen des vulnérabilités est mieux effectué lorsque le testeur dispose d'informations internes sur le logiciel utilisé, y compris les versions et les versions utilisées et les correctifs appliqués au logiciel. Avec ces informations, le testeur peut récupérer les informations auprès du fournisseur lui-même et analyser quelles vulnérabilités pourraient être présentes dans l'architecture et comment elles peuvent affecter l'application elle-même. Lorsque cela est possible, ces vulnérabilités peuvent être testées pour déterminer leurs effets réels et détecter s'il existe des éléments externes (tels que des systèmes de détection ou de prévention des intrusions) susceptibles de réduire ou d'annuler la possibilité d'une exploitation réussie. Les testeurs pourraient même déterminer, par le biais d'un examen de la configuration, que la vulnérabilité n'est même pas présente, car elle affecte un composant logiciel qui n'est pas utilisé.

Il convient également de noter que les fournisseurs corrigent parfois silencieusement les vulnérabilités et rendent les correctifs disponibles avec les nouvelles versions logicielles. Différents fournisseurs auront des cycles de publication différents qui détermineront la prise en charge qu'ils pourraient fournir pour les versions plus anciennes. Un testeur disposant d'informations détaillées sur les versions logicielles utilisées par l'architecture peut analyser le risque associé à l'utilisation d'anciennes versions logicielles qui pourraient ne pas être prises en charge à court terme ou qui ne sont déjà pas prises en charge. Ceci est essentiel, car si une vulnérabilité devait apparaître dans une ancienne version du logiciel qui n'est plus prise en charge, le personnel système pourrait ne pas en être directement conscient. Aucun correctif ne sera jamais mis à disposition pour cela et les avis pourraient ne pas répertorier cette version comme vulnérable car elle n'est plus prise en charge. Même dans le cas où ils sont conscients que la vulnérabilité est présente et que le système est vulnérable, ils devront effectuer une mise à niveau complète vers une nouvelle version du logiciel, ce qui pourrait introduire des temps d'arrêt importants dans l'architecture de l'application ou pourrait forcer l'application à être -codé en raison d'incompatibilités avec la dernière version du logiciel.

### Outils administratifs

Toute infrastructure de serveur Web nécessite l'existence d'outils d'administration pour maintenir et mettre à jour les informations utilisées par l'application. Ces informations incluent le contenu statique (pages Web, fichiers graphiques), le code source de l'application, les bases de données d'authentification des utilisateurs, etc. Les outils d'administration seront différents selon le site, la technologie ou le logiciel utilisé. Par exemple, certains serveurs Web seront gérés à l'aide d'interfaces d'administration qui sont, elles-mêmes, des serveurs Web (comme le serveur Web iPlanet) ou seront administrés par des fichiers de configuration en texte brut (dans le cas d'Apache [3]) ou utiliseront des systèmes d'exploitation Outils GUI (lors de l'utilisation du serveur IIS de Microsoft ou ASP.Net).

Dans la plupart des cas, la configuration du serveur sera gérée à l'aide de différents outils de maintenance de fichiers utilisés par le serveur Web, qui sont gérés via des serveurs FTP, WebDAV, des systèmes de fichiers réseau (NFS, CIFS) ou d'autres mécanismes. Évidemment, le système d'exploitation des éléments qui composent l'architecture de l'application sera également géré à l'aide d'autres outils. Les applications peuvent également avoir des interfaces administratives intégrées qui sont utilisées pour gérer les données de l'application elle-même (utilisateurs, contenu, etc.).

Après avoir cartographié les interfaces d'administration utilisées pour gérer les différentes parties de l'architecture, il est important de les revoir car si un attaquant parvient à accéder à l'une d'entre elles, il peut alors compromettre ou endommager l'architecture de l'application. Pour ce faire, il est important de :

- Déterminer les mécanismes qui contrôlent l'accès à ces interfaces et leurs susceptibilités associées. Ces informations peuvent être disponibles en ligne.
- Modifiez le nom d'utilisateur et le mot de passe par défaut.

Certaines entreprises choisissent de ne pas gérer tous les aspects de leurs applications de serveur Web, mais peuvent confier à d'autres parties la gestion du contenu fourni par l'application Web. Cette société externe peut soit ne fournir qu'une partie du contenu (actualités ou promotions), soit gérer entièrement le serveur Web (y compris le contenu et le code). Il est courant de trouver des interfaces administratives disponibles sur Internet dans ces situations, car l'utilisation d'Internet est moins chère que la fourniture d'une ligne dédiée qui connectera l'entreprise externe à l'infrastructure applicative via une interface de gestion uniquement. Dans cette situation, il est très important de tester si les interfaces administratives peuvent être vulnérables aux attaques.

## Références

- [1] WebSEAL, également connu sous le nom de Tivoli Authentication Manager, est un proxy inverse d'IBM qui fait partie du framework Tivoli.
- [2] Tels que Bugtraq de Symantec, X-Force d'ISS ou la base de données nationale des vulnérabilités (NVD) du NIST.
- [3] Il existe des outils d'administration basés sur une interface graphique pour Apache (comme NetLoony) mais ils ne sont pas encore largement utilisés.
