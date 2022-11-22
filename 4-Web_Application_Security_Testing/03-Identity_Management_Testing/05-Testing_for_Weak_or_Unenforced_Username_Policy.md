# Test de la politique de nom d'utilisateur faible ou non appliqu�e

|ID          |
|------------|
|WSTG-IDNT-05|

## Sommaire

Les noms de compte utilisateur sont souvent tr�s structur�s (par exemple, le nom du compte Joe Bloggs est jbloggs et le nom du compte Fred Nurks est fnurks) et les noms de compte valides peuvent facilement �tre devin�s.

## Objectifs des tests

- D�terminez si une structure de nom de compte coh�rente rend l'application vuln�rable � l'�num�ration des comptes.
- D�terminez si les messages d'erreur de l'application permettent l'�num�ration des comptes.

## Comment tester

- D�terminer la structure des noms de compte.
- �valuer la r�ponse de l'application aux noms de compte valides et non valides.
- Utilisez des r�ponses diff�rentes aux noms de compte valides et non valides pour �num�rer les noms de compte valides.
- Utilisez des dictionnaires de noms de compte pour �num�rer les noms de compte valides.

## Correction

Assurez-vous que l'application renvoie des messages d'erreur g�n�riques coh�rents en r�ponse � un nom de compte, un mot de passe ou d'autres informations d'identification d'utilisateur non valides saisis lors du processus de connexion.
