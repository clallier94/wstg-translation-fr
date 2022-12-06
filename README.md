# Table des matières

## 0. [Préface par Eoin Keary](0-Foreword/README.md)

## 1. [Frontispice](1-Frontispiece/)

## 2. [Introduction](2-Introduction/)

### 2.1 [Le projet de test OWASP](2-Introduction/README.md#Le-projet-de-test-OWASP)

### 2.2 [Principes de test](2-Introduction/README.md#Principes-de-test)

### 2.3 [Techniques de test expliquées](2-Introduction/README.md#Techniques-de-test-expliquées)

### 2.4 [Inspections et révisions manuelles](2-Introduction/README.md#Inspections-et-révisions-manuelles)

### 2.5 [Modélisation des menaces](2-Introduction/README.md#Modélisation-des-menaces)

### 2.6 [Examen du code source](2-Introduction/README.md#Examen-du-code-source)

### 2.7 [Tests de pénétration](2-Introduction/README.md#Tests-de-pénétration)

### 2.8 [La nécessité d'une approche équilibrée](2-Introduction/README.md#la-n%C3%A9cessit%C3%A9-dune-approche-%C3%A9quilibr%C3%A9e)

### 2.9 [Dérivation des exigences de test de sécurité](2-Introduction/README.md#Dérivation-des-exigences-de-test-de-sécurité)

### 2.10 [Tests de sécurité intégrés dans les workflows de développement et de test](2-Introduction/README.md#Tests-de-sécurité-intégrés-dans-les-workflows-de-développement-et-de-test)

### 2.11 [Analyse des données de test de sécurité et création de rapports](2-Introduction/README.md#Analyse-des-données-de-test-de-sécurité-et-création-de-rapports)

## 3. [Le cadre de test OWASP](3-The_OWASP_Testing_Framework/)

### 3.1 [Le cadre de test de sécurité Web](3-The_OWASP_Testing_Framework/0-The_Web_Security_Testing_Framework.md)

### 3.2 [Phase 1 avant le début du développement](3-The_OWASP_Testing_Framework/0-The_Web_Security_Testing_Framework.md#Phase-1-avant-le-début-du-développement)

### 3.3 [Phase 2 pendant la définition et la conception](3-The_OWASP_Testing_Framework/0-The_Web_Security_Testing_Framework.md#Phase-2-pendant-la-définition-et-la-conception)

### 3.4 [Phase 3 pendant le développement](3-The_OWASP_Testing_Framework/0-The_Web_Security_Testing_Framework.md#Phase-3-pendant-le-développement)

### 3.5 [Phase 4 pendant le déploiement](3-The_OWASP_Testing_Framework/0-The_Web_Security_Testing_Framework.md#Phase-4-pendant-le-déploiement)

### 3.6 [Phase 5 pendant la maintenance et les opérations](3-The_OWASP_Testing_Framework/0-The_Web_Security_Testing_Framework.md#Phase-5-pendant-la-maintenance-et-les-opérations)

### 3.7 [Un flux de travail de test SDLC typique](3-The_OWASP_Testing_Framework/0-The_Web_Security_Testing_Framework.md#Un-flux-de-travail-de-test-SDLC-typique)

### 3.8 [Méthodologies des tests d'intrusion](3-The_OWASP_Testing_Framework/1-Penetration_Testing_Methodologies.md)

## 4. [Test de sécurité des applications Web](4-Web_Application_Security_Testing/)

### 4.0 [Présentation et objectifs](4-Web_Application_Security_Testing/00-Introduction_and_Objectives/README.md)

### 4.1 [La collecte d'informations](4-Web_Application_Security_Testing/01-Information_Gathering/README.md)

#### 4.1.1 [Effectuer une reconnaissance de découverte de moteur de recherche pour les fuites d'informations](4-Web_Application_Security_Testing/01-Information_Gathering/01-Conduct_Search_Engine_Discovery_Reconnaissance_for_Information_Leakage.md)

#### 4.1.2 [Serveur Web d'empreintes digitales](4-Web_Application_Security_Testing/01-Information_Gathering/02-Fingerprint_Web_Server.md)

