# 4.0 Introduction et objectifs

Cette section d�crit la m�thodologie de test de s�curit� des applications Web OWASP et explique comment tester les preuves de vuln�rabilit�s au sein de l'application en raison de lacunes dans les contr�les de s�curit� identifi�s.

## Qu'est-ce que le test de s�curit� des applications Web�?

Un test de s�curit� est une m�thode d'�valuation de la s�curit� d'un syst�me informatique ou d'un r�seau en validant et en v�rifiant m�thodiquement l'efficacit� des contr�les de s�curit� des applications. Un test de s�curit� d'application Web se concentre uniquement sur l'�valuation de la s�curit� d'une application Web. Le processus implique une analyse active de l'application pour d�tecter toute faiblesse, d�faut technique ou vuln�rabilit�. Tout probl�me de s�curit� d�tect� sera pr�sent� au propri�taire du syst�me, accompagn� d'une �valuation de l'impact, d'une proposition d'att�nuation ou d'une solution technique.

## Qu'est-ce qu'une vuln�rabilit�?

Une vuln�rabilit� est une faille ou une faiblesse dans la conception, la mise en �uvre, le fonctionnement ou la gestion d'un syst�me qui pourrait �tre exploit�e pour compromettre les objectifs de s�curit� du syst�me.

## Qu'est-ce qu'une menace�?

Une menace est tout (un attaquant externe malveillant, un utilisateur interne, une instabilit� du syst�me, etc.) vuln�rabilit�.

## Qu'est-ce qu'un essai�?

Un test est une action visant � d�montrer qu'une application r�pond aux exigences de s�curit� de ses parties prenantes.

## L'approche dans la r�daction de ce guide

L'approche OWASP est ouverte et collaborative :

- Ouvert : chaque expert en s�curit� peut participer avec son exp�rience au projet. Tout est gratuit.
- Collaboratif : un brainstorming est effectu� avant la r�daction des articles afin que l'�quipe puisse partager des id�es et d�velopper une vision collective du projet. Cela signifie un consensus approximatif, un public plus large et une participation accrue.

Cette approche tend � cr�er une m�thodologie de test d�finie qui sera�:

- Coh�rent
- Reproductible
- Rigoureux
- Sous contr�le qualit�

Les probl�mes � r�soudre sont enti�rement document�s et test�s. Il est important d'utiliser une m�thode pour tester toutes les vuln�rabilit�s connues et documenter toutes les activit�s de test de s�curit�.

## Qu'est-ce que la m�thodologie de test OWASP�?

Les tests de s�curit� ne seront jamais une science exacte o� une liste compl�te de tous les probl�mes possibles qui devraient �tre test�s peut �tre d�finie. En effet, les tests de s�curit� ne sont une technique appropri�e pour tester la s�curit� des applications Web que dans certaines circonstances. Le but de ce projet est de rassembler toutes les techniques de test possibles, d'expliquer ces techniques et de maintenir le guide � jour. La m�thode de test de s�curit� des applications Web OWASP est bas�e sur l'approche de la bo�te noire. Le testeur ne sait rien ou a tr�s peu d'informations sur l'application � tester.

Le mod�le de test se compose de�:

- Testeur�: Qui effectue les activit�s de test
- Outils et m�thodologie�: le c�ur de ce projet de guide de test
- Application : La bo�te noire � tester

Les tests peuvent �tre class�s comme passifs ou actifs�:

### Tests passifs

Lors des tests passifs, un testeur essaie de comprendre la logique de l'application et explore l'application en tant qu'utilisateur. Des outils peuvent �tre utilis�s pour la collecte d'informations. Par exemple, un proxy HTTP peut �tre utilis� pour observer toutes les requ�tes et r�ponses HTTP. � la fin de cette phase, le testeur doit g�n�ralement comprendre tous les points d'acc�s et toutes les fonctionnalit�s du syst�me (par exemple, les en-t�tes HTTP, les param�tres, les cookies, les API, l'utilisation/les mod�les de technologie, etc.). La section [Collecte d'informations](../01-Information_Gathering/README.md) explique comment effectuer des tests passifs.

Par exemple, un testeur peut trouver une page � l'URL suivante�: `https://www.exemple.com/login/auth_form`

Cela peut indiquer un formulaire d'authentification o� l'application demande un nom d'utilisateur et un mot de passe.

Les param�tres suivants repr�sentent deux points d'acc�s � l'application�: `https://www.exemple.com/appx?a=1&b=1`

Dans ce cas, l'application affiche deux points d'acc�s (param�tres 'a' et 'b'). Tous les points d'entr�e trouv�s dans cette phase repr�sentent une cible pour les tests. Garder une trace du r�pertoire ou de l'arborescence des appels de l'application et de tous les points d'acc�s peut �tre utile pendant les tests actifs.

###�Tests actifs

Pendant les tests actifs, un testeur commence � utiliser les m�thodologies d�crites dans les sections suivantes.

L'ensemble des tests actifs a �t� divis� en 12 cat�gories�:

- La collecte d'informations
- Tests de gestion de configuration et de d�ploiement
- Tests de gestion d'identit�
- Tests d'authentification
- Tests d'autorisation
- Test de gestion de session
- Test de validation des entr�es
- La gestion des erreurs
- Cryptographie
- Test de logique m�tier
- Tests c�t� client
- Tests d'API
