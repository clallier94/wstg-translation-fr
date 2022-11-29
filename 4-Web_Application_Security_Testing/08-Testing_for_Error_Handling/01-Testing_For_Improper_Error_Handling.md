# Test de la gestion incorrecte des erreurs

|ID          |
|------------|
|WSTG-ERRH-01|

## Sommaire

Tous les types d'applications (applications Web, serveurs Web, bases de données, etc.) généreront des erreurs pour diverses raisons. Les développeurs ignorent souvent la gestion de ces erreurs ou repoussent l'idée qu'un utilisateur essaiera un jour de déclencher une erreur à dessein (*par exemple* en envoyant une chaîne là où un entier est attendu). Lorsque le développeur ne considère que le chemin heureux, il oublie toutes les autres entrées utilisateur possibles que le code peut recevoir mais ne peut pas gérer.

Les erreurs augmentent parfois comme :

- traces de pile,
- les timeouts du réseau,
- décalage d'entrée,
- et les vidages mémoire.

Une mauvaise gestion des erreurs peut permettre aux attaquants de :

- Comprendre les API utilisées en interne.
- Cartographier les différents services s'intégrant les uns aux autres en obtenant un aperçu des systèmes internes et des cadres utilisés, ce qui ouvre la porte à l'enchaînement des attaques.
- Recueillir les versions et les types d'applications utilisées.
- DoS le système en forçant le système dans une impasse ou une exception non gérée qui envoie un signal de panique au moteur qui l'exécute.
- Contrôle le contournement lorsqu'une certaine exception n'est pas limitée par la logique définie autour du chemin heureux.

## Objectifs des tests

- Identifier la sortie d'erreur existante.
- Analyser les différentes sorties retournées.

## Comment tester

Les erreurs sont généralement considérées comme bénignes car elles fournissent des données de diagnostic et des messages qui pourraient aider l'utilisateur à comprendre le problème en question ou permettre au développeur de déboguer cette erreur.

En essayant d'envoyer des données inattendues ou en forçant le système dans certains cas et scénarios extrêmes, le système ou l'application dévoilera la plupart du temps un peu ce qui se passe en interne, à moins que les développeurs ne désactivent toutes les erreurs possibles et ne renvoient un certain message personnalisé. .

### Serveurs Web

Toutes les applications Web s'exécutent sur un serveur Web, qu'il soit intégré ou à part entière. Les applications Web doivent gérer et analyser les requêtes HTTP, et pour cela, un serveur Web fait toujours partie de la pile. Certains des serveurs Web les plus connus sont NGINX, Apache et IIS.

Les serveurs Web ont des messages d'erreur et des formats connus. Si l'on n'est pas familier avec leur apparence, une recherche en ligne pour eux fournirait des exemples. Une autre façon serait de consulter leur documentation ou simplement de configurer un serveur localement et de découvrir les erreurs en parcourant les pages utilisées par le serveur Web.

Pour déclencher des messages d'erreur, un testeur doit :

- Rechercher des fichiers et dossiers aléatoires qui ne seront pas trouvés (404s).
- Essayez de demander des dossiers qui existent et voyez le comportement du serveur (403, page blanche ou liste de répertoires).
- Essayez d'envoyer une requête qui rompt le [HTTP RFC](https://tools.ietf.org/html/rfc7231). Un exemple serait d'envoyer un très grand chemin, de casser le format des en-têtes ou de changer la version HTTP.
    - Même si les erreurs sont gérées au niveau de l'application, la rupture de la RFC HTTP peut faire apparaître le serveur Web intégré puisqu'il doit gérer la requête, et les développeurs oublient de remplacer ces erreurs.

### Applications

Les applications sont les plus susceptibles d'émettre une grande variété de messages d'erreur, notamment : des traces de pile, des vidages mémoire, des exceptions mal gérées et des erreurs génériques. Cela est dû au fait que les applications sont construites sur mesure la plupart du temps et que les développeurs doivent observer et gérer tous les cas d'erreur possibles (ou disposer d'un mécanisme global de capture d'erreurs), et ces erreurs peuvent apparaître à partir d'intégrations avec d'autres services.

Pour qu'une application génère ces erreurs, un testeur doit :

1. Identifiez les points d'entrée possibles où l'application attend des données.
2. Analysez le type d'entrée attendu (chaînes, entiers, JSON, XML, etc.).
3. Fuzzez chaque point d'entrée en fonction des étapes précédentes pour avoir un scénario de test plus ciblé.
   - Fuzzer chaque entrée avec toutes les injections possibles n'est pas la meilleure solution, sauf si vous disposez d'un temps de test illimité et que l'application peut gérer autant d'entrées.
   - Si le fuzzing n'est pas une option, sélectionnez manuellement les entrées viables qui ont le plus de chances de casser un certain analyseur (*par exemple* un crochet fermant pour un corps JSON, un texte volumineux où seuls quelques caractères sont attendus, injection CLRF avec paramètres qui pourraient être analysés par les serveurs et les contrôles de validation d'entrée, les caractères spéciaux qui ne s'appliquent pas aux noms de fichiers, etc.).
   - Le fuzzing avec des données de jargon doit être exécuté pour chaque type, car parfois les interpréteurs sortiront de la gestion des exceptions du développeur.
4. Comprenez le service qui répond avec le message d'erreur et essayez de créer une liste fuzz plus précise pour faire ressortir plus d'informations ou de détails sur l'erreur de ce service (il peut s'agir d'une base de données, d'un service autonome, etc.).

Les messages d'erreur sont parfois la principale faiblesse de la cartographie des systèmes, en particulier sous une architecture de microservices. Si les services ne sont pas correctement configurés pour gérer les erreurs de manière générique et uniforme, les messages d'erreur permettent à un testeur d'identifier quel service gère quelles requêtes et permettent une attaque plus ciblée par service.

> Le testeur doit garder un œil vigilant sur le type de réponse. Parfois, les erreurs sont renvoyées comme un succès avec un corps d'erreur, masquent l'erreur dans un 302 ou simplement en ayant une manière personnalisée de représenter cette erreur.

## Correction

Pour remédier aux problèmes, consultez les [Contrôles proactifs C10](https://owasp.org/www-project-proactive-controls/v3/en/c10-errors-exceptions) et la [Fiche de triche pour la gestion des erreurs](https:/ /cheatsheetseries.owasp.org/cheatsheets/Error_Handling_Cheat_Sheet.html).

## Cours de récréation

- [Juice Shop - Gestion des erreurs](https://pwning.owasp-juice.shop/part2/security-misconfiguration.html#provoke-an-error-that-is-nither-very-gracefully-ni-consistently-handled )

## Références

- [WSTG : Annexe C - Vecteurs Fuzz](../../6-Appendix/C-Fuzz_Vectors.md)
- [Contrôles proactifs C10 : gérer toutes les erreurs et exceptions] (https://owasp.org/www-project-proactive-controls/v3/en/c10-errors-exceptions)
- [ASVS v4.1 v7.4 : gestion des erreurs](https://github.com/OWASP/ASVS/blob/master/4.0/en/0x15-V7-Error-Logging.md#v74-error-handling)
- [CWE 728 - Traitement incorrect des erreurs](https://cwe.mitre.org/data/definitions/728.html)
- [Cheat Sheet Series : Gestion des erreurs] (https://cheatsheetseries.owasp.org/cheatsheets/Error_Handling_Cheat_Sheet.html)