#### 4.1.3 [Examiner les métafichiers du serveur Web pour détecter les fuites d'informations](4-Web_Application_Security_Testing/01-Information_Gathering/03-Review_Webserver_Metafiles_for_Information_Leakage.md)

#### 4.1.4 [Énumérer les applications sur le serveur Web](4-Web_Application_Security_Testing/01-Information_Gathering/04-Enumerate_Applications_on_Webserver.md)

#### 4.1.5 [Examiner le contenu de la page Web pour détecter les fuites d'informations](4-Web_Application_Security_Testing/01-Information_Gathering/05-Review_Webpage_Content_for_Information_Leakage.md)

#### 4.1.6 [Identifier les points d'entrée de l'application](4-Web_Application_Security_Testing/01-Information_Gathering/06-Identify_Application_Entry_Points.md)

#### 4.1.7 [Mapper les chemins d'exécution via l'application](4-Web_Application_Security_Testing/01-Information_Gathering/07-Map_Execution_Paths_Through_Application.md)

#### 4.1.8 [Cadre d'application Web d'empreintes digitales](4-Web_Application_Security_Testing/01-Information_Gathering/08-Fingerprint_Web_Application_Framework.md)

#### 4.1.9 [Application Web d'empreintes digitales](4-Web_Application_Security_Testing/01-Information_Gathering/09-Fingerprint_Web_Application.md)

#### 4.1.10 [Architecture des applications cartographiques](4-Web_Application_Security_Testing/01-Information_Gathering/10-Map_Application_Architecture.md)

### 4.2 [Test de gestion de la configuration et du déploiement](4-Web_Application_Security_Testing/02-Configuration_and_Deployment_Management_Testing/README.md)

#### 4.2.1 [Tester la configuration de l'infrastructure réseau](4-Web_Application_Security_Testing/02-Configuration_and_Deployment_Management_Testing/01-Test_Network_Infrastructure_Configuration.md)

#### 4.2.2 [Tester la configuration de la plate-forme d'application](4-Web_Application_Security_Testing/02-Configuration_and_Deployment_Management_Testing/02-Test_Application_Platform_Configuration.md)

#### 4.2.3 [Tester la gestion des extensions de fichiers pour les informations sensibles](4-Web_Application_Security_Testing/02-Configuration_and_Deployment_Management_Testing/03-Test_File_Extensions_Handling_for_Sensitive_Information.md)

#### 4.2.4 [Examiner l'ancienne sauvegarde et les fichiers non référencés pour les informations sensibles](4-Web_Application_Security_Testing/02-Configuration_and_Deployment_Management_Testing/04-Review_Old_Backup_and_Unreferenced_Files_for_Sensitive_Information.md)

#### 4.2.5 [Énumérer les interfaces d'administration d'infrastructure et d'application](4-Web_Application_Security_Testing/02-Configuration_and_Deployment_Management_Testing/05-Enumerate_Infrastructure_and_Application_Admin_Interfaces.md)

#### 4.2.6 [Tester les méthodes HTTP](4-Web_Application_Security_Testing/02-Configuration_and_Deployment_Management_Testing/06-Test_HTTP_Methods.md)

#### 4.2.7 [Tester la sécurité du transport strict HTTP](4-Web_Application_Security_Testing/02-Configuration_and_Deployment_Management_Testing/07-Test_HTTP_Strict_Transport_Security.md)

#### 4.2.8 [Tester la stratégie interdomaine RIA](4-Web_Application_Security_Testing/02-Configuration_and_Deployment_Management_Testing/08-Test_RIA_Cross_Domain_Policy.md)

#### 4.2.9 [Autorisation de fichier de test](4-Web_Application_Security_Testing/02-Configuration_and_Deployment_Management_Testing/09-Test_File_Permission.md)

#### 4.2.10 [Test de reprise de sous-domaine](4-Web_Application_Security_Testing/02-Configuration_and_Deployment_Management_Testing/10-Test_for_Subdomain_Takeover.md)

