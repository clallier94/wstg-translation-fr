# Test Nombre de fois qu'une fonction peut �tre utilis�e Limites

|ID          |
|------------|
|WSTG-BUSL-05|

## Sommaire

De nombreux probl�mes que les applications r�solvent n�cessitent des limites quant au nombre de fois qu'une fonction peut �tre utilis�e ou qu'une action peut �tre ex�cut�e. Les applications doivent �tre ��suffisamment intelligentes�� pour ne pas permettre � l'utilisateur de d�passer sa limite d'utilisation de ces fonctions, car dans de nombreux cas, chaque fois que la fonction est utilis�e, l'utilisateur peut obtenir un certain type d'avantage qui doit �tre pris en compte pour compenser correctement le propri�taire. . Par exemple�: un site de commerce �lectronique peut autoriser les utilisateurs � n'appliquer une remise qu'une seule fois par transaction, ou certaines applications peuvent faire l'objet d'un abonnement et autoriser uniquement les utilisateurs � t�l�charger trois�documents complets par mois.

Les vuln�rabilit�s li�es au test des limites de fonction sont sp�cifiques � l'application et des cas d'utilisation abusive doivent �tre cr��s qui s'efforcent d'exercer des parties de l'application/des fonctions/ou des actions plus que le nombre de fois autoris�.

Les attaquants peuvent �tre en mesure de contourner la logique m�tier et d'ex�cuter une fonction plus de fois que ��autoris頻 en exploitant l'application � des fins personnelles.

### Exemple

Supposons qu'un site de commerce �lectronique permette aux utilisateurs de profiter de l'une des nombreuses remises sur leur achat total, puis de proc�der au paiement et � l'appel d'offres. Que se passe-t-il si l'attaquant revient � la page des remises apr�s avoir pris et appliqu� la seule remise "autoris�e"�? Peuvent-ils profiter d'une autre r�duction�? Peuvent-ils profiter plusieurs fois de la m�me remise�?

## Objectifs des tests

- Identifier les fonctions qui doivent fixer des limites aux moments o� elles peuvent �tre appel�es.
- �valuer s'il y a une limite logique fix�e sur les fonctions et si elle est correctement valid�e.

## Comment tester

- Examinez la documentation du projet et utilisez des tests exploratoires pour rechercher des fonctions ou des fonctionnalit�s dans l'application ou le syst�me qui ne doivent pas �tre ex�cut�es plus d'une seule fois ou un nombre sp�cifi� de fois au cours du flux de travail de la logique m�tier.
- Pour chacune des fonctions et fonctionnalit�s trouv�es qui ne doivent �tre ex�cut�es qu'une seule fois ou un nombre de fois sp�cifi� au cours du flux de travail de la logique m�tier, d�veloppez des cas d'abus/de mauvaise utilisation qui peuvent permettre � un utilisateur d'ex�cuter plus que le nombre de fois autoris�. Par exemple, un utilisateur peut-il parcourir plusieurs fois les pages en ex�cutant une fonction qui ne devrait s'ex�cuter qu'une seule fois�? ou un utilisateur peut-il charger et d�charger des paniers d'achat permettant des remises suppl�mentaires.

## Cas de test associ�s

- [Test d'�num�ration de compte et de compte d'utilisateur devinable] (../03-Identity_Management_Testing/04-Testing_for_Account_Enumeration_and_Guessable_User_Account.md)
- [Test du m�canisme de verrouillage faible] (../04-Authentication_Testing/03-Testing_for_Weak_Lock_Out_Mechanism.md)

## Correction

L'application doit d�finir des contr�les stricts pour �viter les abus de limites. Ceci peut �tre r�alis� en d�finissant un coupon pour qu'il ne soit plus valide au niveau de la base de donn�es, afin de d�finir une limite de compteur par utilisateur au niveau du back-end ou de la base de donn�es, car tous les utilisateurs doivent �tre identifi�s via une session, selon ce qui convient le mieux aux besoins de l'entreprise. .

## R�f�rences

- [La logique m�tier d'InfoPath Forms Services a d�pass� la limite maximale d'op�rations] (http://mpwiki.viacode.com/default.aspx?g=posts&t=115678)
- [Le ??commerce de l'or a �t� temporairement interrompu sur le CME ce matin] (https://www.businessinsider.com/gold-halted-on-cme-for-stop-logic-event-2013-10)
