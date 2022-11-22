# Tester le processus d'inscription de l'utilisateur

|ID          |
|------------|
|WSTG-IDNT-02|

## Sommaire

Certains sites Web proposent un processus d'enregistrement des utilisateurs qui automatise (ou semi-automatise) la fourniture de l'acc�s au syst�me aux utilisateurs. Les exigences d'identit� pour l'acc�s varient d'une identification positive � aucune, selon les exigences de s�curit� du syst�me. De nombreuses applications publiques automatisent compl�tement le processus d'enregistrement et d'approvisionnement car la taille de la base d'utilisateurs rend impossible la gestion manuelle. Cependant, de nombreuses applications d'entreprise provisionnent les utilisateurs manuellement, ce cas de test peut donc ne pas s'appliquer.

## Objectifs des tests

- V�rifiez que les exigences d'identit� pour l'enregistrement des utilisateurs sont align�es sur les exigences commerciales et de s�curit�.
- Valider le processus d'inscription.

## Comment tester

V�rifiez que les exigences d'identit� pour l'enregistrement des utilisateurs sont align�es sur les exigences commerciales et de s�curit�:

1. Est-ce que n'importe qui peut s'inscrire pour y acc�der ?
2. Les inscriptions sont-elles v�rifi�es par un humain avant le provisionnement, ou sont-elles automatiquement accord�es si les crit�res sont remplis�?
3. La m�me personne ou identit� peut-elle s'enregistrer plusieurs fois ?
4. Les utilisateurs peuvent-ils s'inscrire pour diff�rents r�les ou autorisations�?
5. Quelle preuve d'identit� faut-il pour qu'une inscription aboutisse ?
6. Les identit�s enregistr�es sont-elles v�rifi�es ?

Validez le processus d'inscription :

1. Les informations d'identit� peuvent-elles �tre facilement falsifi�es ou falsifi�es�?
2. L'�change d'informations d'identit� peut-il �tre manipul� lors de l'enregistrement ?

### Exemple

Dans l'exemple WordPress ci-dessous, la seule exigence d'identification est une adresse e-mail accessible au titulaire.

![Page d'inscription WordPress](images/Wordpress_registration_page.jpg)\
*Figure 4.3.2-1 : Page d'inscription WordPress*

En revanche, dans l'exemple Google ci-dessous, les exigences d'identification incluent le nom, la date de naissance, le pays, le num�ro de t�l�phone portable, l'adresse e-mail et la r�ponse CAPTCHA. Alors que seulement deux d'entre eux peuvent �tre v�rifi�s (adresse e-mail et num�ro de t�l�phone portable), les exigences d'identification sont plus strictes que WordPress.

![Page d'inscription Google](images/Google_registration_page.jpg)\
*Figure 4.3.2-2�: Page d'inscription Google*

## Correction

Mettre en �uvre des exigences d'identification et de v�rification qui correspondent aux exigences de s�curit� des informations prot�g�es par les informations d'identification.

## Outils

Un proxy HTTP peut �tre un outil utile pour tester ce contr�le.

## R�f�rences

[Conception d'enregistrement d'utilisateur] (https://mashable.com/2011/06/09/user-registration-design/)
