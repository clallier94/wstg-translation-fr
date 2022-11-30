# Tester les d�fenses contre l'utilisation abusive des applications

|ID          |
|------------|
|WSTG-BUSL-07|

## Sommaire

L'utilisation abusive et invalide de fonctionnalit�s valides peut identifier les attaques tentant d'�num�rer l'application Web, d'identifier les faiblesses et d'exploiter les vuln�rabilit�s. Des tests doivent �tre entrepris pour d�terminer s'il existe des m�canismes de d�fense de la couche application en place pour prot�ger l'application.

L'absence de d�fenses actives permet � un attaquant de chasser les vuln�rabilit�s sans aucun recours. Le propri�taire de l'application ne saura donc pas que son application est attaqu�e.

### Exemple

Un utilisateur authentifi� entreprend la s�quence d'actions (improbable) suivante�:

1. Tentative d'acc�s � un ID de fichier que leurs r�les ne sont pas autoris�s � t�l�charger
2. Remplace une seule coche `'` au lieu du num�ro d'identification du fichier
3. Modifie une requ�te GET en POST
4. Ajoute un param�tre suppl�mentaire
5. Duplique une paire nom/valeur de param�tre

L'application surveille les abus et r�pond apr�s le 5�me �v�nement avec une confiance extr�mement �lev�e que l'utilisateur est un attaquant. Par exemple l'application :

- D�sactive les fonctionnalit�s critiques
- Active des �tapes d'authentification suppl�mentaires pour les fonctionnalit�s restantes
- Ajoute des d�lais dans chaque cycle de demande-r�ponse
- Commence � enregistrer des donn�es suppl�mentaires sur les interactions de l'utilisateur (par exemple, les en-t�tes de requ�te HTTP nettoy�s, les corps et les corps de r�ponse)

Si l'application ne r�pond pas de quelque mani�re que ce soit et que l'attaquant peut continuer � abuser des fonctionnalit�s et soumettre un contenu clairement malveillant � l'application, l'application a �chou� � ce cas de test. En pratique, il est peu probable que les exemples d'actions discr�tes de l'exemple ci-dessus se produisent ainsi. Il est beaucoup plus probable qu'un outil de fuzzing soit utilis� pour identifier tour � tour les faiblesses de chaque param�tre. C'est ce qu'un testeur de s�curit� aura �galement entrepris.

## Objectifs des tests

- G�n�rer des notes de tous les tests effectu�s contre le syst�me.
- Passez en revue les tests qui avaient une fonctionnalit� diff�rente bas�e sur une entr�e agressive.
- Comprendre les d�fenses en place et v�rifier si elles suffisent � prot�ger le syst�me contre les techniques de contournement.

## Comment tester

Ce test est inhabituel dans la mesure o� le r�sultat peut �tre tir� de tous les autres tests effectu�s sur l'application Web. Lors de l'ex�cution de tous les autres tests, notez les mesures qui pourraient indiquer que l'application dispose d'une autod�fense int�gr�e�:

- R�ponses modifi�es
- Requ�tes bloqu�es
- Actions qui d�connectent un utilisateur ou verrouillent son compte

Celles-ci ne peuvent �tre que localis�es. Les d�fenses localis�es courantes (par fonction) sont�:

- Rejeter l'entr�e contenant certains caract�res
- Verrouillage temporaire d'un compte apr�s un certain nombre d'�checs d'authentification

Les contr�les de s�curit� localis�s ne suffisent pas. Il n'y a souvent aucune d�fense contre une mauvaise utilisation g�n�rale telle que�:

- Navigation forc�e
- Contournement de la validation des entr�es de la couche de pr�sentation
- Plusieurs erreurs de contr�le d'acc�s
- Noms de param�tres suppl�mentaires, dupliqu�s ou manquants
- Plusieurs �checs de validation des entr�es ou de v�rification de la logique m�tier avec des valeurs qui ne peuvent pas �tre le r�sultat d'erreurs ou de fautes de frappe de l'utilisateur
- Des donn�es structur�es (par exemple JSON, XML) d'un format invalide sont re�ues
- R�ception de scripts intersites flagrants ou de charges utiles d'injection SQL
- Utiliser l'application plus rapidement qu'il ne serait possible sans outils d'automatisation
- Changement de g�olocalisation continentale d'un utilisateur
- Changement d'agent utilisateur
- Acc�der � un processus m�tier en plusieurs �tapes dans le mauvais ordre
- Un grand nombre ou un taux �lev� d'utilisation de fonctionnalit�s sp�cifiques � l'application (par exemple, soumission de code de bon d'achat, �chec des paiements par carte de cr�dit, t�l�chargements de fichiers, t�l�chargements de fichiers, d�connexions, etc.).

Ces d�fenses fonctionnent mieux dans les parties authentifi�es de l'application, bien que le taux de cr�ation de nouveaux comptes ou d'acc�s au contenu (par exemple pour r�cup�rer des informations) puisse �tre utile dans les zones publiques.

Tous les �l�ments ci-dessus n'ont pas besoin d'�tre surveill�s par l'application, mais il y a un probl�me si aucun d'entre eux ne l'est. En testant l'application Web, en effectuant le type d'actions ci-dessus, une r�ponse a-t-elle �t� prise contre le testeur�? Si ce n'est pas le cas, le testeur doit signaler que l'application semble n'avoir aucune d�fense active � l'�chelle de l'application contre les abus. Notez qu'il est parfois possible que toutes les r�ponses � la d�tection d'attaques soient silencieuses pour l'utilisateur (par exemple, modifications de journalisation, surveillance accrue, alertes aux administrateurs et demande de proxy), donc la confiance dans cette d�couverte ne peut �tre garantie. En pratique, tr�s peu d'applications (ou d'infrastructures connexes telles qu'un pare-feu d'application Web) d�tectent ces types d'utilisation abusive.

## Cas de test associ�s

Tous les autres cas de test sont pertinents.

## Correction

Les applications doivent impl�menter des d�fenses actives pour repousser les attaquants et les abuseurs.

## R�f�rences

- [Software Assurance](https://www.cisa.gov/uscert/sites/default/files/publications/infosheet_SoftwareAssurance.pdf), US Department Homeland Security
- [IR 7684](https://csrc.nist.gov/publications/detail/nistir/7864/final) Syst�me commun de notation des abus (CMSS), NIST
- [Common Attack Pattern Enumeration and Classification](https://capec.mitre.org/) (CAPEC), The Mitre Corporation
- [Projet AppSensor OWASP](https://owasp.org/www-project-appsensor/)
- [Guide AppSensor v2](https://owasp.org/www-pdf-archive/Owasp-appsensor-guide-v2.pdf), OWASP
- Watson C, Coates M, Melton J et Groves G, [Cr�er des applications logicielles sensibles aux attaques avec des d�fenses en temps r�el](https://pdfs.semanticscholar.org/0236/5631792fa6c953e82cadb0e7268be35df905.pdf), CrossTalk The Journal of Defense Software Ing�nierie, Vol. 24, n� 5, septembre/octobre 2011
