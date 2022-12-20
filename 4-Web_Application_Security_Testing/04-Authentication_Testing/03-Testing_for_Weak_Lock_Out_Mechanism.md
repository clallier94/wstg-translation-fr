# Test du mécanisme de verrouillage faible

|ID          |
|------------|
|WSTG-ATHN-03|

## Sommaire

Des mécanismes de verrouillage de compte sont utilisés pour atténuer les attaques par force brute. Certaines des attaques qui peuvent être vaincues en utilisant le mécanisme de verrouillage :

- Attaque de mot de passe de connexion ou de devinette de nom d'utilisateur.
- Devinette de code sur toute fonctionnalité 2FA ou questions de sécurité.

Les mécanismes de verrouillage de compte nécessitent un équilibre entre la protection des comptes contre les accès non autorisés et la protection des utilisateurs contre le refus d'un accès autorisé. Les comptes sont généralement verrouillés après 3 à 5 tentatives infructueuses et ne peuvent être déverrouillés qu'après une période de temps prédéterminée, via un mécanisme de déverrouillage en libre-service ou l'intervention d'un administrateur.

Bien qu'il soit facile de mener des attaques par force brute, le résultat d'une attaque réussie est dangereux car l'attaquant aura un accès complet au compte d'utilisateur et avec lui toutes les fonctionnalités et tous les services auxquels il a accès.

## Objectifs des tests

- Évaluez la capacité du mécanisme de verrouillage de compte à atténuer la devinette de mot de passe par force brute.
- Évaluer la résistance du mécanisme de déverrouillage au déverrouillage non autorisé de compte.

## Comment tester

### Mécanisme de verrouillage

Pour tester la solidité des mécanismes de verrouillage, vous aurez besoin d'accéder à un compte que vous souhaitez ou pouvez vous permettre de verrouiller. Si vous n'avez qu'un seul compte avec lequel vous pouvez vous connecter à l'application Web, effectuez ce test à la fin de votre plan de test pour éviter de perdre du temps de test en étant bloqué.

Pour évaluer la capacité du mécanisme de verrouillage de compte à atténuer la tentative de deviner un mot de passe par force brute, tentez une connexion non valide en utilisant le mot de passe incorrect un certain nombre de fois, avant d'utiliser le mot de passe correct pour vérifier que le compte a été verrouillé. Un exemple de test peut être le suivant :

1. Essayez de vous connecter 3 fois avec un mot de passe incorrect.
2. Connectez-vous avec succès avec le mot de passe correct, montrant ainsi que le mécanisme de verrouillage ne se déclenche pas après 3 tentatives d'authentification incorrectes.
3. Essayez de vous connecter avec un mot de passe incorrect 4 fois.
4. Connectez-vous avec succès avec le mot de passe correct, montrant ainsi que le mécanisme de verrouillage ne se déclenche pas après 4 tentatives d'authentification incorrectes.
5. Essayez de vous connecter 5 fois avec un mot de passe incorrect.
6. Essayez de vous connecter avec le mot de passe correct. L'application renvoie "Votre compte est verrouillé.", confirmant ainsi que le compte est verrouillé après 5 tentatives d'authentification incorrectes.
7. Essayez de vous connecter avec le mot de passe correct 5 minutes plus tard. L'application renvoie "Votre compte est verrouillé.", indiquant ainsi que le mécanisme de verrouillage ne se déverrouille pas automatiquement après 5 minutes.
8. Essayez de vous connecter avec le mot de passe correct 10 minutes plus tard. L'application renvoie "Votre compte est verrouillé.", indiquant ainsi que le mécanisme de verrouillage ne se déverrouille pas automatiquement après 10 minutes.
9. Connectez-vous avec succès avec le mot de passe correct 15 minutes plus tard, montrant ainsi que le mécanisme de verrouillage se déverrouille automatiquement après une période de 10 à 15 minutes.

Un CAPTCHA peut entraver les attaques par force brute, mais il peut s'accompagner de ses propres faiblesses et ne doit pas remplacer un mécanisme de verrouillage. Un mécanisme CAPTCHA peut être contourné s'il est mal implémenté. Les failles CAPTCHA incluent :

1. Défi facilement vaincu, tel que l'arithmétique ou un ensemble de questions limité.
2. CAPTCHA vérifie le code de réponse HTTP au lieu du succès de la réponse.
3. La logique côté serveur CAPTCHA est par défaut une résolution réussie.
4. Le résultat du défi CAPTCHA n'est jamais validé côté serveur.
5. Le champ ou le paramètre de saisie CAPTCHA est traité manuellement et n'est pas correctement validé ou échappé.

Pour évaluer l'efficacité du CAPTCHA :

