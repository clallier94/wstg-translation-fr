# Tester la configuration de la plate-forme d'application

|ID          |
|------------|
|WSTG-CONF-02|

## Sommaire

Une configuration correcte des éléments individuels qui composent une architecture d'application est importante afin d'éviter les erreurs qui pourraient compromettre la sécurité de l'ensemble de l'architecture.

L'examen et le test de la configuration sont des tâches essentielles dans la création et la maintenance d'une architecture. En effet, de nombreux systèmes différents seront généralement fournis avec des configurations génériques qui pourraient ne pas être adaptées à la tâche qu'ils effectueront sur le site spécifique sur lequel ils sont installés.

Bien que l'installation typique d'un serveur Web et d'applications contienne de nombreuses fonctionnalités (comme des exemples d'application, de la documentation, des pages de test), ce qui n'est pas essentiel doit être supprimé avant le déploiement pour éviter une exploitation post-installation.

## Objectifs des tests

- Assurez-vous que les fichiers par défaut et connus ont été supprimés.
- Vérifiez qu'aucun code de débogage ou extension n'est laissé dans les environnements de production.
- Revoir les mécanismes de journalisation mis en place pour l'application.

## Comment tester

### Test de la boîte noire

#### Exemples de fichiers et répertoires connus

De nombreux serveurs Web et serveurs d'applications fournissent, dans une installation par défaut, des exemples d'applications et de fichiers à l'intention du développeur et afin de tester le bon fonctionnement du serveur juste après l'installation. Cependant, de nombreuses applications de serveur Web par défaut ont été connues par la suite pour être vulnérables. Ce fut le cas, par exemple, pour CVE-1999-0449 (Déni de service dans IIS lorsque le site d'exemple Exair avait été installé), CAN-2002-1744 (Vulnérabilité de traversée de répertoire dans CodeBrws.asp dans Microsoft IIS 5.0), CAN -2002-1630 (utilisation de sendmail.jsp dans Oracle 9iAS), ou CAN-2003-1172 (parcours de répertoire dans l'exemple de source d'affichage dans Cocoon d'Apache).

Les analyseurs CGI incluent une liste détaillée des fichiers connus et des exemples de répertoires fournis par différents serveurs Web ou d'applications et peuvent constituer un moyen rapide de déterminer si ces fichiers sont présents. Cependant, la seule façon d'être vraiment sûr est de faire un examen complet du contenu du serveur Web ou du serveur d'applications et de déterminer s'ils sont liés à l'application elle-même ou non.

#### Examen des commentaires

Il est très courant que les programmeurs ajoutent des commentaires lors du développement de grandes applications Web. Cependant, les commentaires inclus en ligne dans le code HTML peuvent révéler des informations internes qui ne devraient pas être disponibles pour un attaquant. Parfois, même le code source est commenté car une fonctionnalité n'est plus requise, mais ce commentaire est divulgué involontairement aux pages HTML renvoyées aux utilisateurs.

L'examen des commentaires doit être effectué afin de déterminer si des informations sont divulguées par le biais des commentaires. Cet examen ne peut être effectué de manière approfondie que par une analyse du contenu statique et dynamique du serveur Web et par des recherches de fichiers. Il peut être utile de parcourir le site de manière automatique ou guidée et de stocker tout le contenu récupéré. Ce contenu récupéré peut ensuite être recherché afin d'analyser les éventuels commentaires HTML disponibles dans le code.

#### Configuration du système

Divers outils, documents ou listes de contrôle peuvent être utilisés pour donner aux professionnels de l'informatique et de la sécurité une évaluation détaillée de la conformité des systèmes cibles à diverses lignes de base ou références de configuration. Ces outils incluent (mais ne sont pas limités à) :

