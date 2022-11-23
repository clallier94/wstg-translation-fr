# Test du m�canisme de verrouillage faible

|ID          |
|------------|
|WSTG-ATHN-03|

## Sommaire

Des m�canismes de verrouillage de compte sont utilis�s pour att�nuer les attaques par force brute. Certaines des attaques qui peuvent �tre vaincues en utilisant le m�canisme de verrouillage�:

- Attaque de mot de passe de connexion ou de devinette de nom d'utilisateur.
- Devinette de code sur toute fonctionnalit� 2FA ou questions de s�curit�.

Les m�canismes de verrouillage de compte n�cessitent un �quilibre entre la protection des comptes contre les acc�s non autoris�s et la protection des utilisateurs contre le refus d'un acc�s autoris�. Les comptes sont g�n�ralement verrouill�s apr�s 3 � 5 tentatives infructueuses et ne peuvent �tre d�verrouill�s qu'apr�s une p�riode de temps pr�d�termin�e, via un m�canisme de d�verrouillage en libre-service ou l'intervention d'un administrateur.

Bien qu'il soit facile de mener des attaques par force brute, le r�sultat d'une attaque r�ussie est dangereux car l'attaquant aura un acc�s complet au compte d'utilisateur et avec lui toutes les fonctionnalit�s et tous les services auxquels il a acc�s.

## Objectifs des tests

- �valuez la capacit� du m�canisme de verrouillage de compte � att�nuer la devinette de mot de passe par force brute.
- �valuer la r�sistance du m�canisme de d�verrouillage au d�verrouillage non autoris� de compte.

## Comment tester

### M�canisme de verrouillage

Pour tester la solidit� des m�canismes de verrouillage, vous aurez besoin d'acc�der � un compte que vous souhaitez ou pouvez vous permettre de verrouiller. Si vous n'avez qu'un seul compte avec lequel vous pouvez vous connecter � l'application Web, effectuez ce test � la fin de votre plan de test pour �viter de perdre du temps de test en �tant bloqu�.

Pour �valuer la capacit� du m�canisme de verrouillage de compte � att�nuer la tentative de deviner un mot de passe par force brute, tentez une connexion non valide en utilisant le mot de passe incorrect un certain nombre de fois, avant d'utiliser le mot de passe correct pour v�rifier que le compte a �t� verrouill�. Un exemple de test peut �tre le suivant�:

1. Essayez de vous connecter 3 fois avec un mot de passe incorrect.
2. Connectez-vous avec succ�s avec le mot de passe correct, montrant ainsi que le m�canisme de verrouillage ne se d�clenche pas apr�s 3 tentatives d'authentification incorrectes.
3. Essayez de vous connecter avec un mot de passe incorrect 4 fois.
4. Connectez-vous avec succ�s avec le mot de passe correct, montrant ainsi que le m�canisme de verrouillage ne se d�clenche pas apr�s 4 tentatives d'authentification incorrectes.
5. Essayez de vous connecter 5 fois avec un mot de passe incorrect.
6. Essayez de vous connecter avec le mot de passe correct. L'application renvoie "Votre compte est verrouill�.", confirmant ainsi que le compte est verrouill� apr�s 5 tentatives d'authentification incorrectes.
7. Essayez de vous connecter avec le mot de passe correct 5 minutes plus tard. L'application renvoie "Votre compte est verrouill�.", indiquant ainsi que le m�canisme de verrouillage ne se d�verrouille pas automatiquement apr�s 5 minutes.
8. Essayez de vous connecter avec le mot de passe correct 10 minutes plus tard. L'application renvoie "Votre compte est verrouill�.", indiquant ainsi que le m�canisme de verrouillage ne se d�verrouille pas automatiquement apr�s 10 minutes.
9. Connectez-vous avec succ�s avec le mot de passe correct 15 minutes plus tard, montrant ainsi que le m�canisme de verrouillage se d�verrouille automatiquement apr�s une p�riode de 10 � 15 minutes.

Un CAPTCHA peut entraver les attaques par force brute, mais il peut s'accompagner de ses propres faiblesses et ne doit pas remplacer un m�canisme de verrouillage. Un m�canisme CAPTCHA peut �tre contourn� s'il est mal impl�ment�. Les failles CAPTCHA incluent�:

1. D�fi facilement vaincu, tel que l'arithm�tique ou un ensemble de questions limit�.
2. CAPTCHA v�rifie le code de r�ponse HTTP au lieu du succ�s de la r�ponse.
3. La logique c�t� serveur CAPTCHA est par d�faut une r�solution r�ussie.
4. Le r�sultat du d�fi CAPTCHA n'est jamais valid� c�t� serveur.
5. Le champ ou le param�tre de saisie CAPTCHA est trait� manuellement et n'est pas correctement valid� ou �chapp�.

Pour �valuer l'efficacit� du CAPTCHA�:

