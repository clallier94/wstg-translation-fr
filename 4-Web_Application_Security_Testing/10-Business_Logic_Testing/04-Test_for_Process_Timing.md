# Test de la synchronisation du processus

|ID          |
|------------|
|WSTG-BUSL-04|

## Sommaire

Il est possible que des attaquants puissent recueillir des informations sur une application en surveillant le temps n�cessaire pour accomplir une t�che ou donner une r�ponse. De plus, les attaquants peuvent �tre en mesure de manipuler et de casser les flux de processus m�tier con�us en gardant simplement les sessions actives ouvertes et en ne soumettant pas leurs transactions dans le d�lai ��attendu��.

Les vuln�rabilit�s de la logique de synchronisation des processus sont uniques en ce sens que ces cas d'utilisation abusive manuelle doivent �tre cr��s en tenant compte de la synchronisation de l'ex�cution et des transactions qui sont sp�cifiques � l'application/au syst�me.

Le temps de traitement peut donner/fuir des informations sur ce qui est fait dans les processus d'arri�re-plan de l'application/du syst�me. Si une application permet aux utilisateurs de deviner quel sera le prochain r�sultat particulaire en traitant les variations de temps, les utilisateurs pourront s'adapter en cons�quence et changer de comportement en fonction de l'attente et "jouer avec le syst�me".

### Exemple 1

Les jeux d'argent vid�o/machines � sous peuvent prendre plus de temps pour traiter une transaction juste avant un paiement important. Cela permettrait aux joueurs astucieux de parier des montants minimaux jusqu'� ce qu'ils voient le long temps de traitement qui les inciterait alors � parier le maximum.

### Exemple 2

De nombreux processus de connexion au syst�me demandent le nom d'utilisateur et le mot de passe. Si vous regardez attentivement, vous pourrez peut-�tre voir que la saisie d'un nom d'utilisateur et d'un mot de passe utilisateur invalides prend plus de temps pour renvoyer une erreur que la saisie d'un nom d'utilisateur valide et d'un mot de passe utilisateur invalide. Cela peut permettre � l'attaquant de savoir s'il a un nom d'utilisateur valide et s'il n'a pas besoin de se fier au message de l'interface graphique.

![Exemple de flux de contr�le du formulaire de connexion](images/Control_Flow_of_Login_Form.jpg)\
*Figure 4.10.4-1 : Exemple de flux de contr�le du formulaire de connexion*

### Exemple 3

La plupart des ar�nas ou des agences de voyages ont des applications de billetterie qui permettent aux utilisateurs d'acheter des billets et de r�server des places. Lorsque l'utilisateur demande les billets, les si�ges sont verrouill�s ou r�serv�s en attente de paiement. Que se passe-t-il si un attaquant continue de r�server des si�ges mais ne v�rifie pas�? Les places seront-elles lib�r�es ou aucun billet ne sera-t-il vendu ? Certains vendeurs de billets n'accordent plus que 5 minutes aux utilisateurs pour effectuer une transaction ou la transaction est invalid�e.

### Exemple 4

Supposons qu'un site de commerce �lectronique de m�taux pr�cieux permette aux utilisateurs d'effectuer des achats avec un devis bas� sur le prix du march� au moment o� ils se connectent. Que se passe-t-il si un attaquant se connecte et passe une commande mais ne termine la transaction que plus tard dans la journ�e uniquement lorsque le prix des m�taux augmente�? L'attaquant obtiendra-t-il le prix inf�rieur initial�?

## Objectifs des tests

- Examiner la documentation du projet pour les fonctionnalit�s du syst�me qui peuvent �tre affect�es par le temps.
- D�velopper et ex�cuter des cas d'abus.

## Comment tester

Le testeur doit identifier quels processus sont d�pendants du temps, s'il s'agissait d'une fen�tre pour qu'une t�che soit accomplie, ou s'il s'agissait d'un temps d'ex�cution entre deux processus qui pouvait permettre le contournement de certains contr�les.

Ensuite, il est pr�f�rable d'automatiser les requ�tes qui abuseront des processus d�couverts ci-dessus, car les outils sont mieux adapt�s pour analyser le timing et sont plus pr�cis que les tests manuels. Si cela n'est pas possible, des tests manuels peuvent toujours �tre utilis�s.

Le testeur doit dessiner un sch�ma du d�roulement du processus, des points d'injection, et pr�parer les requ�tes en amont pour les lancer sur les processus vuln�rables. Une fois cela fait, une analyse approfondie doit �tre effectu�e pour identifier les diff�rences dans l'ex�cution du processus et si le processus se comporte mal par rapport � la logique m�tier convenue.

## Cas de test associ�s

- [Test des attributs des cookies](../06-Session_Management_Testing/02-Testing_for_Cookies_Attributes.md)
- [D�lai d'expiration de la session de test] (../06-Session_Management_Testing/07-Testing_Session_Timeout.md)

## Correction

D�veloppez des applications en tenant compte du temps de traitement. Si les attaquants pouvaient �ventuellement tirer un certain avantage en connaissant les diff�rents temps de traitement et r�sultats, ajoutez des �tapes ou un traitement suppl�mentaires afin que, quels que soient les r�sultats, ils soient fournis dans le m�me laps de temps.

De plus, l'application/le syst�me doit avoir un m�canisme en place pour emp�cher les attaquants d'�tendre les transactions sur une dur�e "acceptable". Cela peut �tre fait en annulant ou en r�initialisant les transactions apr�s un laps de temps sp�cifi�, comme certains vendeurs de billets l'utilisent actuellement.
