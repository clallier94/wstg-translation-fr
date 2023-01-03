# Introduction à la logique métier

Tester les failles de la logique métier dans une application Web dynamique multifonctionnelle nécessite de penser à des méthodes non conventionnelles. Si le mécanisme d'authentification d'une application est développé avec l'intention d'effectuer les étapes 1, 2, 3 dans cet ordre spécifique pour authentifier un utilisateur. Que se passe-t-il si l'utilisateur passe directement de l'étape 1 à l'étape 3 ? Dans cet exemple simpliste, l'application fournit-elle un accès par échec d'ouverture ? refuser l'accès, ou juste une erreur avec un message 500 ?

Il existe de nombreux exemples qui peuvent être donnés, mais la seule leçon constante est de "penser en dehors de la sagesse conventionnelle". Ce type de vulnérabilité ne peut pas être détecté par un scanner de vulnérabilité et repose sur les compétences et la créativité du testeur d'intrusion. De plus, ce type de vulnérabilité est généralement l'un des plus difficiles à détecter, et généralement spécifique à l'application, mais, en même temps, généralement l'un des plus préjudiciables à l'application, s'il est exploité.

La classification des failles de la logique métier a été sous-étudiée ; bien que l'exploitation des failles commerciales se produise fréquemment dans les systèmes du monde réel, et de nombreux chercheurs en vulnérabilité appliquée les étudient. L'accent est mis sur les applications Web. Il y a un débat au sein de la communauté pour savoir si ces problèmes représentent des concepts particulièrement nouveaux ou s'il s'agit de variations de principes bien connus.

Le test des failles de la logique métier est similaire aux types de test utilisés par les testeurs fonctionnels qui se concentrent sur les tests logiques ou à états finis. Ces types de tests exigent que les professionnels de la sécurité pensent un peu différemment, développent des cas d'abus et d'utilisation abusive et utilisent de nombreuses techniques de test adoptées par les testeurs fonctionnels. L'automatisation des cas d'abus de logique métier n'est pas possible et reste un art manuel reposant sur les compétences du testeur et sa connaissance du processus métier complet et de ses règles.

## Limites et restrictions commerciales

Tenez compte des règles de la fonction métier fournie par l'application. Y a-t-il des limites ou des restrictions sur le comportement des gens ? Déterminez ensuite si l'application applique ces règles. Il est généralement assez facile d'identifier les cas de test et d'analyse pour vérifier l'application si vous êtes familier avec l'entreprise. Si vous êtes un testeur tiers, vous allez devoir faire preuve de bon sens et demander à l'entreprise si différentes opérations doivent être autorisées par l'application.

Parfois, dans des applications très complexes, le testeur n'aura pas une compréhension complète de tous les aspects de l'application au départ. Dans ces situations, il est préférable que le client guide le testeur dans l'application, afin qu'il puisse mieux comprendre les limites et les fonctionnalités prévues de l'application, avant le début du test proprement dit. De plus, avoir une ligne directe avec les développeurs (si possible) pendant les tests aidera grandement, si des questions se posent concernant la fonctionnalité de l'application.

## Défis des tests logiques

Les outils automatisés ont du mal à comprendre le contexte, c'est donc à une personne d'effectuer ce genre de tests. Les deux exemples suivants illustrent comment la compréhension de la fonctionnalité de l'application, les intentions du développeur et certaines réflexions créatives "prêtes à l'emploi" peuvent briser la logique de l'application. Le premier exemple commence par une manipulation simpliste des paramètres, tandis que le second est un exemple réel d'un processus en plusieurs étapes conduisant à subvertir complètement l'application.

**Exemple 1** :

Supposons qu'un site de commerce électronique permette aux utilisateurs de sélectionner des articles à acheter, d'afficher une page de résumé, puis de proposer la vente. Que se passerait-il si un attaquant pouvait revenir à la page de résumé, maintenir sa même session valide et injecter un coût inférieur pour un article et terminer la transaction, puis passer à la caisse ?

**Exemple 2** :

Conserver/verrouiller des ressources et empêcher les autres d'acheter ces articles en ligne peut amener les attaquants à acheter des articles à un prix inférieur. La contre-mesure à ce problème consiste à implémenter des délais d'attente et des mécanismes pour s'assurer que seul le prix correct peut être facturé.

**Exemple 3** :

Que se passe-t-il si un utilisateur a pu démarrer une transaction liée à son compte club/fidélité, puis après que des points ont été ajoutés à son compte, annuler la transaction ? Les points/crédits seront-ils toujours appliqués à leur compte ?

## Outils

Bien qu'il existe des outils pour tester et vérifier que les processus métier fonctionnent correctement dans des situations valides, ces outils sont incapables de détecter les vulnérabilités logiques. Par exemple, les outils n'ont aucun moyen de détecter si un utilisateur est capable de contourner le flux des processus métier en éditant des paramètres, en prédisant des noms de ressources ou en augmentant les privilèges pour accéder à des ressources restreintes, et ils n'ont aucun mécanisme pour aider les testeurs humains à suspecter cet état de affaires.

Voici quelques types d'outils courants qui peuvent être utiles pour identifier les problèmes de logique métier.

