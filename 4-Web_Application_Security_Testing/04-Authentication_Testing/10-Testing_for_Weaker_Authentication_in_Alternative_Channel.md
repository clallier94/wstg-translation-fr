# Test d'authentification plus faible dans un canal alternatif

|ID          |
|------------|
|WSTG-ATHN-10|

## Sommaire

Même si les mécanismes d'authentification primaires n'incluent aucune vulnérabilité, il se peut que des vulnérabilités existent dans d'autres canaux d'utilisateurs légitimes d'authentification pour les mêmes comptes d'utilisateurs. Des tests doivent être entrepris pour identifier les canaux alternatifs et, sous réserve de la portée des tests, identifier les vulnérabilités.

Les canaux d'interaction utilisateur alternatifs pourraient être utilisés pour contourner le canal principal ou exposer des informations qui peuvent ensuite être utilisées pour assister une attaque contre le canal principal. Certains de ces canaux peuvent eux-mêmes être des applications Web distinctes utilisant des noms d'hôte ou des chemins différents. Par exemple:

- Site Web standard
- Mobile, ou appareil spécifique, site Web optimisé
- Site Web optimisé pour l'accessibilité
- Sites Web alternatifs de pays et de langue
- Sites Web parallèles qui utilisent les mêmes comptes d'utilisateurs (par exemple, un autre site Web offrant des fonctionnalités différentes de la même organisation, un site Web partenaire avec lequel les comptes d'utilisateurs sont partagés)
- Développement, test, UAT et versions staging du site standard

Mais il peut également s'agir d'autres types d'applications ou de processus métier :

- Application pour appareil mobile
- Application de bureau
- Opérateurs de centres d'appels
- Systèmes interactifs de réponse vocale ou d'arborescence téléphonique

Notez que ce test se concentre sur les canaux alternatifs ; certaines alternatives d'authentification peuvent apparaître comme des contenus différents diffusés via le même site Web et seraient presque certainement susceptibles d'être testées. Celles-ci ne sont pas décrites plus en détail ici et auraient dû être identifiées lors de la collecte d'informations et des tests d'authentification primaire. Par exemple:

- Enrichissement progressif et dégradation gracieuse qui modifient les fonctionnalités
- Utilisation du site sans cookies
- Utilisation du site sans JavaScript
- Utilisation du site sans plugins comme pour Flash et Java

Même si le périmètre du test ne permet pas de tester les canaux alternatifs, leur existence doit être documentée. Ceux-ci peuvent miner le degré d'assurance dans les mécanismes d'authentification et peuvent être un précurseur de tests supplémentaires.

## Exemple

Le site Web principal est `http://www.exemple.com` et les fonctions d'authentification ont toujours lieu sur les pages utilisant TLS `https://www.exemple.com/myaccount/`.

Cependant, il existe un site Web distinct optimisé pour les mobiles qui n'utilise pas du tout TLS et qui dispose d'un mécanisme de récupération de mot de passe plus faible `http://m.exemple.com/myaccount/`.

## Objectifs des tests

- Identifier les canaux d'authentification alternatifs.
- Évaluer les mesures de sécurité utilisées et s'il existe des contournements sur les canaux alternatifs.

## Comment tester

### Comprendre le mécanisme principal

Testez entièrement les principales fonctions d'authentification du site Web. Cela devrait identifier comment les comptes sont émis, créés ou modifiés et comment les mots de passe sont récupérés, réinitialisés ou modifiés. De plus, la connaissance de toute authentification à privilèges élevés et des mesures de protection de l'authentification doit être connue. Ces précurseurs sont nécessaires pour pouvoir comparer avec d'éventuelles voies alternatives.

### Identifier d'autres chaînes

D'autres canaux peuvent être trouvés en utilisant les méthodes suivantes :

- Lire le contenu du site, en particulier la page d'accueil, nous contacter, les pages d'aide, les articles de support et les FAQ, les CGU, les avis de confidentialité, le fichier robots.txt et tous les fichiers sitemap.xml.
- Recherche dans les journaux de proxy HTTP, enregistrés lors de la collecte et des tests d'informations précédents, pour des chaînes telles que "mobile", "android", blackberry", "ipad", "iphone", "application mobile", "e-reader", "sans fil ", "auth", "sso", "single sign on" dans les chemins d'URL et le contenu du corps.
- Utilisez les moteurs de recherche pour trouver différents sites Web de la même organisation, ou utilisant le même nom de domaine, qui ont un contenu de page d'accueil similaire ou qui ont également des mécanismes d'authentification.

Pour chaque canal possible, confirmez si les comptes d'utilisateurs sont partagés entre ceux-ci ou permettent d'accéder à la même fonctionnalité ou à une fonctionnalité similaire.

### Enumérer la fonctionnalité d'authentification

Pour chaque canal alternatif où les comptes d'utilisateurs ou les fonctionnalités sont partagés, identifiez si toutes les fonctions d'authentification du canal principal sont disponibles et s'il existe des éléments supplémentaires. Il peut être utile de créer une grille comme celle ci-dessous :

  |             Primaire             | Mobile | Centre d'appels | Site Web partenaire |
  |----------------------------------|--------|-----------------|---------------------|
  | Inscrivez-vous                   |  Oui   |        -        |          -          |
  | Se connecter                     |  Oui   |       Oui       |       Oui(SSO)      |
  | Déconnectez-vous                 |   -    |        -        |          -          |
  | Réinitialisation du mot de passe |  Oui   |       Oui       |          -          |
  | Changer le mot de passe          |        |        -        |          -          |

Dans cet exemple, le mobile a une fonction supplémentaire "changer le mot de passe" mais n'offre pas la "déconnexion". Un nombre limité de tâches est également possible en téléphonant au centre d'appels. Les centres d'appels peuvent être intéressants, car leurs contrôles de confirmation d'identité peuvent être plus faibles que ceux du site Web, ce qui permet d'utiliser ce canal pour faciliter une attaque contre le compte d'un utilisateur.

Tout en les énumérant, il convient de prendre note de la façon dont la gestion des sessions est entreprise, en cas de chevauchement entre les canaux (par exemple, les cookies étendus au même nom de domaine parent, les sessions simultanées autorisées sur les canaux, mais pas sur le même canal).

### Examen et test

Les canaux alternatifs doivent être mentionnés dans le rapport de test, même s'ils sont marqués comme "informations uniquement" ou "hors champ". Dans certains cas, la portée du test peut inclure le canal alternatif (par exemple, parce qu'il s'agit simplement d'un autre chemin sur le nom d'hôte cible), ou peut être ajoutée à la portée après discussion avec les propriétaires de tous les canaux. Si les tests sont autorisés et autorisés, tous les autres tests d'authentification de ce guide doivent alors être effectués et comparés au canal principal.

## Cas de test associés

Les cas de test pour tous les autres tests d'authentification doivent être utilisés.

## Correction

Assurez-vous qu'une politique d'authentification cohérente est appliquée sur tous les canaux afin qu'ils soient également sécurisés.