1. �valuez les d�fis CAPTCHA et tentez d'automatiser les solutions en fonction de la difficult�.
2. Essayez de soumettre une demande sans r�soudre le CAPTCHA via le(s) m�canisme(s) d'interface utilisateur normal(s).
3. Tentative de soumission d'une demande avec �chec intentionnel du test CAPTCHA.
4. Essayez de soumettre une demande sans r�soudre CAPTCHA (en supposant que certaines valeurs par d�faut peuvent �tre transmises par le code c�t� client, etc.) tout en utilisant un proxy de test (demande soumise directement c�t� serveur).
5. Essayez de fuzzer les points d'entr�e de donn�es CAPTCHA (le cas �ch�ant) avec des charges utiles d'injection communes ou des s�quences de caract�res sp�ciaux.
6. V�rifiez si la solution au CAPTCHA peut �tre le texte alternatif de l'image ou des noms de fichier ou une valeur dans un champ masqu� associ�.
7. Essayez de soumettre � nouveau les bonnes r�ponses connues pr�c�demment identifi�es.
8. V�rifiez si la suppression des cookies entra�ne le contournement du CAPTCHA (par exemple, si le CAPTCHA ne s'affiche qu'apr�s un certain nombre d'�checs).
9. Si le CAPTCHA fait partie d'un processus en plusieurs �tapes, essayez simplement d'acc�der ou de compl�ter une �tape au-del� du CAPTCHA (par exemple, si le CAPTCHA est la premi�re �tape d'un processus de connexion, essayez simplement de soumettre la deuxi�me �tape [nom d'utilisateur et mot de passe] ).
10. Recherchez d'autres m�thodes qui pourraient ne pas appliquer CAPTCHA, comme un point de terminaison d'API destin� � faciliter l'acc�s aux applications mobiles.

R�p�tez ce processus pour toutes les fonctionnalit�s possibles qui pourraient n�cessiter un m�canisme de verrouillage.

### M�canisme de d�verrouillage

Pour �valuer la r�sistance du m�canisme de d�verrouillage au d�verrouillage de compte non autoris�, lancez le m�canisme de d�verrouillage et recherchez les faiblesses. Les m�canismes de d�verrouillage typiques peuvent impliquer des questions secr�tes ou un lien de d�verrouillage envoy� par e-mail. Le lien de d�verrouillage doit �tre un lien unique unique, pour emp�cher un attaquant de deviner ou de rejouer le lien et d'effectuer des attaques par force brute par lots.

Notez qu'un m�canisme de d�verrouillage ne doit �tre utilis� que pour d�verrouiller des comptes. Ce n'est pas la m�me chose qu'un m�canisme de r�cup�ration de mot de passe, mais pourrait suivre les m�mes pratiques de s�curit�.

## Correction

Appliquez des m�canismes de d�verrouillage de compte en fonction du niveau de risque. Dans l'ordre de l'assurance la plus faible � la plus �lev�e�:

1. Verrouillage et d�verrouillage bas�s sur le temps.
2. D�verrouillage en libre-service (envoie un e-mail de d�verrouillage � l'adresse e-mail enregistr�e).
3. D�verrouillage manuel de l'administrateur.
4. D�verrouillage manuel de l'administrateur avec identification positive de l'utilisateur.

Facteurs � prendre en compte lors de la mise en �uvre d'un m�canisme de verrouillage de compte�:

1. Quel est le risque de deviner un mot de passe par force brute contre l'application�?
2. Un CAPTCHA est-il suffisant pour att�nuer ce risque ?
3. Un m�canisme de verrouillage c�t� client est-il utilis� (par exemple, JavaScript)�? (Si c'est le cas, d�sactivez le code c�t� client � tester.)
4. Nombre de tentatives de connexion infructueuses avant le verrouillage. Si le seuil de verrouillage est trop bas, les utilisateurs valides peuvent �tre verrouill�s trop souvent. Si le seuil de verrouillage est trop �lev�, plus un attaquant peut tenter de forcer brutalement le compte avant qu'il ne soit verrouill�. Selon l'objectif de l'application, une plage de 5 � 10 tentatives infructueuses est un seuil de verrouillage typique.
5. Comment les comptes seront-ils d�verrouill�s�?
    1. Manuellement par un administrateur : il s'agit de la m�thode de verrouillage la plus s�curis�e, mais elle peut causer des d�sagr�ments aux utilisateurs et accaparer le temps "pr�cieux" de l'administrateur.
        1. Notez que l'administrateur doit �galement disposer d'une m�thode de r�cup�ration au cas o� son compte serait verrouill�.
        2. Ce m�canisme de d�verrouillage peut conduire � une attaque par d�ni de service si l'objectif d'un attaquant est de verrouiller les comptes de tous les utilisateurs de l'application Web.
    2. Apr�s un certain temps : Quelle est la dur�e du blocage ? Est-ce suffisant pour que l'application soit prot�g�e ? Par exemple. une dur�e de verrouillage de 5 � 30 minutes peut �tre un bon compromis entre att�nuer les attaques par force brute et g�ner les utilisateurs valides.
    3. Via un m�canisme de libre-service : Comme indiqu� pr�c�demment, ce m�canisme de libre-service doit �tre suffisamment s�curis� pour �viter que l'attaquant puisse d�verrouiller lui-m�me les comptes.

## R�f�rences

- Voir l'article de l'OWASP sur les attaques [Brute Force](https://owasp.org/www-community/attacks/Brute_force_attack).
- [Mot de passe oubli� CS] (https://cheatsheetseries.owasp.org/cheatsheets/Forgot_Password_Cheat_Sheet.html).
