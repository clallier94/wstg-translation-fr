# Rapports

La r�alisation de l'aspect technique de l'�valuation ne repr�sente que la moiti� du processus d'�valuation global. Le produit final est la production d'un rapport bien �crit et informatif. Un rapport doit �tre facile � comprendre et doit mettre en �vidence tous les risques constat�s lors de la phase d'�valuation. Le rapport devrait plaire � la fois � la direction g�n�rale et au personnel technique.

## � propos de cette section

Ce guide ne fournit que des suggestions sur une approche possible en mati�re de signalement et ne doit pas �tre consid�r� comme des r�gles strictes � suivre. Lorsque vous envisagez l'une des recommandations ci-dessous, demandez-vous toujours si la recommandation am�liorerait votre rapport.

Ce guide de reporting est le mieux adapt� aux rapports bas�s sur des conseils. Cela peut �tre exag�r� pour les rapports internes ou de prime de bogue.

Quelle que soit l'audience, il est conseill� de s�curiser le rapport et de le crypter pour s'assurer que seul le destinataire est en mesure de l'utiliser.

Un bon rapport aide votre client � comprendre vos conclusions et met en �vidence la qualit� de vos tests techniques. La qualit� des tests techniques est compl�tement hors de propos si le client ne peut pas comprendre vos conclusions.

## 1. Introduction

### 1.1 Contr�le des versions

D�finit les modifications du rapport, principalement pr�sent�es sous forme de tableau comme ci-dessous.

| version | Descriptif | Rendez-vous | Auteur |
|:-------�:|-------------|------|--------|
| 1.0 | Rapport initial | JJ/MM/AAAA | J. Doe |

### 1.2 Table des mati�res

Une page de table des mati�res pour le document.

### 1.3 L'�quipe

Une liste des membres de l'�quipe d�taillant leur expertise et leurs qualifications.

### 1.4 Champ d'application

Les limites et les besoins de l'engagement convenus avec l'organisation.

### 1.5 Limites

Les limites peuvent �tre�:

- Zones interdites par rapport aux tests.
- Fonctionnalit� cass�e.
- Manque de coop�ration.
- Manque de temps.
- Manque d'acc�s ou d'informations d'identification.

### 1.6 Chronologie

La dur�e de l'engagement.

### 1.7 Clause de non-responsabilit�

Vous voudrez peut-�tre fournir une clause de non-responsabilit� pour votre service. Consultez toujours un professionnel du droit afin de cr�er un document juridiquement contraignant.

L'exemple suivant est fourni � titre indicatif uniquement. Il ne doit pas �tre utilis� tel quel et ne constitue pas un avis juridique.

*Ce test est une �valuation "� un moment donn�" et, en tant que tel, l'environnement peut avoir chang� depuis que le test a �t� ex�cut�. Il n'y a aucune garantie que tous les probl�mes de s�curit� possibles ont �t� identifi�s et que de nouvelles vuln�rabilit�s ont pu �tre d�couvertes depuis l'ex�cution des tests. En tant que tel, ce rapport sert de document d'orientation et non de garantie que le rapport fournit une repr�sentation compl�te des risques mena�ant les syst�mes en question.*

## 2. R�sum� analytique

C'est comme l'elevator pitch du rapport, il vise � fournir aux dirigeants�:

- L'objectif du test.
    - D�crire le besoin m�tier derri�re le test de s�curit�.
    - D�crivez comment les tests ont aid� l'organisation � comprendre ses syst�mes.
- Principaux r�sultats dans un contexte commercial, tels que les �ventuels probl�mes de conformit�, les atteintes � la r�putation, etc. Concentrez-vous sur l'impact commercial et laissez de c�t� les d�tails techniques pour le moment.
- Les recommandations strat�giques sur la mani�re dont l'entreprise peut emp�cher que les probl�mes ne se reproduisent. D�crivez-les dans un contexte non technique et laissez de c�t� les recommandations techniques sp�cifiques pour le moment.

Le r�sum� doit �tre constructif et significatif. �vitez le jargon et les sp�culations n�gatives. Si des figures, des graphiques ou des illustrations sont utilis�s, assurez-vous qu'ils aident � transmettre un message d'une mani�re plus claire que le texte.

## 3. R�sultats

Cette section est destin�e � l'�quipe technique. Il doit inclure toutes les informations n�cessaires pour comprendre la vuln�rabilit�, la r�pliquer et la r�soudre. La s�paration logique peut aider � am�liorer la lisibilit� du rapport. Par exemple, vous pouvez avoir des sections distinctes intitul�es "Acc�s externe" et "Acc�s interne".