#### 4.2.11 [Tester le stockage en nuage](4-Web_Application_Security_Testing/02-Configuration_and_Deployment_Management_Testing/11-Test_Cloud_Storage.md)

#### 4.2.12 [Tester la politique de sécurité du contenu](4-Web_Application_Security_Testing/02-Configuration_and_Deployment_Management_Testing/12-Test_for_Content_Security_Policy.md)

### 4.3 [Test de gestion des identités](4-Web_Application_Security_Testing/03-Identity_Management_Testing/README.md)

#### 4.3.1 [Tester les définitions de rôle](4-Web_Application_Security_Testing/03-Identity_Management_Testing/01-Test_Role_Definitions.md)

#### 4.3.2 [Tester le processus d'enregistrement des utilisateurs](4-Web_Application_Security_Testing/03-Identity_Management_Testing/02-Test_User_Registration_Process.md)

#### 4.3.3 [Tester le processus de provisionnement de compte](4-Web_Application_Security_Testing/03-Identity_Management_Testing/03-Test_Account_Provisioning_Process.md)

#### 4.3.4 [Test d'énumération de compte et de compte d'utilisateur devinable](4-Web_Application_Security_Testing/03-Identity_Management_Testing/04-Testing_for_Account_Enumeration_and_Guessable_User_Account.md)

#### 4.3.5 [Test de la politique de nom d'utilisateur faible ou non appliquée](4-Web_Application_Security_Testing/03-Identity_Management_Testing/05-Testing_for_Weak_or_Unenforced_Username_Policy.md)

### 4.4 [Test d'authentification](4-Web_Application_Security_Testing/04-Authentication_Testing/README.md)

#### 4.4.1 [Test des informations d'identification transportées sur un canal crypté](4-Web_Application_Security_Testing/04-Authentication_Testing/01-Testing_for_Credentials_Transported_over_an_Encrypted_Channel.md)

#### 4.4.2 [Test des informations d'identification par défaut](4-Web_Application_Security_Testing/04-Authentication_Testing/02-Testing_for_Default_Credentials.md)

#### 4.4.3 [Test de mécanisme de verrouillage faible](4-Web_Application_Security_Testing/04-Authentication_Testing/03-Testing_for_Weak_Lock_Out_Mechanism.md)

#### 4.4.4 [Test de contournement du schéma d'authentification](4-Web_Application_Security_Testing/04-Authentication_Testing/04-Testing_for_Bypassing_Authentication_Schema.md)

#### 4.4.5 [Test de mémorisation du mot de passe vulnérable](4-Web_Application_Security_Testing/04-Authentication_Testing/05-Testing_for_Vulnerable_Remember_Password.md)

#### 4.4.6 [Test des faiblesses du cache du navigateur](4-Web_Application_Security_Testing/04-Authentication_Testing/06-Testing_for_Browser_Cache_Weaknesses.md)

#### 4.4.7 [Test de la politique de mot de passe faible](4-Web_Application_Security_Testing/04-Authentication_Testing/07-Testing_for_Weak_Password_Policy.md)

#### 4.4.8 [Test de réponse à la question de sécurité faible](4-Web_Application_Security_Testing/04-Authentication_Testing/08-Testing_for_Weak_Security_Question_Answer.md)

#### 4.4.9 [Test des fonctionnalités de modification ou de réinitialisation de mot de passe faibles](4-Web_Application_Security_Testing/04-Authentication_Testing/09-Testing_for_Weak_Password_Change_or_Reset_Functionalities.md)

#### 4.4.10 [Test d'authentification plus faible dans un canal alternatif](4-Web_Application_Security_Testing/04-Authentication_Testing/10-Testing_for_Weaker_Authentication_in_Alternative_Channel.md)

#### 4.4.11 [Test de l'authentification multifacteur](4-Web_Application_Security_Testing/04-Authentication_Testing/11-Testing_Multi-Factor_Authentication.md)

### 4.5 [Test d'autorisation](4-Web_Application_Security_Testing/05-Authorization_Testing/README.md)

#### 4.5.1 [Tester l'inclusion du fichier de traversée de répertoire](4-Web_Application_Security_Testing/05-Authorization_Testing/01-Testing_Directory_Traversal_File_Include.md)

