# Tester le processus d'inscription de l'utilisateur

|ID          |
|------------|
|WSTG-IDNT-02|

## Sommaire

Certains sites Web proposent un processus d'enregistrement des utilisateurs qui automatise (ou semi-automatise) la fourniture de l'accès au système aux utilisateurs. Les exigences d'identité pour l'accès varient d'une identification positive à aucune, selon les exigences de sécurité du système. De nombreuses applications publiques automatisent complètement le processus d'enregistrement et d'approvisionnement car la taille de la base d'utilisateurs rend impossible la gestion manuelle. Cependant, de nombreuses applications d'entreprise provisionnent les utilisateurs manuellement, ce cas de test peut donc ne pas s'appliquer.

## Objectifs des tests

- Vérifiez que les exigences d'identité pour l'enregistrement des utilisateurs sont alignées sur les exigences commerciales et de sécurité.
- Valider le processus d'inscription.

## Comment tester

Vérifiez que les exigences d'identité pour l'enregistrement des utilisateurs sont alignées sur les exigences commerciales et de sécurité :

1. Est-ce que n'importe qui peut s'inscrire pour y accéder ?
2. Les inscriptions sont-elles vérifiées par un humain avant le provisionnement, ou sont-elles automatiquement accordées si les critères sont remplis ?
3. La même personne ou identité peut-elle s'enregistrer plusieurs fois ?
4. Les utilisateurs peuvent-ils s'inscrire pour différents rôles ou autorisations ?
5. Quelle preuve d'identité faut-il pour qu'une inscription aboutisse ?
6. Les identités enregistrées sont-elles vérifiées ?

Validez le processus d'inscription :

1. Les informations d'identité peuvent-elles être facilement falsifiées ou falsifiées ?
2. L'échange d'informations d'identité peut-il être manipulé lors de l'enregistrement ?

### Exemple

Dans l'exemple WordPress ci-dessous, la seule exigence d'identification est une adresse e-mail accessible au titulaire.

![Page d'inscription WordPress](images/Wordpress_registration_page.jpg)\
*Figure 4.3.2-1 : Page d'inscription WordPress*

En revanche, dans l'exemple Google ci-dessous, les exigences d'identification incluent le nom, la date de naissance, le pays, le numéro de téléphone portable, l'adresse e-mail et la réponse CAPTCHA. Alors que seulement deux d'entre eux peuvent être vérifiés (adresse e-mail et numéro de téléphone portable), les exigences d'identification sont plus strictes que WordPress.

![Page d'inscription Google](images/Google_registration_page.jpg)\
*Figure 4.3.2-2 : Page d'inscription Google*

## Correction

Mettre en œuvre des exigences d'identification et de vérification qui correspondent aux exigences de sécurité des informations protégées par les informations d'identification.

## Outils

Un proxy HTTP peut être un outil utile pour tester ce contrôle.

## Références

[Conception d'enregistrement d'utilisateur] (https://mashable.com/2011/06/09/user-registration-design/)
