# Tester le processus de provisionnement du compte

|ID          |
|------------|
|WSTG-IDNT-03|

## Sommaire

L'approvisionnement des comptes pr�sente une opportunit� pour un attaquant de cr�er un compte valide sans appliquer le processus d'identification et d'autorisation appropri�.

## Objectifs des tests

- V�rifiez quels comptes peuvent approvisionner d'autres comptes et de quel type.

## Comment tester

D�terminez quels r�les peuvent provisionner des utilisateurs et quels types de comptes ils peuvent provisionner.

- Existe-t-il une v�rification, un contr�le et une autorisation des demandes de provisionnement�?
- Existe-t-il une v�rification, un contr�le et une autorisation des demandes de d�provisionnement�?
- Un administrateur peut-il provisionner d'autres administrateurs ou uniquement des utilisateurs�?
- Un administrateur ou un autre utilisateur peut-il provisionner des comptes avec des privil�ges sup�rieurs aux siens�?
- Un administrateur ou un utilisateur peut-il se d�provisionner lui-m�me�?
- Comment sont g�r�s les fichiers ou les ressources appartenant � l'utilisateur d�provisionn�? Sont-ils supprim�s ? L'acc�s est-il transf�r� ?

### Exemple

Dans WordPress, seuls le nom et l'adresse e-mail d'un utilisateur sont requis pour provisionner l'utilisateur, comme indiqu� ci-dessous�:

![Ajout utilisateur WordPress](images/Wordpress_useradd.png)\
*Figure 4.3.3-1 : Ajout d'un utilisateur WordPress*

Le d�provisionnement des utilisateurs n�cessite que l'administrateur s�lectionne les utilisateurs � d�provisionner, s�lectionne Supprimer dans le menu d�roulant (encercl�), puis applique cette action. L'administrateur se voit alors pr�senter une bo�te de dialogue lui demandant quoi faire avec les messages de l'utilisateur (les supprimer ou les transf�rer).

![Authentification et utilisateurs WordPress](images/Wordpress_authandusers.png)\
*Figure 4.3.3-2�: Authentification et utilisateurs de WordPress*

## Outils

Bien que l'approche la plus approfondie et la plus pr�cise pour effectuer ce test consiste � le mener manuellement, les outils de proxy HTTP pourraient �galement �tre utiles.