#### 4.5.2 [Test de contournement du schéma d'autorisation](4-Web_Application_Security_Testing/05-Authorization_Testing/02-Testing_for_Bypassing_Authorization_Schema.md)

#### 4.5.3 [Test d'élévation des privilèges](4-Web_Application_Security_Testing/05-Authorization_Testing/03-Testing_for_Privilege_Escalation.md)

#### 4.5.4 [Test des références d'objets directes non sécurisées](4-Web_Application_Security_Testing/05-Authorization_Testing/04-Testing_for_Insecure_Direct_Object_References.md)

#### 4.5.5 [Test des faiblesses OAuth](4-Web_Application_Security_Testing/05-Authorization_Testing/05-Testing_for_OAuth_Weaknesses.md)

##### 4.5.5.1 [Test des faiblesses du serveur d'autorisation OAuth](4-Web_Application_Security_Testing/05-Authorization_Testing/05.1-Testing_for_OAuth_Authorization_Server_Weaknesses.md)

##### 4.5.5.2 [Test des faiblesses du client OAuth](4-Web_Application_Security_Testing/05-Authorization_Testing/05.2-Testing_for_OAuth_Client_Weaknesses.md)

### 4.6 [Test de gestion de session](4-Web_Application_Security_Testing/06-Session_Management_Testing/README.md)

#### 4.6.1 [Test du schéma de gestion de session](4-Web_Application_Security_Testing/06-Session_Management_Testing/01-Testing_for_Session_Management_Schema.md)

#### 4.6.2 [Test des attributs des cookies](4-Web_Application_Security_Testing/06-Session_Management_Testing/02-Testing_for_Cookies_Attributes.md)

#### 4.6.3 [Test de la fixation de session](4-Web_Application_Security_Testing/06-Session_Management_Testing/03-Testing_for_Session_Fixation.md)

#### 4.6.4 [Test des variables de session exposées](4-Web_Application_Security_Testing/06-Session_Management_Testing/04-Testing_for_Exposed_Session_Variables.md)

#### 4.6.5 [Test de falsification de demande intersite](4-Web_Application_Security_Testing/06-Session_Management_Testing/05-Testing_for_Cross_Site_Request_Forgery.md)

#### 4.6.6 [Test de la fonctionnalité de déconnexion](4-Web_Application_Security_Testing/06-Session_Management_Testing/06-Testing_for_Logout_Functionality.md)

#### 4.6.7 [Délai de session de test](4-Web_Application_Security_Testing/06-Session_Management_Testing/07-Testing_Session_Timeout.md)

#### 4.6.8 [Test de Session Puzzling](4-Web_Application_Security_Testing/06-Session_Management_Testing/08-Testing_for_Session_Puzzling.md)

#### 4.6.9 [Test de piratage de session](4-Web_Application_Security_Testing/06-Session_Management_Testing/09-Testing_for_Session_Hijacking.md)

#### 4.6.10 [Test des jetons Web JSON](4-Web_Application_Security_Testing/06-Session_Management_Testing/10-Testing_JSON_Web_Tokens.md)

### 4.7 [Test de validation des entrées](4-Web_Application_Security_Testing/07-Input_Validation_Testing/README.md)

#### 4.7.1 [Test pour les scripts intersites réfléchis](4-Web_Application_Security_Testing/07-Input_Validation_Testing/01-Testing_for_Reflected_Cross_Site_Scripting.md)

#### 4.7.2 [Test pour les scripts intersites stockés](4-Web_Application_Security_Testing/07-Input_Validation_Testing/02-Testing_for_Stored_Cross_Site_Scripting.md)

#### 4.7.3 [Test de falsification de verbe HTTP](4-Web_Application_Security_Testing/07-Input_Validation_Testing/03-Testing_for_HTTP_Verb_Tampering.md)

#### 4.7.4 [Test de la pollution des paramètres HTTP](4-Web_Application_Security_Testing/07-Input_Validation_Testing/04-Testing_for_HTTP_Parameter_Pollution.md)

