# Tester les défenses contre l'utilisation abusive des applications

|ID          |
|------------|
|WSTG-BUSL-07|

## Sommaire

L'utilisation abusive et invalide de fonctionnalités valides peut identifier les attaques tentant d'énumérer l'application Web, d'identifier les faiblesses et d'exploiter les vulnérabilités. Des tests doivent être entrepris pour déterminer s'il existe des mécanismes de défense de la couche application en place pour protéger l'application.

L'absence de défenses actives permet à un attaquant de chasser les vulnérabilités sans aucun recours. Le propriétaire de l'application ne saura donc pas que son application est attaquée.

### Exemple

Un utilisateur authentifié entreprend la séquence d'actions (improbable) suivante :

1. Tentative d'accès à un ID de fichier que leurs rôles ne sont pas autorisés à télécharger
2. Remplace une seule coche `'` au lieu du numéro d'identification du fichier
3. Modifie une requête GET en POST
4. Ajoute un paramètre supplémentaire
5. Duplique une paire nom/valeur de paramètre

L'application surveille les abus et répond après le 5ème événement avec une confiance extrêmement élevée que l'utilisateur est un attaquant. Par exemple l'application :

- Désactive les fonctionnalités critiques
- Active des étapes d'authentification supplémentaires pour les fonctionnalités restantes
- Ajoute des délais dans chaque cycle de demande-réponse
- Commence à enregistrer des données supplémentaires sur les interactions de l'utilisateur (par exemple, les en-têtes de requête HTTP nettoyés, les corps et les corps de réponse)

Si l'application ne répond pas de quelque manière que ce soit et que l'attaquant peut continuer à abuser des fonctionnalités et soumettre un contenu clairement malveillant à l'application, l'application a échoué à ce cas de test. En pratique, il est peu probable que les exemples d'actions discrètes de l'exemple ci-dessus se produisent ainsi. Il est beaucoup plus probable qu'un outil de fuzzing soit utilisé pour identifier tour à tour les faiblesses de chaque paramètre. C'est ce qu'un testeur de sécurité aura également entrepris.

## Objectifs des tests

- Générer des notes de tous les tests effectués contre le système.
- Passez en revue les tests qui avaient une fonctionnalité différente basée sur une entrée agressive.
- Comprendre les défenses en place et vérifier si elles suffisent à protéger le système contre les techniques de contournement.

## Comment tester

Ce test est inhabituel dans la mesure où le résultat peut être tiré de tous les autres tests effectués sur l'application Web. Lors de l'exécution de tous les autres tests, notez les mesures qui pourraient indiquer que l'application dispose d'une autodéfense intégrée :

- Réponses modifiées
- Requêtes bloquées
- Actions qui déconnectent un utilisateur ou verrouillent son compte

Celles-ci ne peuvent être que localisées. Les défenses localisées courantes (par fonction) sont :

- Rejeter l'entrée contenant certains caractères
- Verrouillage temporaire d'un compte après un certain nombre d'échecs d'authentification

Les contrôles de sécurité localisés ne suffisent pas. Il n'y a souvent aucune défense contre une mauvaise utilisation générale telle que :

- Navigation forcée
- Contournement de la validation des entrées de la couche de présentation
- Plusieurs erreurs de contrôle d'accès
- Noms de paramètres supplémentaires, dupliqués ou manquants
- Plusieurs échecs de validation des entrées ou de vérification de la logique métier avec des valeurs qui ne peuvent pas être le résultat d'erreurs ou de fautes de frappe de l'utilisateur
- Des données structurées (par exemple JSON, XML) d'un format invalide sont reçues
- Réception de scripts intersites flagrants ou de charges utiles d'injection SQL
- Utiliser l'application plus rapidement qu'il ne serait possible sans outils d'automatisation
- Changement de géolocalisation continentale d'un utilisateur
- Changement d'agent utilisateur
- Accéder à un processus métier en plusieurs étapes dans le mauvais ordre
- Un grand nombre ou un taux élevé d'utilisation de fonctionnalités spécifiques à l'application (par exemple, soumission de code de bon d'achat, échec des paiements par carte de crédit, téléchargements de fichiers, téléchargements de fichiers, déconnexions, etc.).

Ces défenses fonctionnent mieux dans les parties authentifiées de l'application, bien que le taux de création de nouveaux comptes ou d'accès au contenu (par exemple pour récupérer des informations) puisse être utile dans les zones publiques.

Tous les éléments ci-dessus n'ont pas besoin d'être surveillés par l'application, mais il y a un problème si aucun d'entre eux ne l'est. En testant l'application Web, en effectuant le type d'actions ci-dessus, une réponse a-t-elle été prise contre le testeur ? Si ce n'est pas le cas, le testeur doit signaler que l'application semble n'avoir aucune défense active à l'échelle de l'application contre les abus. Notez qu'il est parfois possible que toutes les réponses à la détection d'attaques soient silencieuses pour l'utilisateur (par exemple, modifications de journalisation, surveillance accrue, alertes aux administrateurs et demande de proxy), donc la confiance dans cette découverte ne peut être garantie. En pratique, très peu d'applications (ou d'infrastructures connexes telles qu'un pare-feu d'application Web) détectent ces types d'utilisation abusive.

## Cas de test associés

Tous les autres cas de test sont pertinents.

## Correction

Les applications doivent implémenter des défenses actives pour repousser les attaquants et les abuseurs.

## Références

- [Software Assurance](https://www.cisa.gov/uscert/sites/default/files/publications/infosheet_SoftwareAssurance.pdf), US Department Homeland Security
- [IR 7684](https://csrc.nist.gov/publications/detail/nistir/7864/final) Système commun de notation des abus (CMSS), NIST
- [Common Attack Pattern Enumeration and Classification](https://capec.mitre.org/) (CAPEC), The Mitre Corporation
- [Projet AppSensor OWASP](https://owasp.org/www-project-appsensor/)
- [Guide AppSensor v2](https://owasp.org/www-pdf-archive/Owasp-appsensor-guide-v2.pdf), OWASP
- Watson C, Coates M, Melton J et Groves G, [Créer des applications logicielles sensibles aux attaques avec des défenses en temps réel](https://pdfs.semanticscholar.org/0236/5631792fa6c953e82cadb0e7268be35df905.pdf), CrossTalk The Journal of Defense Software Ingénierie, Vol. 24, n° 5, septembre/octobre 2011