1. Évaluez les défis CAPTCHA et tentez d'automatiser les solutions en fonction de la difficulté.
2. Essayez de soumettre une demande sans résoudre le CAPTCHA via le(s) mécanisme(s) d'interface utilisateur normal(s).
3. Tentative de soumission d'une demande avec échec intentionnel du test CAPTCHA.
4. Essayez de soumettre une demande sans résoudre CAPTCHA (en supposant que certaines valeurs par défaut peuvent être transmises par le code côté client, etc.) tout en utilisant un proxy de test (demande soumise directement côté serveur).
5. Essayez de fuzzer les points d'entrée de données CAPTCHA (le cas échéant) avec des charges utiles d'injection communes ou des séquences de caractères spéciaux.
6. Vérifiez si la solution au CAPTCHA peut être le texte alternatif de l'image ou des noms de fichier ou une valeur dans un champ masqué associé.
7. Essayez de soumettre à nouveau les bonnes réponses connues précédemment identifiées.
8. Vérifiez si la suppression des cookies entraîne le contournement du CAPTCHA (par exemple, si le CAPTCHA ne s'affiche qu'après un certain nombre d'échecs).
9. Si le CAPTCHA fait partie d'un processus en plusieurs étapes, essayez simplement d'accéder ou de compléter une étape au-delà du CAPTCHA (par exemple, si le CAPTCHA est la première étape d'un processus de connexion, essayez simplement de soumettre la deuxième étape [nom d'utilisateur et mot de passe] ).
10. Recherchez d'autres méthodes qui pourraient ne pas appliquer CAPTCHA, comme un point de terminaison d'API destiné à faciliter l'accès aux applications mobiles.

Répétez ce processus pour toutes les fonctionnalités possibles qui pourraient nécessiter un mécanisme de verrouillage.

### Mécanisme de déverrouillage

Pour évaluer la résistance du mécanisme de déverrouillage au déverrouillage de compte non autorisé, lancez le mécanisme de déverrouillage et recherchez les faiblesses. Les mécanismes de déverrouillage typiques peuvent impliquer des questions secrètes ou un lien de déverrouillage envoyé par e-mail. Le lien de déverrouillage doit être un lien unique unique, pour empêcher un attaquant de deviner ou de rejouer le lien et d'effectuer des attaques par force brute par lots.

Notez qu'un mécanisme de déverrouillage ne doit être utilisé que pour déverrouiller des comptes. Ce n'est pas la même chose qu'un mécanisme de récupération de mot de passe, mais pourrait suivre les mêmes pratiques de sécurité.

## Correction

Appliquez des mécanismes de déverrouillage de compte en fonction du niveau de risque. Dans l'ordre de l'assurance la plus faible à la plus élevée :

1. Verrouillage et déverrouillage basés sur le temps.
2. Déverrouillage en libre-service (envoie un e-mail de déverrouillage à l'adresse e-mail enregistrée).
3. Déverrouillage manuel de l'administrateur.
4. Déverrouillage manuel de l'administrateur avec identification positive de l'utilisateur.

Facteurs à prendre en compte lors de la mise en œuvre d'un mécanisme de verrouillage de compte :

1. Quel est le risque de deviner un mot de passe par force brute contre l'application ?
2. Un CAPTCHA est-il suffisant pour atténuer ce risque ?
3. Un mécanisme de verrouillage côté client est-il utilisé (par exemple, JavaScript) ? (Si c'est le cas, désactivez le code côté client à tester.)
4. Nombre de tentatives de connexion infructueuses avant le verrouillage. Si le seuil de verrouillage est trop bas, les utilisateurs valides peuvent être verrouillés trop souvent. Si le seuil de verrouillage est trop élevé, plus un attaquant peut tenter de forcer brutalement le compte avant qu'il ne soit verrouillé. Selon l'objectif de l'application, une plage de 5 à 10 tentatives infructueuses est un seuil de verrouillage typique.
5. Comment les comptes seront-ils déverrouillés ?
    1. Manuellement par un administrateur : il s'agit de la méthode de verrouillage la plus sécurisée, mais elle peut causer des désagréments aux utilisateurs et accaparer le temps "précieux" de l'administrateur.
        1. Notez que l'administrateur doit également disposer d'une méthode de récupération au cas où son compte serait verrouillé.
        2. Ce mécanisme de déverrouillage peut conduire à une attaque par déni de service si l'objectif d'un attaquant est de verrouiller les comptes de tous les utilisateurs de l'application Web.
    2. Après un certain temps : Quelle est la durée du blocage ? Est-ce suffisant pour que l'application soit protégée ? Par exemple. une durée de verrouillage de 5 à 30 minutes peut être un bon compromis entre atténuer les attaques par force brute et gêner les utilisateurs valides.
    3. Via un mécanisme de libre-service : Comme indiqué précédemment, ce mécanisme de libre-service doit être suffisamment sécurisé pour éviter que l'attaquant puisse déverrouiller lui-même les comptes.

## Références

- Voir l'article de l'OWASP sur les attaques [Brute Force](https://owasp.org/www-community/attacks/Brute_force_attack).
- [Mot de passe oublié CS](https://cheatsheetseries.owasp.org/cheatsheets/Forgot_Password_Cheat_Sheet.html).
