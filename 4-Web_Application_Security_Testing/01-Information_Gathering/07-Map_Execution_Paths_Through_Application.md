# Mapper les chemins d'ex�cution via l'application

|ID          |
|------------|
|WSTG-INFO-07|

## Sommaire

Avant de commencer les tests de s�curit�, il est primordial de comprendre la structure de l'application. Sans une compr�hension approfondie de la disposition de l'application, il est peu probable qu'elle soit test�e de mani�re approfondie.

## Objectifs des tests

- Cartographier l'application cible et comprendre les principaux workflows.

## Comment tester

Dans les tests en bo�te noire, il est extr�mement difficile de tester l'int�gralit� de la base de code. Non seulement parce que le testeur n'a aucune vue sur les chemins de code � travers l'application, mais m�me s'il le faisait, tester tous les chemins de code prendrait beaucoup de temps. Une fa�on de concilier cela consiste � documenter les chemins de code d�couverts et test�s.

Il existe plusieurs fa�ons d'aborder le test et la mesure de la couverture de code�:

- **Chemin**�: testez chacun des chemins via une application qui inclut des tests d'analyse combinatoire et des valeurs limites pour chaque chemin de d�cision. Bien que cette approche offre de la rigueur, le nombre de chemins testables augmente de fa�on exponentielle avec chaque branche de d�cision.
- **Data Flow (or Taint Analysis)** - teste l'affectation de variables via une interaction externe (g�n�ralement des utilisateurs). Se concentre sur la cartographie du flux, de la transformation et de l'utilisation des donn�es dans une application.
- **Race** - teste plusieurs instances simultan�es de l'application manipulant les m�mes donn�es.

Le compromis quant � la m�thode utilis�e et � quel degr� chaque m�thode est utilis�e doit �tre n�goci� avec le propri�taire de l'application. Des approches plus simples pourraient �galement �tre adopt�es, notamment en demandant au propri�taire de l'application quelles fonctions ou sections de code le pr�occupent particuli�rement et comment ces segments de code peuvent �tre atteints.

Pour d�montrer la couverture du code au propri�taire de l'application, le testeur peut commencer avec une feuille de calcul et documenter tous les liens d�couverts en parcourant l'application (manuellement ou automatiquement). Ensuite, le testeur peut examiner de plus pr�s les points de d�cision dans l'application et d�terminer combien de chemins de code significatifs sont d�couverts. Ceux-ci doivent ensuite �tre document�s dans la feuille de calcul avec des URL, des descriptions en texte et en capture d'�cran des chemins d�couverts.

### Spidering automatique

L'araign�e automatique est un outil utilis� pour d�couvrir automatiquement de nouvelles ressources (URL) sur un site Web particulier. Cela commence par une liste d'URL � visiter, appel�es graines, qui d�pend de la fa�on dont l'araign�e est d�marr�e. Bien qu'il existe de nombreux outils de Spidering, l'exemple suivant utilise le [Zed Attack Proxy (ZAP)](https://github.com/zaproxy/zaproxy)�:

![�cran Proxy Zed Attack](images/OWASPZAPSP.png)\
*Figure 4.1.7-1 : �cran Proxy Zed Attack*

[ZAP](https://github.com/zaproxy/zaproxy) propose diverses options de spidering automatique, qui peuvent �tre exploit�es en fonction des besoins du testeur�:

- [Spider](https://www.zaproxy.org/docs/desktop/start/features/spider/)
- [Spider Ajax](https://www.zaproxy.org/docs/desktop/addons/ajax-spider/)
- [Prise en charge d'OpenAPI] (https://www.zaproxy.org/docs/desktop/addons/openapi-support/)

## Outils

- [Zed Attack Proxy (ZAP)] (https://github.com/zaproxy/zaproxy)
- [Liste des tableurs](https://en.wikipedia.org/wiki/List_of_spreadsheet_software)
- [Logiciel de cr�ation de diagrammes](https://en.wikipedia.org/wiki/List_of_concept-_and_mind-mapping_software)

## R�f�rences

- [Couverture de code](https://en.wikipedia.org/wiki/Code_coverage)
