# Test de la politique de mot de passe faible

|ID          |
|------------|
|WSTG-ATHN-07|

## Sommaire

Le m�canisme d'authentification le plus r�pandu et le plus facilement administr� est un mot de passe statique. Le mot de passe repr�sente les cl�s du royaume, mais est souvent d�tourn� par les utilisateurs au nom de la convivialit�. Dans chacun des hacks r�cents de haut niveau qui ont r�v�l� les informations d'identification des utilisateurs, il est d�plor� que les mots de passe les plus courants soient toujours�: `123456`, `password` et `qwerty`.

## Objectifs des tests

- D�terminer la r�sistance de l'application contre la force brute de deviner les mots de passe � l'aide des dictionnaires de mots de passe disponibles en �valuant la longueur, la complexit�, la r�utilisation et les exigences de vieillissement des mots de passe.

## Comment tester

1. Quels caract�res sont autoris�s et interdits dans un mot de passe�? L'utilisateur doit-il utiliser des caract�res de diff�rents jeux de caract�res tels que des lettres minuscules et majuscules, des chiffres et des symboles sp�ciaux�?
2. � quelle fr�quence un utilisateur peut-il changer son mot de passe�? Dans quel d�lai un utilisateur peut-il changer son mot de passe apr�s un changement pr�c�dent�? Les utilisateurs peuvent contourner les exigences d'historique de mot de passe en modifiant leur mot de passe 5 fois de suite afin qu'apr�s le dernier changement de mot de passe, ils aient � nouveau configur� leur mot de passe initial.
3. Quand un utilisateur doit-il changer son mot de passe ?
    - � la fois [NIST](https://pages.nist.gov/800-63-3/sp800-63b.html#memsecretver) et [NCSC](https://www.ncsc.gov.uk/collection/passwords/updating-your-approach#PasswordGuidance:UpdatingYourApproach-Don'tenforceregularpasswordexpiry) recommande **contre** de forcer l'expiration r�guli�re du mot de passe, bien que cela puisse �tre requis par des normes telles que PCI DSS.
4. � quelle fr�quence un utilisateur peut-il r�utiliser un mot de passe�? L'application conserve-t-elle un historique des 8 derniers mots de passe utilis�s par l'utilisateur�?
5. Dans quelle mesure le prochain mot de passe doit-il �tre diff�rent du dernier mot de passe�?
6. L'utilisateur est-il emp�ch� d'utiliser son nom d'utilisateur ou d'autres informations de compte (comme le pr�nom ou le nom) dans le mot de passe ?
7. Quelles sont les longueurs de mot de passe minimale et maximale pouvant �tre d�finies, et sont-elles adapt�es � la sensibilit� du compte et de l'application ?
8. Est-il possible de d�finir des mots de passe communs tels que `Password1` ou `123456`�?

## Correction

Pour att�nuer le risque que des mots de passe faciles � deviner facilitent un acc�s non autoris�, il existe deux solutions�: introduire des contr�les d'authentification suppl�mentaires (c'est-�-dire une authentification � deux facteurs) ou introduire une politique de mots de passe forts. La plus simple et la moins ch�re d'entre elles est l'introduction d'une politique de mot de passe forte qui garantit la longueur, la complexit�, la r�utilisation et le vieillissement du mot de passe�; m�me si id�alement les deux devraient �tre mis en �uvre.

## R�f�rences

- [Attaques par force brute] (https://owasp.org/www-community/attacks/Brute_force_attack)