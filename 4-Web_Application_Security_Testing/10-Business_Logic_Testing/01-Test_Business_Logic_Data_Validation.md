# Tester la validation des données de la logique métier

|ID          |
|------------|
|WSTG-BUSL-01|

## Sommaire

L'application doit s'assurer que seules des données logiquement valides peuvent être saisies à l'avant ainsi que directement côté serveur d'une application de système. Seule la vérification locale des données peut rendre les applications vulnérables aux injections de serveur via des proxys ou lors de transferts avec d'autres systèmes. Ceci est différent de la simple analyse des valeurs limites (BVA) en ce sens qu'il est plus difficile et dans la plupart des cas ne peut pas être simplement vérifié au point d'entrée, mais nécessite généralement la vérification d'un autre système.

Par exemple : une application peut vous demander votre numéro de sécurité sociale. Dans BVA, l'application doit vérifier les formats et la sémantique (la valeur est composée de 9 chiffres, non négative et pas uniquement de 0) pour les données saisies, mais il existe également des considérations logiques. Les SSN sont regroupés et catégorisés. Cette personne est-elle dans un dossier de décès ? Sont-ils originaires d'une certaine partie du pays?

Les vulnérabilités liées à la validation des données métier sont uniques en ce sens qu'elles sont spécifiques à l'application et différentes des vulnérabilités liées aux requêtes falsifiées en ce sens qu'elles sont plus préoccupées par les données logiques que par la simple rupture du flux de travail de la logique métier.

Le front-end et le back-end de l'application doivent vérifier et valider que les données dont il dispose, qu'il utilise et qu'il transmet sont logiquement valides. Même si l'utilisateur fournit des données valides à une application, la logique métier peut faire en sorte que l'application se comporte différemment selon les données ou les circonstances.

### Exemple 1

Supposons que vous gérez un site de commerce électronique à plusieurs niveaux qui permet aux utilisateurs de commander de la moquette. L'utilisateur sélectionne son tapis, entre la taille, effectue le paiement et l'application frontale a vérifié que toutes les informations saisies sont correctes et valides pour les informations de contact, la taille, la marque et la couleur du tapis. Mais, la logique métier en arrière-plan a deux chemins, si le tapis est en stock, il est directement expédié depuis votre entrepôt, mais s'il est en rupture de stock dans votre entrepôt, un appel est passé au système d'un partenaire et s'il l'a dans -stock ils expédieront la commande depuis leur entrepôt et remboursés par eux. Que se passe-t-il si un attaquant est en mesure de poursuivre une transaction en stock valide et de l'envoyer comme étant en rupture de stock à votre partenaire ? Que se passe-t-il si un attaquant est capable de s'interposer et d'envoyer des messages à l'entrepôt partenaire en commandant du tapis sans paiement ?

### Exemple 2

De nombreux systèmes de cartes de crédit téléchargent désormais les soldes des comptes la nuit afin que les clients puissent régler plus rapidement les montants inférieurs à une certaine valeur. L'inverse est également vrai. Si je rembourse ma carte de crédit le matin, je ne pourrai peut-être pas utiliser le crédit disponible le soir. Un autre exemple peut être que si j'utilise ma carte de crédit à plusieurs endroits très rapidement, il peut être possible de dépasser ma limite si les systèmes fondent leurs décisions sur les données de la nuit dernière.

### Exemple 3

**[Distributed Denial of Dollar (DDo$)](https://news.hitb.org/content/pirate-bay-proposes-distributed-denial-dollars-attack-ddo):**
Il s'agit d'une campagne proposée par le fondateur du site "The Pirate Bay" contre le cabinet d'avocats qui a engagé des poursuites contre "The Pirate Bay". L'objectif était de tirer parti des erreurs dans la conception des fonctionnalités commerciales et dans le processus de validation des virements.

Cette attaque a été réalisée en envoyant de très petites sommes d'argent de 1 SEK (0,13 USD) au cabinet d'avocats.
Le compte bancaire vers lequel les paiements ont été dirigés n'avait que 1 000 virements gratuits, après quoi tout virement entraîne un supplément pour le titulaire du compte (2 SEK). Après les mille premières transactions sur Internet, chaque don de 1 SEK au cabinet d'avocats finira par lui coûter 1 SEK à la place.

## Objectifs des tests

- Identifier les points d'injection de données.
- Validez que toutes les vérifications se produisent sur le back-end et ne peuvent pas être contournées.
- Essayez de casser le format des données attendues et analysez comment l'application les gère.

## Comment tester

### Méthode de test générique

- Examinez la documentation du projet et utilisez des tests exploratoires pour rechercher des points d'entrée de données ou des points de transfert entre les systèmes ou les logiciels.
- Une fois trouvé, essayez d'insérer des données logiquement invalides dans l'application/le système.
Méthode de test spécifique :
- Effectuer des tests de validation fonctionnelle de l'interface graphique frontale sur l'application pour s'assurer que seules les valeurs "valides" sont acceptées.
- À l'aide d'un proxy d'interception, observez le HTTP POST/GET à la recherche d'endroits où des variables telles que le coût et la qualité sont transmises. Plus précisément, recherchez les "transferts" entre les applications/systèmes qui peuvent être des points d'injection ou de sabotage possibles.
- Une fois les variables trouvées, commencez à interroger le champ avec des données logiquement "invalides", telles que des numéros de sécurité sociale ou des identifiants uniques qui n'existent pas ou qui ne correspondent pas à la logique métier. Ce test vérifie que le serveur fonctionne correctement et n'accepte pas de données logiquement invalides.

## Cas de test associés

- Tous les cas de test [Input Validation](../07-Input_Validation_Testing/README.md).
- [Test d'énumération de compte et de compte d'utilisateur devinable](../03-Identity_Management_Testing/04-Testing_for_Account_Enumeration_and_Guessable_User_Account.md).
- [Test pour contourner le schéma de gestion de session](../06-Session_Management_Testing/01-Testing_for_Session_Management_Schema.md).
- [Test des variables de session exposées](../06-Session_Management_Testing/04-Testing_for_Exposed_Session_Variables.md).

## Correction

L'application/le système doit s'assurer que seules les données "logiquement valides" sont acceptées à tous les points d'entrée et de transfert de l'application ou du système et que les données ne sont pas simplement fiables une fois qu'elles sont entrées dans le système.

## Outils

- [Proxy d'attaque Zed OWASP (ZAP)](https://www.zaproxy.org)
- [Burp Suite](https://portswigger.net/burp)

## Références

- [Contrôles proactifs OWASP (C5) - Valider toutes les entrées](https://owasp.org/www-project-proactive-controls/v3/en/c5-validate-inputs)
- [Série de feuilles de triche OWASP - Input_Validation_Cheat_Sheet](https://cheatsheetseries.owasp.org/cheatsheets/Input_Validation_Cheat_Sheet.html)