#### 4.7.5 [Test pour l'injection SQL](4-Web_Application_Security_Testing/07-Input_Validation_Testing/05-Testing_for_SQL_Injection.md)

##### 4.7.5.1 [Test pour Oracle](4-Web_Application_Security_Testing/07-Input_Validation_Testing/05.1-Testing_for_Oracle.md)

##### 4.7.5.2 [Test pour MySQL](4-Web_Application_Security_Testing/07-Input_Validation_Testing/05.2-Testing_for_MySQL.md)

##### 4.7.5.3 [Test pour SQL Server](4-Web_Application_Security_Testing/07-Input_Validation_Testing/05.3-Testing_for_SQL_Server.md)

##### 4.7.5.4 [Tester PostgreSQL](4-Web_Application_Security_Testing/07-Input_Validation_Testing/05.4-Testing_PostgreSQL.md)

##### 4.7.5.5 [Test pour MS Access](4-Web_Application_Security_Testing/07-Input_Validation_Testing/05.5-Testing_for_MS_Access.md)

##### 4.7.5.6 [Test pour l'injection NoSQL](4-Web_Application_Security_Testing/07-Input_Validation_Testing/05.6-Testing_for_NoSQL_Injection.md)

##### 4.7.5.7 [Test d'injection ORM](4-Web_Application_Security_Testing/07-Input_Validation_Testing/05.7-Testing_for_ORM_Injection.md)

##### 4.7.5.8 [Test pour le côté client](4-Web_Application_Security_Testing/07-Input_Validation_Testing/05.8-Testing_for_Client-side.md)

#### 4.7.6 [Test pour l'injection LDAP](4-Web_Application_Security_Testing/07-Input_Validation_Testing/06-Testing_for_LDAP_Injection.md)

#### 4.7.7 [Test pour l'injection XML](4-Web_Application_Security_Testing/07-Input_Validation_Testing/07-Testing_for_XML_Injection.md)

#### 4.7.8 [Test pour l'injection SSI](4-Web_Application_Security_Testing/07-Input_Validation_Testing/08-Testing_for_SSI_Injection.md)

#### 4.7.9 [Test pour l'injection XPath](4-Web_Application_Security_Testing/07-Input_Validation_Testing/09-Testing_for_XPath_Injection.md)

#### 4.7.10 [Test pour l'injection IMAP SMTP](4-Web_Application_Security_Testing/07-Input_Validation_Testing/10-Testing_for_IMAP_SMTP_Injection.md)

#### 4.7.11 [Test pour l'injection de code](4-Web_Application_Security_Testing/07-Input_Validation_Testing/11-Testing_for_Code_Injection.md)

##### 4.7.11.1 [Test d'inclusion de fichier](4-Web_Application_Security_Testing/07-Input_Validation_Testing/11.1-Testing_for_File_Inclusion.md)

#### 4.7.12 [Test pour l'injection de commande](4-Web_Application_Security_Testing/07-Input_Validation_Testing/12-Testing_for_Command_Injection.md)

#### 4.7.13 [Test d'injection de chaîne de format](4-Web_Application_Security_Testing/07-Input_Validation_Testing/13-Testing_for_Format_String_Injection.md)

#### 4.7.14 [Test de la vulnérabilité incubée](4-Web_Application_Security_Testing/07-Input_Validation_Testing/14-Testing_for_Incubated_Vulnerability.md)

#### 4.7.15 [Test de la contrebande de fractionnement HTTP](4-Web_Application_Security_Testing/07-Input_Validation_Testing/15-Testing_for_HTTP_Splitting_Smuggling.md)

#### 4.7.16 [Test des requêtes entrantes HTTP](4-Web_Application_Security_Testing/07-Input_Validation_Testing/16-Testing_for_HTTP_Incoming_Requests.md)

#### 4.7.17 [Test d'injection d'en-tête d'hôte](4-Web_Application_Security_Testing/07-Input_Validation_Testing/17-Testing_for_Host_Header_Injection.md)

#### 4.7.18 [Test pour l'injection de modèle côté serveur](4-Web_Application_Security_Testing/07-Input_Validation_Testing/18-Testing_for_Server-side_Template_Injection.md)