Lors de l'installation d'addons, vous devez toujours faire preuve de diligence en tenant compte des autorisations qu'ils demandent et des habitudes d'utilisation de votre navigateur.

### Proxy d'interception

Pour observer les blocs de requête et de réponse du trafic HTTP

- [Proxy d'attaque Zed OWASP](https://www.zaproxy.org)
- [Burp Proxy](https://portswigger.net/burp)

### Plug-ins de navigateur Web

Pour afficher et modifier les en-têtes HTTP/HTTPS, publier les paramètres et observer le DOM du navigateur

- [Tamper Data for FF Quantum](https://addons.mozilla.org/en-US/firefox/addon/tamper-data-for-ff-quantum)
- [Tamper Chrome (pour Google Chrome)](https://chrome.google.com/webstore/detail/tamper-chrome-extension/hifhgpdkfodlpnlmlnmhchnkepplebkb)

## Outils de test divers

- [Barre d'outils du développeur Web](https://chrome.google.com/webstore/detail/bfbameneiokkgbdmiekhjnmfkcnldhhm)
    - L'extension Web Developer ajoute un bouton de barre d'outils au navigateur avec divers outils de développement Web. Il s'agit du portage officiel de l'extension Web Developer pour Firefox.
- [Fabricant de requêtes HTTP pour Chrome](https://chrome.google.com/webstore/detail/kajfghlhfkcocafkcjlajldicbikpgnp)
- [Fabricant de requêtes HTTP pour Firefox](https://addons.mozilla.org/en-US/firefox/addon/http-request-maker)
    - Request Maker est un outil de test d'intrusion. Avec lui, vous pouvez facilement capturer les requêtes faites par les pages Web, falsifier l'URL, les en-têtes et les données POST et, bien sûr, faire de nouvelles requêtes
- [Éditeur de cookies pour Chrome](https://chrome.google.com/webstore/detail/fngmhnnpilhplaeedifhccceomclgfbg)
- [Éditeur de cookies pour Firefox](https://addons.mozilla.org/en-US/firefox/addon/cookie-editor)
    - Cookie Editor est un gestionnaire de cookies. Vous pouvez ajouter, supprimer, modifier, rechercher, protéger et bloquer les cookies

## Références

### Papiers blanc

- [The Common Misuse Scoring System (CMSS): Metrics for Software Feature Misuse Vulnerabilities - NISTIR 7864](https://csrc.nist.gov/publications/detail/nistir/7864/final)
- [Test à l'état fini des interfaces utilisateur graphiques, Fevzi Belli](https://pdfs.semanticscholar.org/b57c/6c8022abfd2cb17ec785d3622027b3edfaaf.pdf)
- [Principes et méthodes de test des machines à états finis - Une enquête, David Lee, Mihalis Yannakakis](http://citeseerx.ist.psu.edu/viewdoc/download?doi=10.1.1.380.3405&rep=rep1&type=pdf)
- [Problèmes de sécurité dans les jeux en ligne, Jianxin Jeff Yan et Hyun-Jin Choi](https://www.researchgate.net/publication/220677013_Security_issues_in_online_games)
- [Securing Virtual Worlds Against Real Attack, Dr. Igor Muttik, McAfee](https://www.info-point-security.com/open_downloads/2008/McAfee_wp_online_gaming_0808.pdf)
- [Seven Business Logic Flaws That Put Your Website At Risk - Jeremiah Grossman Fondateur et CTO, WhiteHat Security](https://www.slideshare.net/jeremiahgrossman/seven-business-logic-flaws-that-put-your-website-at-risk-harvard-07062008)
- [Vers une détection automatisée des vulnérabilités logiques dans les applications Web - Viktoria Felmetsger Ludovico Cavedon Christopher Kruegel Giovanni Vigna](https://www.usenix.org/legacy/event/sec10/tech/full_papers/Felmetsger.pdf)

### Lié à l'OWASP

- [Comment prévenir les failles commerciales dans les applications Web, Marco Morana](http://www.slideshare.net/marco_morana/issa-louisville-2010morana)

### Sites Web utiles

- [Abus de fonctionnalité](http://projects.webappsec.org/w/page/13246913/Abuse-of-Functionality)
- [Logique métier](https://en.wikipedia.org/wiki/Business_logic)
- [Business Logic Flaws et Yahoo Games](http://jeremiahgrossman.blogspot.com/2006/12/business-logic-flaws.html)
- [CWE-840 : Erreurs de logique métier](https://cwe.mitre.org/data/definitions/840.html)
- [Défier la logique : théorie, conception et mise en œuvre de systèmes complexes pour tester la logique d'application](https://pdfs.semanticscholar.org/d14a/18f08f6488f903f2f691a1d159e95d8ee04f.pdf)
- [Cycle de vie des tests logiciels](http://softwaretestingfundamentals.com/software-testing-life-cycle/)

### Livres

- [Le ​​modèle de décision : un cadre logique métier reliant les entreprises et la technologie, par Barbara Von Halle, Larry Goldberg, publié par CRC Press, ISBN1420082817 (2010)](https://isbnsearch.org/isbn/1420082817)
