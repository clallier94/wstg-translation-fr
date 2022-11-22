# Énumérer les interfaces d'administration de l'infrastructure et de l'application

|ID          |
|------------|
|WSTG-CONF-05|

## Sommaire

Des interfaces administrateur peuvent être présentes dans l'application ou sur le serveur applicatif pour permettre à certains utilisateurs d'entreprendre des activités privilégiées sur le site. Des tests doivent être entrepris pour révéler si et comment cette fonctionnalité privilégiée peut être accessible par un utilisateur non autorisé ou standard.

Une application peut nécessiter une interface administrateur pour permettre à un utilisateur privilégié d'accéder à des fonctionnalités susceptibles d'apporter des modifications au fonctionnement du site. Ces modifications peuvent inclure :

- provisionnement du compte utilisateur
- conception et mise en page du site
- manipulation de données
- changements de configuration

Dans de nombreux cas, ces interfaces ne disposent pas de contrôles suffisants pour les protéger contre les accès non autorisés. Les tests visent à découvrir ces interfaces d'administration et à accéder aux fonctionnalités destinées aux utilisateurs privilégiés.

## Objectifs des tests

- Identifiez les interfaces et les fonctionnalités cachées de l'administrateur.

## Comment tester

### Test de la boîte noire

La section suivante décrit les vecteurs qui peuvent être utilisés pour tester la présence d'interfaces administratives. Ces techniques peuvent également être utilisées pour tester des problèmes connexes, y compris l'élévation des privilèges, et sont décrites ailleurs dans ce guide (par exemple [Test pour contourner le schéma d'autorisation](../05-Authorization_Testing/02-Testing_for_Bypassing_Authorization_Schema.md) et [Test pour Références d'objets directes non sécurisées] (../05-Authorization_Testing/04-Testing_for_Insecure_Direct_Object_References.md) plus en détail.

- Énumération des répertoires et des fichiers. Une interface administrative peut être présente mais pas visiblement disponible pour le testeur. Tenter de deviner le chemin de l'interface d'administration peut être aussi simple que de demander : */admin ou /administrator etc..* ou, dans certains scénarios, peut être révélé en quelques secondes à l'aide de [Google dorks](https://www.exploit-db .com/google-hacking-database).
- Il existe de nombreux outils disponibles pour effectuer un forçage brutal du contenu du serveur, consultez la section outils ci-dessous pour plus d'informations. Un testeur devra peut-être également identifier le nom de fichier de la page d'administration. La navigation forcée vers la page identifiée peut donner accès à l'interface.
- Commentaires et liens dans le code source. De nombreux sites utilisent un code commun qui est chargé pour tous les utilisateurs du site. En examinant toutes les sources envoyées au client, des liens vers la fonctionnalité d'administrateur peuvent être découverts et doivent être étudiés.
- Révision de la documentation du serveur et de l'application. Si le serveur d'applications ou l'application est déployé dans sa configuration par défaut, il peut être possible d'accéder à l'interface d'administration à l'aide des informations décrites dans la documentation de configuration ou d'aide. Les listes de mots de passe par défaut doivent être consultées si une interface administrative est trouvée et que des informations d'identification sont requises.
- Informations accessibles au public. De nombreuses applications telles que WordPress ont des interfaces d'administration par défaut.
- Port de serveur alternatif. Les interfaces d'administration peuvent être vues sur un port différent sur l'hôte que l'application principale. Par exemple, l'interface d'administration d'Apache Tomcat est souvent visible sur le port 8080.
- Altération des paramètres. Un paramètre GET ou POST ou une variable de cookie peut être requis pour activer la fonctionnalité d'administrateur. Les indices à cela incluent la présence de champs cachés tels que :

```html
<input type="hidden" name="admin" value="no">
```

ou dans un cookie :

`Cookie: session_cookie; useradmin=0`

Une fois qu'une interface administrative a été découverte, une combinaison des techniques ci-dessus peut être utilisée pour tenter de contourner l'authentification. Si cela échoue, le testeur peut souhaiter tenter une attaque par force brute. Dans un tel cas, le testeur doit être conscient du risque de verrouillage du compte administratif si une telle fonctionnalité est présente.

### Test de la boîte grise

Un examen plus détaillé des composants du serveur et de l'application doit être entrepris pour garantir le renforcement (c'est-à-dire que les pages de l'administrateur ne sont pas accessibles à tous via l'utilisation du filtrage IP ou d'autres contrôles) et, le cas échéant, la vérification que tous les composants n'utilisent pas les informations d'identification par défaut ou configurations.
Le code source doit être revu pour s'assurer que le modèle d'autorisation et d'authentification assure une séparation claire des tâches entre les utilisateurs normaux et les administrateurs du site. Les fonctions d'interface utilisateur partagées entre les utilisateurs normaux et les administrateurs doivent être revues pour assurer une séparation claire entre le dessin de ces composants et les fuites d'informations provenant de ces fonctionnalités partagées.

Chaque infrastructure Web peut avoir ses propres pages ou chemin d'administration par défaut. Par exemple

WebSphere :

```html
/admin
/admin-authz.xml
/admin.conf
/admin.passwd
/admin/*
/admin/logon.jsp
/admin/secure/logon.jsp
```

PHP :

```html
/phpinfo
/phpmyadmin/
/phpMyAdmin/
/mysqladmin/
/MySQLadmin
/MySQLAdmin
/login.php
/logon.php
/xmlrpc.php
/dbadmin
```

FrontPage :

```html
/admin.dll
/admin.exe
/administrators.pwd
/author.dll
/author.exe
/author.log
/authors.pwd
/cgi-bin
```

WebLogic :

```html
/AdminCaptureRootCA
/AdminClients
/AdminConnections
/AdminEvents
/AdminJDBC
/AdminLicense
/AdminMain
/AdminProps
/AdminRealm
/AdminThreads
```

WordPress :

```html
wp-admin/
wp-admin/about.php
wp-admin/admin-ajax.php
wp-admin/admin-db.php
wp-admin/admin-footer.php
wp-admin/admin-functions.php
wp-admin/admin-header.php
```

## Outils

- [OWASP ZAP - Forced Browse] (https://www.zaproxy.org/docs/desktop/addons/forced-browse/) est une utilisation actuellement maintenue du précédent projet DirBuster d'OWASP.
- [THC-HYDRA](https://github.com/vanhauser-thc/thc-hydra) est un outil qui permet de forcer brutalement de nombreuses interfaces, y compris l'authentification HTTP basée sur des formulaires.
- Un "brute force" est bien meilleur quand il utilise un bon dictionnaire, par exemple le dictionnaire [netsparker](https://www.netsparker.com/blog/web-security/svn-digger-better-lists-for-forced-browsing/).

## Références

- [Cirt : liste de mots de passe par défaut] (https://cirt.net/passwords)
- [FuzzDB peut être utilisé pour parcourir par force brute le chemin de connexion de l'administrateur] (https://github.com/fuzzdb-project/fuzzdb/blob/master/discovery/predictable-filepaths/login-file-locations/Logins.txt)
- [Paramètres d'administration ou de débogage communs] (https://github.com/fuzzdb-project/fuzzdb/blob/master/attack/business-logic/CommonDebugParamNames.txt)
