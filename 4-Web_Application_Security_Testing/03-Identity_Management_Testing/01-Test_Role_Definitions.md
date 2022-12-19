# Définitions de rôle de test

|ID          |
|------------|
|WSTG-IDNT-01|

## Sommaire

Les applications ont plusieurs types de fonctionnalités et de services, et ceux-ci nécessitent des autorisations d'accès en fonction des besoins de l'utilisateur. Cet utilisateur pourrait être :

- un administrateur, où ils gèrent les fonctionnalités de l'application.
- un auditeur, où ils examinent les opérations d'application et fournissent un rapport détaillé.
- un ingénieur de support, où ils aident les clients à déboguer et à résoudre les problèmes sur leurs comptes.
- un client, où il interagit avec l'application et bénéficie de ses services.

Afin de gérer ces utilisations et tout autre cas d'utilisation pour cette application, des définitions de rôles sont configurées (plus communément appelées [RBAC](https://en.wikipedia.org/wiki/Role-based_access_control)). Sur la base de ces rôles, l'utilisateur est capable d'accomplir la tâche requise.

## Objectifs des tests

- Identifier et documenter les rôles utilisés par l'application.
- Tenter de changer, de changer ou d'accéder à un autre rôle.
- Revoir la granularité des rôles et des besoins derrière les permissions données.

## Comment tester

### Identification des rôles

Le testeur doit commencer par identifier les rôles d'application testés via l'une des méthodes suivantes :

- Dossier de candidature.
- Accompagnement par les développeurs ou administrateurs de l'application.
- Commentaires d'application.
- Rôles Fuzz possibles :
     - variable de cookie (*par exemple* `role=admin`, `isAdmin=True`)
     - variable de compte (*e.g.* `Role: manager`)
     - répertoires ou fichiers cachés (*par exemple* `/admin`, `/mod`, `/backups`)
     - passer à des utilisateurs bien connus (*e.g.* `admin`, `backups`, etc.)

### Passer aux rôles disponibles

Après avoir identifié les vecteurs d'attaque possibles, le testeur doit tester et valider qu'il peut accéder aux rôles disponibles.

> Certaines applications définissent les rôles de l'utilisateur à la création, par des contrôles et politiques rigoureux, ou en s'assurant que le rôle de l'utilisateur est correctement protégé par une signature créée par le backend. Découvrir que des rôles existent ne signifie pas qu'ils sont une vulnérabilité.

### Examiner les autorisations des rôles

Après avoir obtenu l'accès aux rôles sur le système, le testeur doit comprendre les autorisations accordées à chaque rôle.

Un ingénieur de support ne doit pas être en mesure de gérer les fonctionnalités administratives, de gérer les sauvegardes ou d'effectuer des transactions à la place d'un utilisateur.

Un administrateur ne devrait pas avoir les pleins pouvoirs sur le système. Les fonctionnalités d'administration sensibles doivent tirer parti d'un principe de fabricant-vérificateur ou utiliser MFA pour s'assurer que l'administrateur effectue la transaction. Un exemple clair à ce sujet est l'[incident Twitter en 2020](https://blog.twitter.com/en_us/topics/company/2020/an-update-on-our-security-incident.html).

## Outils

Les tests mentionnés ci-dessus peuvent être effectués sans l'utilisation d'aucun outil, à l'exception de celui utilisé pour accéder au système.

Pour rendre les choses plus faciles et plus documentées, on peut utiliser :

- [Extension d'autorisation de Burp](https://github.com/Quitten/Autorize)
- [Module complémentaire de test de contrôle d'accès de ZAP](https://www.zaproxy.org/docs/desktop/addons/access-control-testing/)

## Références

- [Ingénierie de rôle pour la gestion de la sécurité d'entreprise, E Coyne & J Davis, 2007](https://www.bookdepository.co.uk/Role-Engineering-for-Enterprise-Security-Management-Edward-Coyne/9781596932180)
- [Ingénierie des rôles et normes RBAC](https://csrc.nist.gov/projects/role-based-access-control#rbac-standard)
