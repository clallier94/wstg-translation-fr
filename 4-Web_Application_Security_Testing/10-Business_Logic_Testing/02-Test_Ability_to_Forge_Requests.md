# Tester la capacit� � falsifier des requ�tes

|ID          |
|------------|
|WSTG-BUSL-02|

## Sommaire

Les requ�tes falsifi�es sont une m�thode utilis�e par les attaquants pour contourner l'application GUI frontale afin de soumettre directement des informations pour le traitement back-end. L'objectif de l'attaquant est d'envoyer des requ�tes HTTP POST/GET via un proxy d'interception avec des valeurs de donn�es qui ne sont pas prises en charge, prot�g�es ou attendues par la logique m�tier des applications. Certains exemples de requ�tes falsifi�es incluent l'exploitation de param�tres devinables ou pr�visibles ou l'exposition de fonctionnalit�s et fonctionnalit�s "cach�es" telles que l'activation du d�bogage ou la pr�sentation d'�crans ou de fen�tres sp�ciaux qui sont tr�s utiles pendant le d�veloppement mais qui peuvent divulguer des informations ou contourner la logique m�tier.

Les vuln�rabilit�s li�es � la capacit� de falsifier des requ�tes sont propres � chaque application et diff�rentes de la validation des donn�es de la logique m�tier en ce sens qu'elles se concentrent sur la rupture du flux de travail de la logique m�tier.

Les applications doivent avoir mis en place des v�rifications logiques pour emp�cher le syst�me d'accepter des demandes falsifi�es qui pourraient permettre aux attaquants d'exploiter la logique m�tier, le processus ou le flux de l'application. La contrefa�on de demande n'est pas nouvelle ; l'attaquant utilise un proxy d'interception pour envoyer des requ�tes HTTP POST/GET � l'application. Gr�ce aux falsifications de requ�tes, les attaquants peuvent contourner la logique ou le processus m�tier en trouvant, pr�disant et manipulant des param�tres pour faire croire � l'application qu'un processus ou une t�che a ou n'a pas eu lieu.

En outre, les requ�tes falsifi�es peuvent permettre de subventionner le flux de logique programmatique ou m�tier en invoquant des fonctionnalit�s ou fonctionnalit�s "cach�es" telles que le d�bogage initialement utilis� par les d�veloppeurs et les testeurs, parfois appel� ["�uf de P�ques"](http://en.wikipedia. org/wiki/Easter_egg_(media)). "Un �uf de P�ques est une blague int�rieure intentionnelle, un message cach� ou une fonctionnalit� dans une �uvre telle qu'un programme informatique, un film, un livre ou des mots crois�s. Selon le concepteur de jeux Warren Robinett, le terme a �t� invent� chez Atari par le personnel qui a �t� alert� de la pr�sence d'un message secret qui avait �t� cach� par Robinett dans son jeu d�j� largement diffus�, Adventure. On a dit que le nom �voquait l'id�e d'une chasse aux �ufs de P�ques traditionnelle.

### Exemple 1

Supposons qu'un site de th��tre e-commerce permette aux utilisateurs de s�lectionner leur billet, d'appliquer une remise unique de 10�% sur l'ensemble de la vente, d'afficher le sous-total et de proposer la vente. Si un attaquant est capable de voir � travers un proxy que l'application a un champ cach� (de 1 ou 0) utilis� par la logique m�tier pour d�terminer si une remise a �t� prise ou non. L'attaquant est alors en mesure de soumettre plusieurs fois la valeur 1 ou "aucune remise n'a �t� prise" pour profiter de la m�me remise plusieurs fois.

### Exemple 2

Supposons qu'un jeu vid�o en ligne paie des jetons pour les points marqu�s pour trouver des tr�sors de pirates et des pirates et pour chaque niveau termin�. Ces jetons peuvent ensuite �tre �chang�s contre des prix. De plus, les points de chaque niveau ont une valeur multiplicatrice �gale au niveau. Si un attaquant pouvait voir � travers un proxy que l'application a un champ cach� utilis� pendant le d�veloppement et les tests pour acc�der rapidement aux niveaux les plus �lev�s du jeu, il pourrait rapidement atteindre les niveaux les plus �lev�s et accumuler rapidement des points non gagn�s.

De plus, si un attaquant pouvait voir via un proxy que l'application a un champ cach� utilis� pendant le d�veloppement et les tests pour activer un journal indiquant o� d'autres joueurs en ligne ou un tr�sor cach� se trouvaient en relation avec l'attaquant, il pourrait alors pour aller rapidement � ces endroits et marquer des points.

## Objectifs des tests

- Examinez la documentation du projet � la recherche de fonctionnalit�s devinables, pr�visibles ou masqu�es des champs.
- Ins�rez des donn�es logiquement valides afin de contourner le flux de travail normal de la logique m�tier.

## Comment tester

### En identifiant des valeurs devinables

- � l'aide d'un proxy d'interception, observez le HTTP POST/GET � la recherche d'indications indiquant que les valeurs s'incr�mentent � intervalles r�guliers ou sont facilement devinables.
- S'il s'av�re qu'une certaine valeur est devinable, cette valeur peut �tre modifi�e et on peut obtenir une visibilit� inattendue.

### En identifiant les options cach�es

- � l'aide d'un proxy d'interception, observez le HTTP POST/GET � la recherche d'indications de fonctionnalit�s cach�es telles que le d�bogage qui peuvent �tre activ�es ou activ�es.
- Si vous en trouvez, essayez de deviner et modifiez ces valeurs pour obtenir une r�ponse ou un comportement diff�rent de l'application.

## Cas de test associ�s

- [Test des variables de session expos�es](../06-Session_Management_Testing/04-Testing_for_Exposed_Session_Variables.md)
- [Test de falsification de requ�te intersite (CSRF)](../06-Session_Management_Testing/05-Testing_for_Cross_Site_Request_Forgery.md)
- [Test d'�num�ration de compte et de compte d'utilisateur devinable] (../03-Identity_Management_Testing/04-Testing_for_Account_Enumeration_and_Guessable_User_Account.md)

## Correction

L'application doit �tre suffisamment intelligente et con�ue avec une logique m�tier qui emp�chera les attaquants de pr�dire et de manipuler les param�tres pour subvertir le flux de logique programmatique ou m�tier, ou d'exploiter des fonctionnalit�s cach�es/non document�es telles que le d�bogage.

## Outils

- [Proxy d'attaque Zed OWASP (ZAP)] (https://www.zaproxy.org)
- [Burp Suite] (https://portswigger.net/burp)

## R�f�rences

- [Cross Site Request Forgery - L�gitimation des fausses requ�tes](http://www.stan.gr/2012/11/cross-site-request-forgery-legitimazing.html)
- [Easter Eggs](https://en.wikipedia.org/wiki/Easter_egg_(media))
- [Top 10 des Easter Eggs logiciels] (https://lifehacker.com/371083/top-10-software-easter-eggs)
