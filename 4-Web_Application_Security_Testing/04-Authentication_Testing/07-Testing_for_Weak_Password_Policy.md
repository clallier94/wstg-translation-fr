# Test de la politique de mot de passe faible

|ID          |
|------------|
|WSTG-ATHN-07|

## Sommaire

Le mécanisme d'authentification le plus répandu et le plus facilement administré est un mot de passe statique. Le mot de passe représente les clés du royaume, mais est souvent détourné par les utilisateurs au nom de la convivialité. Dans chacun des hacks récents de haut niveau qui ont révélé les informations d'identification des utilisateurs, il est déploré que les mots de passe les plus courants soient toujours : `123456`, `password` et `qwerty`.

## Objectifs des tests

- Déterminer la résistance de l'application contre la force brute de deviner les mots de passe à l'aide des dictionnaires de mots de passe disponibles en évaluant la longueur, la complexité, la réutilisation et les exigences de vieillissement des mots de passe.

## Comment tester

1. Quels caractères sont autorisés et interdits dans un mot de passe ? L'utilisateur doit-il utiliser des caractères de différents jeux de caractères tels que des lettres minuscules et majuscules, des chiffres et des symboles spéciaux ?
2. À quelle fréquence un utilisateur peut-il changer son mot de passe ? Dans quel délai un utilisateur peut-il changer son mot de passe après un changement précédent ? Les utilisateurs peuvent contourner les exigences d'historique de mot de passe en modifiant leur mot de passe 5 fois de suite afin qu'après le dernier changement de mot de passe, ils aient à nouveau configuré leur mot de passe initial.
3. Quand un utilisateur doit-il changer son mot de passe ?
    - À la fois [NIST](https://pages.nist.gov/800-63-3/sp800-63b.html#memsecretver) et [NCSC](https://www.ncsc.gov.uk/collection/passwords/updating-your-approach#PasswordGuidance:UpdatingYourApproach-Don'tenforceregularpasswordexpiry) recommande **contre** de forcer l'expiration régulière du mot de passe, bien que cela puisse être requis par des normes telles que PCI DSS.
4. À quelle fréquence un utilisateur peut-il réutiliser un mot de passe ? L'application conserve-t-elle un historique des 8 derniers mots de passe utilisés par l'utilisateur ?
5. Dans quelle mesure le prochain mot de passe doit-il être différent du dernier mot de passe ?
6. L'utilisateur est-il empêché d'utiliser son nom d'utilisateur ou d'autres informations de compte (comme le prénom ou le nom) dans le mot de passe ?
7. Quelles sont les longueurs de mot de passe minimale et maximale pouvant être définies, et sont-elles adaptées à la sensibilité du compte et de l'application ?
8. Est-il possible de définir des mots de passe communs tels que `Password1` ou `123456` ?

## Correction

Pour atténuer le risque que des mots de passe faciles à deviner facilitent un accès non autorisé, il existe deux solutions : introduire des contrôles d'authentification supplémentaires (c'est-à-dire une authentification à deux facteurs) ou introduire une politique de mots de passe forts. La plus simple et la moins chère d'entre elles est l'introduction d'une politique de mot de passe forte qui garantit la longueur, la complexité, la réutilisation et le vieillissement du mot de passe ; même si idéalement les deux devraient être mis en œuvre.

## Références

- [Attaques par force brute](https://owasp.org/www-community/attacks/Brute_force_attack)