S'il s'agit d'un nouveau test, vous pouvez cr�er une sous-section qui r�sume les r�sultats du test pr�c�dent, l'�tat mis � jour des vuln�rabilit�s pr�c�demment identifi�es et toute r�f�rence crois�e avec le test actuel.

### 3.1 R�sum� des r�sultats

Une liste des r�sultats avec leur niveau de risque. Une table peut �tre utilis�e pour faciliter l'utilisation par les deux �quipes.

| R�f. identifiant | Titre | Niveau de risque |
|:-----------:|--------|------------|
| 1 | Contournement de l'authentification utilisateur | Haut |

### 3.2 D�tails des r�sultats

Chaque constatation doit �tre d�taill�e avec les informations suivantes�:

- ID de r�f�rence, qui peut �tre utilis� pour la communication entre les parties et pour les r�f�rences crois�es dans le rapport.
- Le titre de la vuln�rabilit�, tel que "Contournement de l'authentification de l'utilisateur".
- La probabilit� ou l'exploitabilit� du probl�me, en fonction de divers facteurs tels que�:
    - Comme il est facile � exploiter.
    - S'il existe un code d'exploitation fonctionnel pour cela.
    - Le niveau d'acc�s requis.
    - Motivation de l'attaquant pour l'exploiter.
- L'impact de la vuln�rabilit� sur le syst�me.
- Risque de vuln�rabilit� sur l'application.
    - Certaines valeurs sugg�r�es sont�: Informationnel, Faible, Moyen, �lev� et Critique. Assurez-vous de d�tailler les valeurs que vous d�cidez d'utiliser dans une annexe. Cela permet au lecteur de comprendre comment chaque score est d�termin�.
    - Sur certains engagements, il est n�cessaire d'avoir un score [CVSS](https://www.first.org/cvss/). S'il n'est pas n�cessaire, il est parfois bon de l'avoir, et d'autres fois, cela ne fait qu'ajouter de la complexit� au rapport.
- Description d�taill�e de ce qu'est la vuln�rabilit�, comment l'exploiter et les dommages qui peuvent r�sulter de son exploitation. Toutes les donn�es �ventuellement sensibles doivent �tre masqu�es, par exemple, les mots de passe, les informations personnelles ou les d�tails de la carte de cr�dit.
- �tapes d�taill�es sur la fa�on de rem�dier � la vuln�rabilit�, am�liorations possibles qui pourraient aider � renforcer la posture de s�curit� et pratiques de s�curit� manquantes.
- Des ressources suppl�mentaires qui pourraient aider le lecteur � comprendre la vuln�rabilit�, comme une image, une vid�o, un CVE, un guide externe, etc.

Formatez cette section de mani�re � transmettre au mieux votre message.

Assurez-vous toujours que vos descriptions fournissent suffisamment d'informations pour que l'ing�nieur lisant ce rapport puisse prendre des mesures en fonction de celles-ci. Expliquez soigneusement la constatation et fournissez autant de d�tails techniques que n�cessaire pour y rem�dier.

## Annexes

Plusieurs annexes peuvent �tre ajout�es, telles que�:

- M�thodologie de test utilis�e.
- Explications sur la gravit� et la cote de risque.
- Rendement pertinent des outils utilis�s.
    - Assurez-vous de nettoyer la sortie et de ne pas simplement la vider.
- Une checklist de tous les tests effectu�s, comme la [liste de contr�le WSTG](https://github.com/OWASP/wstg/tree/master/checklists). Ceux-ci peuvent �tre fournis en pi�ces jointes au rapport.

## R�f�rences

Cette section ne fait pas partie du format de rapport sugg�r�. Les liens ci-dessous fournissent plus de conseils pour r�diger vos rapports.

- [SANS�: Conseils pour cr�er un rapport d'�valuation de la cybers�curit� solide](https://www.sans.org/blog/tips-for-creating-a-strong-cybersecurity-assessment-report/)
- [SANS�: R�daction d'un rapport de test d'intrusion](https://www.sans.org/reading-room/whitepapers/bestprac/paper/33343)
- [Institut Infosec�: l'art de r�diger des rapports de test d'intrusion](https://resources.infosecinstitute.com/topic/writing-penetration-testing-reports/)
- [Pour les nuls�:�Comment structurer un rapport de test d'intrusion] (https://www.dummies.com/computers/macs/security/how-to-structure-a-pen-test-report/)
- [Rhino Security Labs�: quatre choses que chaque rapport de test d'intrusion devrait avoir] (https://rhinosecuritylabs.com/penetration-testing/four-things-every-penetration-test-report/)
