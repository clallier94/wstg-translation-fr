# Tester la politique de nom d'utilisateur faible ou non appliquée

|ID          |
|------------|
|WSTG-IDNT-05|

## Sommaire

Les noms de compte utilisateur sont souvent très structurés (par exemple, le nom du compte Joe Bloggs est jbloggs et le nom du compte Fred Nurks est fnurks) et les noms de compte valides peuvent facilement être devinés.

## Objectifs des tests

- Déterminez si une structure de nom de compte cohérente rend l'application vulnérable à l'énumération des comptes.
- Déterminez si les messages d'erreur de l'application permettent l'énumération des comptes.

## Comment tester

- Déterminer la structure des noms de compte.
- Évaluer la réponse de l'application aux noms de compte valides et non valides.
- Utilisez des réponses différentes aux noms de compte valides et non valides pour énumérer les noms de compte valides.
- Utilisez des dictionnaires de noms de compte pour énumérer les noms de compte valides.

## Correction

Assurez-vous que l'application renvoie des messages d'erreur génériques cohérents en réponse à un nom de compte, un mot de passe ou d'autres informations d'identification d'utilisateur non valides saisis lors du processus de connexion.