- [CIS-CAT Lite](https://www.cisecurity.org/blog/introducing-cis-cat-lite/)
- [Analyseur de surface d'attaque de Microsoft](https://github.com/microsoft/AttackSurfaceAnalyzer)
- [Programme national de liste de contrôle du NIST] (https://nvd.nist.gov/ncp/repository)

### Test de la boîte grise

#### Examen de la configuration

La configuration du serveur Web ou du serveur d'application joue un rôle important dans la protection du contenu du site et doit être soigneusement examinée afin de repérer les erreurs de configuration courantes. Évidemment, la configuration recommandée varie en fonction de la politique du site et des fonctionnalités qui doivent être fournies par le logiciel serveur. Dans la plupart des cas, cependant, les instructions de configuration (fournies par le fournisseur du logiciel ou par des tiers) doivent être suivies pour déterminer si le serveur a été correctement sécurisé.

Il est impossible de dire de manière générique comment un serveur doit être configuré, cependant, certaines directives communes doivent être prises en compte :

- N'activez que les modules serveur (extensions ISAPI dans le cas d'IIS) nécessaires à l'application. Cela réduit la surface d'attaque puisque le serveur est réduit en taille et en complexité lorsque les modules logiciels sont désactivés. Il empêche également les vulnérabilités susceptibles d'apparaître dans le logiciel du fournisseur d'affecter le site si elles ne sont présentes que dans des modules déjà désactivés.
- Gérez les erreurs de serveur (40x ou 50x) avec des pages personnalisées au lieu des pages de serveur Web par défaut. Assurez-vous spécifiquement que les erreurs d'application ne seront pas renvoyées à l'utilisateur final et qu'aucun code n'est divulgué à travers ces erreurs, car cela aidera un attaquant. Il est en fait très courant d'oublier ce point car les développeurs ont besoin de ces informations dans les environnements de pré-production.
- Assurez-vous que le logiciel serveur s'exécute avec des privilèges minimisés dans le système d'exploitation. Cela empêche une erreur dans le logiciel serveur de compromettre directement l'ensemble du système, bien qu'un attaquant puisse élever les privilèges une fois le code exécuté en tant que serveur Web.
- Assurez-vous que le logiciel serveur enregistre correctement les accès légitimes et les erreurs.
- Assurez-vous que le serveur est configuré pour gérer correctement les surcharges et empêcher les attaques par déni de service. Assurez-vous que les performances du serveur ont été correctement réglées.
- N'accordez jamais aux identités non administratives (à l'exception de `NT SERVICE\WMSvc`) l'accès à applicationHost.config, redirection.config et administration.config (accès en lecture ou en écriture). Cela inclut `Network Service`, `IIS_IUSRS`, `IUSR` ou toute identité personnalisée utilisée par les pools d'applications IIS. Les processus de travail IIS ne sont pas destinés à accéder directement à ces fichiers.
- Ne partagez jamais applicationHost.config, redirection.config et administration.config sur le réseau. Lorsque vous utilisez la configuration partagée, préférez exporter applicationHost.config vers un autre emplacement (voir la section intitulée "Définir les autorisations pour la configuration partagée).
- Gardez à l'esprit que tous les utilisateurs peuvent lire les fichiers .NET Framework `machine.config` et racine `web.config` par défaut. Ne stockez pas d'informations sensibles dans ces fichiers si elles doivent être réservées aux yeux de l'administrateur.
- Chiffrez les informations sensibles qui doivent être lues uniquement par les processus de travail IIS et non par les autres utilisateurs de la machine.
- N'accordez pas l'accès en écriture à l'identité que le serveur Web utilise pour accéder à l'`applicationHost.config` partagé. Cette identité ne doit avoir qu'un accès en lecture.
- Utilisez une identité distincte pour publier applicationHost.config sur le partage. N'utilisez pas cette identité pour configurer l'accès à la configuration partagée sur les serveurs Web.
- Utilisez un mot de passe fort lors de l'exportation des clés de cryptage à utiliser avec la configuration partagée.
- Maintenir un accès restreint au partage contenant la configuration partagée et les clés de chiffrement. Si ce partage est compromis, un attaquant pourra lire et écrire n'importe quelle configuration IIS pour vos serveurs Web, rediriger le trafic de votre site Web vers des sources malveillantes et, dans certains cas, prendre le contrôle de tous les serveurs Web en chargeant du code arbitraire dans IIS worker. processus.
- Envisagez de protéger ce partage avec des règles de pare-feu et des stratégies IPsec pour autoriser uniquement les serveurs Web membres à se connecter.

#### Journalisation

La journalisation est un atout important de la sécurité d'une architecture applicative, puisqu'elle peut être utilisée pour détecter les failles des applications (les utilisateurs essayant constamment de récupérer un fichier qui n'existe pas réellement) ainsi que les attaques soutenues d'utilisateurs malhonnêtes. Les journaux sont généralement correctement générés par le Web et d'autres logiciels de serveur. Il n'est pas courant de trouver des applications qui enregistrent correctement leurs actions dans un journal et, lorsqu'elles le font, l'intention principale des journaux d'application est de produire une sortie de débogage qui pourrait être utilisée par le programmeur pour analyser une erreur particulière.

Dans les deux cas (journaux du serveur et des applications), plusieurs problèmes doivent être testés et analysés en fonction du contenu des journaux :

1. Les journaux contiennent-ils des informations sensibles ?
2. Les logs sont-ils stockés sur un serveur dédié ?
3. L'utilisation du journal peut-elle générer une condition de déni de service ?
4. Comment sont-ils tournés ? Les journaux sont-ils conservés suffisamment longtemps ?
5. Comment les journaux sont-ils examinés ? Les administrateurs peuvent-ils utiliser ces révisions pour détecter des attaques ciblées ?
6. Comment les sauvegardes de journaux sont-elles conservées ?
7. Les données enregistrées sont-elles validées (longueur min/max, caractères, etc.) avant d'être enregistrées ?

##### Informations sensibles dans les journaux

Certaines applications peuvent, par exemple, utiliser des requêtes GET pour transférer des données de formulaire qui seront visibles dans les journaux du serveur. Cela signifie que les journaux du serveur peuvent contenir des informations sensibles (telles que des noms d'utilisateur comme des mots de passe ou des détails de compte bancaire). Ces informations sensibles peuvent être utilisées à mauvais escient par un attaquant s'il a obtenu les journaux, par exemple, via des interfaces administratives ou des vulnérabilités ou une mauvaise configuration connues du serveur Web (comme la mauvaise configuration bien connue de l'état du serveur dans les serveurs HTTP basés sur Apache).

Les journaux d'événements contiennent souvent des données utiles à un attaquant (fuite d'informations) ou peuvent être utilisés directement dans des exploits :

- Informations de débogage
- Traces de pile
- Noms d'utilisateur
- Noms des composants du système
- Adresses IP internes
- Données personnelles moins sensibles (par exemple, adresses e-mail, adresses postales et numéros de téléphone associés à des personnes nommées)
- Données d'entreprise

En outre, dans certaines juridictions, le stockage de certaines informations sensibles dans des fichiers journaux, telles que des données personnelles, peut obliger l'entreprise à appliquer également les lois sur la protection des données qu'elle appliquerait à ses bases de données principales pour les fichiers journaux. Et ne pas le faire, même sans le savoir, peut entraîner des sanctions en vertu des lois sur la protection des données qui s'appliquent.

Une liste plus large d'informations sensibles est:

- Code source de l'application
- Valeurs d'identification de session
- Jetons d'accès
- Données personnelles sensibles et certaines formes d'informations personnellement identifiables (IPI)
- Mots de passe d'authentification
- Chaînes de connexion à la base de données
- Clés de chiffrement
- Données du titulaire du compte bancaire ou de la carte de paiement
- Données d'une classification de sécurité supérieure à celle que le système de journalisation est autorisé à stocker
- Informations commercialement sensibles
- Informations qu'il est illégal de collecter dans la juridiction concernée
- Informations qu'un utilisateur a refusé de collecter ou auxquelles il n'a pas consenti, par ex. utilisation de ne pas suivre, ou lorsque le consentement à la collecte a expiré

#### Emplacement du journal

Généralement, les serveurs génèrent des journaux locaux de leurs actions et erreurs, consommant le disque du système sur lequel le serveur s'exécute. Cependant, si le serveur est compromis, ses journaux peuvent être effacés par l'intrus pour nettoyer toutes les traces de son attaque et de ses méthodes. Si cela devait se produire, l'administrateur système n'aurait aucune connaissance de la manière dont l'attaque s'est produite ni de l'emplacement de la source de l'attaque. En fait, la plupart des kits d'outils des attaquants incluent un « zappeur de journaux » capable de nettoyer tous les journaux contenant des informations données (comme l'adresse IP de l'attaquant) et sont régulièrement utilisés dans les rootkits au niveau du système de l'attaquant.

Par conséquent, il est plus sage de conserver les journaux dans un emplacement séparé et non sur le serveur Web lui-même. Cela facilite également l'agrégation de journaux provenant de différentes sources qui font référence à la même application (comme ceux d'une batterie de serveurs Web) et facilite également l'analyse des journaux (qui peut être gourmande en CPU) sans affecter le serveur lui-même.

#### Stockage des journaux

Les journaux peuvent introduire une condition de déni de service s'ils ne sont pas correctement stockés. Tout attaquant disposant de ressources suffisantes pourrait être en mesure de produire un nombre suffisant de requêtes qui rempliraient l'espace alloué aux fichiers journaux, s'il n'en est pas spécifiquement empêché. Cependant, si le serveur n'est pas correctement configuré, les fichiers journaux seront stockés dans la même partition de disque que celle utilisée pour le logiciel du système d'exploitation ou l'application elle-même. Cela signifie que si le disque devait être rempli, le système d'exploitation ou l'application pourrait échouer car il ne peut pas écrire sur le disque.

Généralement, dans les systèmes UNIX, les journaux se trouvent dans /var (bien que certaines installations de serveur puissent résider dans /opt ou /usr/local) et il est important de s'assurer que les répertoires dans lesquels les journaux sont stockés se trouvent dans une partition distincte. Dans certains cas, et afin d'éviter que les journaux système ne soient affectés, le répertoire des journaux du logiciel serveur lui-même (tel que /var/log/apache dans le serveur Web Apache) doit être stocké dans une partition dédiée.

Cela ne veut pas dire que les journaux doivent être autorisés à croître pour remplir le système de fichiers dans lequel ils résident. La croissance des journaux du serveur doit être surveillée afin de détecter cette condition car elle peut indiquer une attaque.

Tester cette condition est aussi simple et aussi dangereux dans les environnements de production que de lancer un nombre suffisant et soutenu de requêtes pour voir si ces requêtes sont enregistrées et s'il est possible de remplir la partition de journal via ces requêtes. Dans certains environnements où les paramètres QUERY_STRING sont également enregistrés, qu'ils soient produits via des requêtes GET ou POST, de grandes requêtes peuvent être simulées, ce qui remplira les journaux plus rapidement car, généralement, une seule requête ne causera qu'une petite quantité de données. enregistrés, tels que la date et l'heure, l'adresse IP source, la requête URI et le résultat du serveur.

#### Rotation du journal

La plupart des serveurs (mais peu d'applications personnalisées) effectueront une rotation des journaux afin de les empêcher de remplir le système de fichiers sur lequel ils résident. Lors de la rotation des journaux, l'hypothèse est que les informations qu'ils contiennent ne sont nécessaires que pour une durée limitée.

Cette fonctionnalité doit être testée afin de s'assurer que :

- Les logs sont conservés pendant le temps défini dans la politique de sécurité, ni plus ni moins.
- Les journaux sont compressés une fois tournés (c'est une commodité, car cela signifie que plus de journaux seront stockés pour le même espace disque disponible).
- Les autorisations du système de fichiers des fichiers journaux en rotation sont les mêmes (ou plus strictes) que celles des fichiers journaux eux-mêmes. Par exemple, les serveurs Web devront écrire dans les journaux qu'ils utilisent, mais ils n'ont pas réellement besoin d'écrire dans les journaux en rotation, ce qui signifie que les autorisations des fichiers peuvent être modifiées lors de la rotation pour empêcher le processus du serveur Web de les modifier.

Certains serveurs peuvent effectuer une rotation des journaux lorsqu'ils atteignent une taille donnée. Si cela se produit, il faut s'assurer qu'un attaquant ne peut pas forcer les bûches à tourner afin de masquer ses traces.

#### Contrôle d'accès au journal

Les informations du journal des événements ne doivent jamais être visibles pour les utilisateurs finaux. Même les administrateurs Web ne devraient pas être en mesure de voir ces journaux, car cela rompt les contrôles de séparation des tâches. Assurez-vous que tout schéma de contrôle d'accès utilisé pour protéger l'accès aux journaux bruts et toutes les applications fournissant des fonctionnalités permettant d'afficher ou de rechercher les journaux ne sont pas liés à des schémas de contrôle d'accès pour d'autres rôles d'utilisateur d'application. Les données de journal ne doivent pas non plus être visibles par des utilisateurs non authentifiés.

#### Examen du journal

L'examen des journaux peut être utilisé pour plus que l'extraction des statistiques d'utilisation des fichiers sur les serveurs Web (ce qui est généralement ce sur quoi la plupart des applications basées sur les journaux se concentreront), mais également pour déterminer si des attaques ont lieu sur le serveur Web.

Afin d'analyser les attaques de serveur Web, les fichiers journaux d'erreurs du serveur doivent être analysés. L'examen doit se concentrer sur :

- Messages d'erreur 40x (non trouvés). Une grande quantité d'entre eux provenant de la même source peut indiquer qu'un outil d'analyse CGI est utilisé contre le serveur Web
- Messages 50x (erreur de serveur). Ceux-ci peuvent être une indication d'un attaquant abusant de parties de l'application qui échouent de manière inattendue. Par exemple, les premières phases d'une attaque par injection SQL produiront ces messages d'erreur lorsque la requête SQL n'est pas correctement construite et que son exécution échoue sur la base de données principale.

Les statistiques ou l'analyse des journaux ne doivent pas être générées, ni stockées, sur le même serveur qui produit les journaux. Sinon, un attaquant pourrait, via une vulnérabilité de serveur Web ou une configuration incorrecte, y accéder et récupérer des informations similaires à celles qui seraient divulguées par les fichiers journaux eux-mêmes.

## Références

- Apache
    - Apache Security, par Ivan Ristic, O'reilly, mars 2005.
    - [Apache Security Secrets: Revealed (Again), Mark Cox, novembre 2003](https://awe.com/mark/talks/apachecon2003us.html)
    - [Apache Security Secrets : Revealed, ApacheCon 2002, Las Vegas, Mark J Cox, octobre 2002](https://awe.com/mark/talks/apachecon2002us.html)
    - [Réglage des performances](https://httpd.apache.org/docs/current/misc/perf-tuning.html)
-Lotus Domino
    - Lotus Security Handbook, William Tworek et al., avril 2004, disponible dans la collection IBM Redbooks
    - Lotus Domino Security, un livre blanc X-force, Internet Security Systems, décembre 2002
    - Protection contre le piratage du serveur Web Lotus Domino, David Litchfield, octobre 2001
-Microsoft IIS
    - [Meilleures pratiques de sécurité pour IIS 8](https://docs.microsoft.com/en-us/previous-versions/windows/it-pro/windows-server-2012-R2-and-2012/jj635855(v=ws.11))
    - [CIS Microsoft IIS Benchmarks](https://www.cisecurity.org/benchmark/microsoft_iis/)
    - Sécurisation de votre serveur Web (modèles et pratiques), Microsoft Corporation, janvier 2004
    - Contre-mesures de sécurité et de programmation IIS, par Jason Coombs
    - From Blueprint to Fortress : A Guide to Securing IIS 5.0, par John Davis, Microsoft Corporation, juin 2001
    - Liste de contrôle Secure Internet Information Services 5, par Michael Howard, Microsoft Corporation, juin 2000
- iPlanet de Red Hat (anciennement Netscape)
    - Guide de configuration et d'administration sécurisées d'iPlanet Web Server, Enterprise Edition 4.1, par James M Hayes, l'équipe des applications réseau du Systems and Network Attack Center (SNAC), NSA, janvier 2001
- WebSphere
    - IBM WebSphere V5.0 Security, WebSphere Handbook Series, par Peter Kovari et al., IBM, décembre 2002.
    - IBM WebSphere V4.0 Advanced Edition Security, par Peter Kovari et al., IBM, mars 2002.
- Général
    - [Feuille de triche de journalisation] (https://cheatsheetseries.owasp.org/cheatsheets/Logging_Cheat_Sheet.html), OWASP
    - [SP 800-92](https://csrc.nist.gov/publications/detail/sp/800-92/final) Guide de gestion des journaux de sécurité informatique, NIST
    - [PCI DSS v3.2.1](https://www.pcisecuritystandards.org/document_library) Exigence 10 et PA-DSS v3.2 Exigence 4, Conseil des normes de sécurité PCI

- Générique:
    - [Modules d'amélioration de la sécurité CERT : sécurisation des serveurs Web publics] (https://resources.sei.cmu.edu/asset_files/SecurityImprovementModule/2000_006_001_13637.pdf)
