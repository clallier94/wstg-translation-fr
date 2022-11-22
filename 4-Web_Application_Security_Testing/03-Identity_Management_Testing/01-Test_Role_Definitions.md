# D�finitions de r�le de test

|ID          |
|------------|
|WSTG-IDNT-01|

## Sommaire

Les applications ont plusieurs types de fonctionnalit�s et de services, et ceux-ci n�cessitent des autorisations d'acc�s en fonction des besoins de l'utilisateur. Cet utilisateur pourrait �tre�:

- un administrateur, o� ils g�rent les fonctionnalit�s de l'application.
- un auditeur, o� ils examinent les op�rations d'application et fournissent un rapport d�taill�.
- un ing�nieur de support, o� ils aident les clients � d�boguer et � r�soudre les probl�mes sur leurs comptes.
- un client, o� il interagit avec l'application et b�n�ficie de ses services.

Afin de g�rer ces utilisations et tout autre cas d'utilisation pour cette application, des d�finitions de r�les sont configur�es (plus commun�ment appel�es [RBAC](https://en.wikipedia.org/wiki/Role-based_access_control)). Sur la base de ces r�les, l'utilisateur est capable d'accomplir la t�che requise.

## Objectifs des tests

- Identifier et documenter les r�les utilis�s par l'application.
- Tenter de changer, de changer ou d'acc�der � un autre r�le.
- Revoir la granularit� des r�les et des besoins derri�re les permissions donn�es.

## Comment tester

### Identification des r�les

Le testeur doit commencer par identifier les r�les d'application test�s via l'une des m�thodes suivantes�:

- Dossier de candidature.
- Accompagnement par les d�veloppeurs ou administrateurs de l'application.
- Commentaires d'application.
- Fuzz r�les possibles :
     - variable de cookie (*par exemple* `role=admin`, `isAdmin=True`)
     - variable de compte (*e.g.* `Role: manager`)
     - r�pertoires ou fichiers cach�s (*par exemple* `/admin`, `/mod`, `/backups`)
     - passer � des utilisateurs bien connus (*e.g.* `admin`, `backups`, etc.)

### Passer aux r�les disponibles

Apr�s avoir identifi� les vecteurs d'attaque possibles, le testeur doit tester et valider qu'il peut acc�der aux r�les disponibles.

> Certaines applications d�finissent les r�les de l'utilisateur � la cr�ation, par des contr�les et politiques rigoureux, ou en s'assurant que le r�le de l'utilisateur est correctement prot�g� par une signature cr��e par le backend. D�couvrir que des r�les existent ne signifie pas qu'ils sont une vuln�rabilit�.

### Examiner les autorisations des r�les

Apr�s avoir obtenu l'acc�s aux r�les sur le syst�me, le testeur doit comprendre les autorisations accord�es � chaque r�le.

Un ing�nieur de support ne doit pas �tre en mesure de g�rer les fonctionnalit�s administratives, de g�rer les sauvegardes ou d'effectuer des transactions � la place d'un utilisateur.

Un administrateur ne devrait pas avoir les pleins pouvoirs sur le syst�me. Les fonctionnalit�s d'administration sensibles doivent tirer parti d'un principe de fabricant-v�rificateur ou utiliser MFA pour s'assurer que l'administrateur effectue la transaction. Un exemple clair � ce sujet est l'[incident Twitter en 2020](https://blog.twitter.com/en_us/topics/company/2020/an-update-on-our-security-incident.html).

## Outils

Les tests mentionn�s ci-dessus peuvent �tre effectu�s sans l'utilisation d'aucun outil, � l'exception de celui utilis� pour acc�der au syst�me.

Pour rendre les choses plus faciles et plus document�es, on peut utiliser :

- [Extension d'autorisation de Burp] (https://github.com/Quitten/Autorize)
- [Module compl�mentaire de test de contr�le d'acc�s de ZAP] (https://www.zaproxy.org/docs/desktop/addons/access-control-testing/)

## R�f�rences

- [Ing�nierie de r�le pour la gestion de la s�curit� d'entreprise, E Coyne & J Davis, 2007](https://www.bookdepository.co.uk/Role-Engineering-for-Enterprise-Security-Management-Edward-Coyne/9781596932180)
- [Ing�nierie des r�les et normes RBAC](https://csrc.nist.gov/projects/role-based-access-control#rbac-standard)