#### 4.7.19 [Test de contrefaçon de requête côté serveur](4-Web_Application_Security_Testing/07-Input_Validation_Testing/19-Testing_for_Server-Side_Request_Forgery.md)

#### 4.7.20 [Test pour l'affectation en masse](4-Web_Application_Security_Testing/07-Input_Validation_Testing/20-Testing_for_Mass_Assignment.md)

### 4.8 [Test de la gestion des erreurs](4-Web_Application_Security_Testing/08-Testing_for_Error_Handling/README.md)

#### 4.8.1 [Test de la gestion incorrecte des erreurs](4-Web_Application_Security_Testing/08-Testing_for_Error_Handling/01-Testing_For_Improper_Error_Handling.md)

#### 4.8.2 [Test des traces de pile](4-Web_Application_Security_Testing/08-Testing_for_Error_Handling/02-Testing_for_Stack_Traces.md)

### 4.9 [Test de cryptographie faible](4-Web_Application_Security_Testing/09-Testing_for_Weak_Cryptography/README.md)

#### 4.9.1 [Test de sécurité de la couche de transport faible](4-Web_Application_Security_Testing/09-Testing_for_Weak_Cryptography/01-Testing_for_Weak_Transport_Layer_Security.md)

#### 4.9.2 [Test pour le rembourrage Oracle](4-Web_Application_Security_Testing/09-Testing_for_Weak_Cryptography/02-Testing_for_Padding_Oracle.md)

#### 4.9.3 [Test des informations sensibles envoyées via des canaux non cryptés](4-Web_Application_Security_Testing/09-Testing_for_Weak_Cryptography/03-Testing_for_Sensitive_Information_Sent_via_Unencrypted_Channels.md)

#### 4.9.4 [Test de chiffrement faible](4-Web_Application_Security_Testing/09-Testing_for_Weak_Cryptography/04-Testing_for_Weak_Encryption.md)

### 4.10 [Test de la logique métier](4-Web_Application_Security_Testing/10-Business_Logic_Testing/README.md)

#### 4.10.0 [Introduction à la logique métier](4-Web_Application_Security_Testing/10-Business_Logic_Testing/00-Introduction_to_Business_Logic.md)

#### 4.10.1 [Tester la validation des données de la logique métier](4-Web_Application_Security_Testing/10-Business_Logic_Testing/01-Test_Business_Logic_Data_Validation.md)

#### 4.10.2 [Tester la capacité de forger des requêtes](4-Web_Application_Security_Testing/10-Business_Logic_Testing/02-Test_Ability_to_Forge_Requests.md)

#### 4.10.3 [Tester les vérifications d'intégrité](4-Web_Application_Security_Testing/10-Business_Logic_Testing/03-Test_Integrity_Checks.md)

#### 4.10.4 [Tester la synchronisation du processus](4-Web_Application_Security_Testing/10-Business_Logic_Testing/04-Test_for_Process_Timing.md)

#### 4.10.5 [Tester les limites du nombre de fois qu'une fonction peut être utilisée](4-Web_Application_Security_Testing/10-Business_Logic_Testing/05-Test_Number_of_Times_a_Function_Can_Be_Used_Limits.md)

#### 4.10.6 [Test pour le contournement des flux de travail](4-Web_Application_Security_Testing/10-Business_Logic_Testing/06-Testing_for_the_Circumvention_of_Work_Flows.md)

#### 4.10.7 [Tester les défenses contre l'utilisation abusive des applications](4-Web_Application_Security_Testing/10-Business_Logic_Testing/07-Test_Defenses_Against_Application_Misuse.md)

#### 4.10.8 [Tester le téléchargement de types de fichiers inattendus](4-Web_Application_Security_Testing/10-Business_Logic_Testing/08-Test_Upload_of_Unexpected_File_Types.md)

#### 4.10.9 [Tester le téléchargement de fichiers malveillants](4-Web_Application_Security_Testing/10-Business_Logic_Testing/09-Test_Upload_of_Malicious_Files.md)

