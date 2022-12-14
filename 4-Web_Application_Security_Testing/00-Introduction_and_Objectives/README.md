# 4.0 Introduction et objectifs

Cette section décrit la méthodologie de test de sécurité des applications Web OWASP et explique comment tester les preuves de vulnérabilités au sein de l'application en raison de lacunes dans les contrôles de sécurité identifiés.

## Qu'est-ce que le test de sécurité des applications Web ?

Un test de sécurité est une méthode d'évaluation de la sécurité d'un système informatique ou d'un réseau en validant et en vérifiant méthodiquement l'efficacité des contrôles de sécurité des applications. Un test de sécurité d'application Web se concentre uniquement sur l'évaluation de la sécurité d'une application Web. Le processus implique une analyse active de l'application pour détecter toute faiblesse, défaut technique ou vulnérabilité. Tout problème de sécurité détecté sera présenté au propriétaire du système, accompagné d'une évaluation de l'impact, d'une proposition d'atténuation ou d'une solution technique.

## Qu'est-ce qu'une vulnérabilité ?

Une vulnérabilité est une faille ou une faiblesse dans la conception, la mise en œuvre, le fonctionnement ou la gestion d'un système qui pourrait être exploitée pour compromettre les objectifs de sécurité du système.

## Qu'est-ce qu'une menace ?

Une menace est tout (un attaquant externe malveillant, un utilisateur interne, une instabilité du système, etc.) ce qui peut nuire aux actifs détenus par une application (ressources de valeur, telles que les données dans une base de données ou dans le système de fichiers) en exploitant une vulnérabilité.

## Qu'est-ce qu'un essai ?

Un test est une action visant à démontrer qu'une application répond aux exigences de sécurité de ses parties prenantes.

## L'approche dans la rédaction de ce guide

L'approche OWASP est ouverte et collaborative :

- Ouvert : chaque expert en sécurité peut participer avec son expérience au projet. Tout est gratuit.
- Collaboratif : un brainstorming est effectué avant la rédaction des articles afin que l'équipe puisse partager des idées et développer une vision collective du projet. Cela signifie un consensus approximatif, un public plus large et une participation accrue.

Cette approche tend à créer une méthodologie de test définie qui sera :

- Cohérent
- Reproductible
- Rigoureux
- Sous contrôle qualité

Les problèmes à résoudre sont entièrement documentés et testés. Il est important d'utiliser une méthode pour tester toutes les vulnérabilités connues et documenter toutes les activités de test de sécurité.

## Qu'est-ce que la méthodologie de test OWASP ?

Les tests de sécurité ne seront jamais une science exacte où une liste complète de tous les problèmes possibles qui devraient être testés peut être définie. En effet, les tests de sécurité ne sont une technique appropriée pour tester la sécurité des applications Web que dans certaines circonstances. Le but de ce projet est de rassembler toutes les techniques de test possibles, d'expliquer ces techniques et de maintenir le guide à jour. La méthode de test de sécurité des applications Web OWASP est basée sur l'approche de la boîte noire. Le testeur ne sait rien ou a très peu d'informations sur l'application à tester.

Le modèle de test se compose de :

- Testeur : Qui effectue les activités de test
- Outils et méthodologie : le cœur de ce projet de guide de test
- Application : La boîte noire à tester

Les tests peuvent être classés comme passifs ou actifs :

### Tests passifs

Lors des tests passifs, un testeur essaie de comprendre la logique de l'application et explore l'application en tant qu'utilisateur. Des outils peuvent être utilisés pour la collecte d'informations. Par exemple, un proxy HTTP peut être utilisé pour observer toutes les requêtes et réponses HTTP. À la fin de cette phase, le testeur doit généralement comprendre tous les points d'accès et toutes les fonctionnalités du système (par exemple, les en-têtes HTTP, les paramètres, les cookies, les API, l'utilisation/les modèles de technologie, etc.). La section [Collecte d'informations](../01-Information_Gathering/README.md) explique comment effectuer des tests passifs.

Par exemple, un testeur peut trouver une page à l'URL suivante : `https://www.exemple.com/login/auth_form`

Cela peut indiquer un formulaire d'authentification où l'application demande un nom d'utilisateur et un mot de passe.

Les paramètres suivants représentent deux points d'accès à l'application : `https://www.exemple.com/appx?a=1&b=1`

Dans ce cas, l'application affiche deux points d'accès (paramètres 'a' et 'b'). Tous les points d'entrée trouvés dans cette phase représentent une cible pour les tests. Garder une trace du répertoire ou de l'arborescence des appels de l'application et de tous les points d'accès peut être utile pendant les tests actifs.

### Tests actifs

Pendant les tests actifs, un testeur commence à utiliser les méthodologies décrites dans les sections suivantes.

L'ensemble des tests actifs a été divisé en 12 catégories :

- La collecte d'informations
- Tests de gestion de configuration et de déploiement
- Tests de gestion d'identité
- Tests d'authentification
- Tests d'autorisation
- Test de gestion de session
- Test de validation des entrées
- La gestion des erreurs
- Cryptographie
- Test de logique métier
- Tests côté client
- Tests d'API
