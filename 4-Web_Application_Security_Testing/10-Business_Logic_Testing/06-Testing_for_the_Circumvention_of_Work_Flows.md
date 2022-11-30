# Tests pour le contournement des flux de travail

|ID          |
|------------|
|WSTG-BUSL-06|

## Sommaire

Les vulnérabilités de flux de travail impliquent tout type de vulnérabilité qui permet à l'attaquant d'utiliser à mauvais escient une application/un système d'une manière qui lui permettra de contourner (et non de suivre) le flux de travail conçu/prévu.

[Définition d'un workflow sur Wikipedia](https://en.wikipedia.org/wiki/Workflow) :

> Un flux de travail se compose d'une séquence d'étapes connectées où chaque étape suit sans délai ni intervalle et se termine juste avant que l'étape suivante puisse commencer. C'est la représentation d'une séquence d'opérations, déclarées comme travail d'une personne ou d'un groupe, d'une organisation de personnel, ou d'un ou plusieurs mécanismes simples ou complexes. Le flux de travail peut être considéré comme toute abstraction du travail réel.

La logique métier de l'application doit exiger que l'utilisateur effectue des étapes spécifiques dans l'ordre correct/spécifique et si le flux de travail se termine sans se terminer correctement, toutes les actions et les actions générées sont "annulées" ou annulées. Les vulnérabilités liées au contournement des flux de travail ou au contournement du flux de travail de la logique métier appropriée sont uniques en ce sens qu'elles sont très spécifiques à l'application/au système et que des cas d'utilisation manuelle abusive doivent être développés à l'aide d'exigences et de cas d'utilisation.

Le processus métier des applications doit avoir des vérifications pour s'assurer que les transactions/actions de l'utilisateur se déroulent dans l'ordre correct/acceptable et si une transaction déclenche une sorte d'action, cette action sera "annulée" et supprimée si la transaction n'est pas terminée avec succès .

### Exemple 1

Beaucoup d'entre nous reçoivent ce type de "points de club/de fidélité" pour les achats dans les épiceries et les stations-service. Supposons qu'un utilisateur puisse démarrer une transaction liée à son compte, puis après que des points ont été ajoutés à son compte club/fidélité, annuler la transaction ou supprimer des articles de son « panier » et de son appel d'offres. Dans ce cas, soit le système ne doit pas appliquer de points/crédits au compte jusqu'à ce qu'il soit offert, soit les points/crédits doivent être "annulés" si l'incrément de points/crédits ne correspond pas à l'offre finale. Dans cet esprit, un attaquant peut démarrer des transactions et les annuler pour augmenter leurs niveaux de points sans rien acheter.

### Exemple 2

Un système de tableau d'affichage électronique peut être conçu pour garantir que les messages initiaux ne contiennent pas de grossièretés sur la base d'une liste à laquelle le message est comparé. Si un mot sur une liste de refus est trouvé dans le texte saisi par l'utilisateur, la soumission n'est pas publiée. Mais, une fois qu'une soumission est publiée, l'auteur peut accéder, modifier et modifier le contenu de la soumission pour inclure des mots inclus dans la liste des blasphèmes/refus puisque lors de la modification, la publication n'est plus jamais comparée. En gardant cela à l'esprit, les attaquants peuvent ouvrir une discussion initiale vide ou minimale, puis ajouter ce qu'ils veulent comme mise à jour.

## Objectifs des tests

- Consultez la documentation du projet pour connaître les méthodes permettant d'ignorer ou de parcourir les étapes du processus de candidature dans un ordre différent du flux de logique métier prévu.
- Développer un cas d'utilisation abusive et essayer de contourner chaque flux logique identifié.

## Comment tester

### Méthode de test 1

- Démarrez une transaction en passant par l'application au-delà des points qui déclenchent des crédits/points sur le compte de l'utilisateur.
- Annuler la transaction ou réduire l'offre finale afin que les valeurs des points soient réduites et vérifier le système de points/crédits pour s'assurer que les bons points/crédits ont été enregistrés.

### Méthode de test 2

- Sur un système de gestion de contenu ou de tableau d'affichage, saisissez et enregistrez un texte ou des valeurs initiales valides.
- Essayez ensuite d'ajouter, de modifier et de supprimer des données qui laisseraient les données existantes dans un état invalide ou avec des valeurs invalides pour vous assurer que l'utilisateur n'est pas autorisé à enregistrer les informations incorrectes. Certaines données ou informations "invalides" peuvent être des mots spécifiques (blasphèmes) ou des sujets spécifiques (tels que des questions politiques).

## Cas de test associés

- [Test de la traversée du répertoire/de l'inclusion de fichiers](../05-Authorization_Testing/01-Testing_Directory_Traversal_File_Include.md)
- [Test de contournement du schéma d'autorisation](../05-Authorization_Testing/02-Testing_for_Bypassing_Authorization_Schema.md)
- [Test pour contourner le schéma de gestion de session](../06-Session_Management_Testing/01-Testing_for_Session_Management_Schema.md)
- [Tester la validation des données de la logique métier] (01-Test_Business_Logic_Data_Validation.md)
- [Tester la capacité de forger des demandes] (02-Test_Ability_to_Forge_Requests.md)
- [Tester les vérifications d'intégrité] (03-Test_Integrity_Checks.md)
- [Tester la synchronisation du processus] (04-Test_for_Process_Timing.md)
- [Tester le nombre de fois qu'une fonction peut être utilisée dans les limites] (05-Test_Number_of_Times_a_Function_Can_Be_Used_Limits.md)
- [Tester les défenses contre l'utilisation abusive des applications] (07-Test_Defenses_Against_Application_Misuse.md)
- [Téléchargement test de types de fichiers inattendus] (08-Test_Upload_of_Unexpected_File_Types.md)
- [Tester le téléchargement de fichiers malveillants] (09-Test_Upload_of_Malicious_Files.md)

## Correction

L'application doit être consciente d'elle-même et disposer de contrôles garantissant que les utilisateurs effectuent chaque étape du processus de flux de travail dans le bon ordre et empêchent les attaquants de contourner/sauter/ou répéter les étapes/processus du flux de travail. Le test des vulnérabilités du flux de travail implique de développer des cas d'abus/de mauvaise utilisation de la logique métier dans le but de mener à bien le processus métier sans suivre les étapes correctes dans le bon ordre.

## Références

- [Aide-mémoire sur les cas d'abus OWASP] (https://cheatsheetseries.owasp.org/cheatsheets/Abuse_Case_Cheat_Sheet.html)
- [CWE-840 : Erreurs de logique métier](https://cwe.mitre.org/data/definitions/840.html)