#### 4.10.10 [Tester la fonctionnalité de paiement](4-Web_Application_Security_Testing/10-Business_Logic_Testing/10-Test-Payment-Functionality.md)

### 4.11 [Test côté client](4-Web_Application_Security_Testing/11-Client-side_Testing/README.md)

#### 4.11.1 [Test pour les scripts intersites basés sur DOM](4-Web_Application_Security_Testing/11-Client-side_Testing/01-Testing_for_DOM-based_Cross_Site_Scripting.md)

##### 4.11.1.1 [Test de script intersite basé sur Self DOM](4-Web_Application_Security_Testing/11-Client-side_Testing/01.1-Testing_for_Self_DOM_Based_Cross_Site_Scripting.md)

#### 4.11.2 [Test de l'exécution de JavaScript](4-Web_Application_Security_Testing/11-Client-side_Testing/02-Testing_for_JavaScript_Execution.md)

#### 4.11.3 [Test pour l'injection HTML](4-Web_Application_Security_Testing/11-Client-side_Testing/03-Testing_for_HTML_Injection.md)

#### 4.11.4 [Test de la redirection d'URL côté client](4-Web_Application_Security_Testing/11-Client-side_Testing/04-Testing_for_Client-side_URL_Redirect.md)

#### 4.11.5 [Test pour l'injection CSS](4-Web_Application_Security_Testing/11-Client-side_Testing/05-Testing_for_CSS_Injection.md)

#### 4.11.6 [Test de la manipulation des ressources côté client](4-Web_Application_Security_Testing/11-Client-side_Testing/06-Testing_for_Client-side_Resource_Manipulation.md)

#### 4.11.7 [Tester le partage de ressources d'origine croisée](4-Web_Application_Security_Testing/11-Client-side_Testing/07-Testing_Cross_Origin_Resource_Sharing.md)

#### 4.11.8 [Test pour le clignotement intersite](4-Web_Application_Security_Testing/11-Client-side_Testing/08-Testing_for_Cross_Site_Flashing.md)

#### 4.11.9 [Test de détournement de clic](4-Web_Application_Security_Testing/11-Client-side_Testing/09-Testing_for_Clickjacking.md)

#### 4.11.10 [Tester WebSockets](4-Web_Application_Security_Testing/11-Client-side_Testing/10-Testing_WebSockets.md)

#### 4.11.11 [Test de la messagerie Web](4-Web_Application_Security_Testing/11-Client-side_Testing/11-Testing_Web_Messaging.md)

#### 4.11.12 [Test du stockage du navigateur](4-Web_Application_Security_Testing/11-Client-side_Testing/12-Testing_Browser_Storage.md)

#### 4.11.13 [Test d'inclusion de scripts intersites](4-Web_Application_Security_Testing/11-Client-side_Testing/13-Testing_for_Cross_Site_Script_Inclusion.md)

#### 4.11.14 [Test pour Reverse Tabnabbing](4-Web_Application_Security_Testing/11-Client-side_Testing/14-Testing_for_Reverse_Tabnabbing.md)

### 4.12 [Test API](4-Web_Application_Security_Testing/12-API_Testing/README.md)

#### 4.12.1 [Tester GraphQL](4-Web_Application_Security_Testing/12-API_Testing/01-Testing_GraphQL.md)

## 5. [Rapport](5-Rapport/README.md)

### 5.1 [Structure de rapport](5-Reporting/01-Reporting_Structure.md)

### 5.2 [Schémas de nommage](5-Reporting/02-Naming_Schemes.md)

## Annexe A. [Ressource sur les outils de test](6-Appendix/A-Testing_Tools_Resource.md)

## Annexe B. [Lecture suggérée](6-Appendix/B-Suggested_Reading.md)

## Annexe C. [Vecteurs Fuzz](6-Appendix/C-Fuzz_Vectors.md)

## Annexe D. [Injection codée](6-Appendix/D-Encoded_Injection.md)

## Annexe E. [Historique](6-Appendix/E-History.md)

## Annexe F. [Exploitation des outils de développement](6-Appendix/F-Leveraging_Dev_Tools.md)
