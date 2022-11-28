# Test pour l'injection XPath

|ID          |
|------------|
|WSTG-INPV-09|

## Sommaire

XPath est un langage qui a �t� con�u et d�velopp� principalement pour traiter des parties d'un document XML. Dans les tests d'injection XPath, nous testons s'il est possible d'injecter la syntaxe XPath dans une requ�te interpr�t�e par l'application, permettant � un attaquant d'ex�cuter des requ�tes XPath contr�l�es par l'utilisateur. Lorsqu'elle est exploit�e avec succ�s, cette vuln�rabilit� peut permettre � un attaquant de contourner les m�canismes d'authentification ou d'acc�der � des informations sans autorisation appropri�e.

Les applications Web utilisent fortement les bases de donn�es pour stocker et acc�der aux donn�es dont elles ont besoin pour leurs op�rations. Historiquement, les bases de donn�es relationnelles ont �t� de loin la technologie la plus courante pour le stockage de donn�es, mais, ces derni�res ann�es, nous assistons � une popularit� croissante des bases de donn�es qui organisent les donn�es � l'aide du langage XML. Tout comme les bases de donn�es relationnelles sont accessibles via le langage SQL, les bases de donn�es XML utilisent XPath comme langage de requ�te standard.

�tant donn� que, d'un point de vue conceptuel, XPath est tr�s similaire � SQL dans son objectif et ses applications, un r�sultat int�ressant est que les attaques par injection XPath suivent la m�me logique que [SQL Injection](https://owasp.org/www-community /attacks/SQL_Injection). Sous certains aspects, XPath est encore plus puissant que le SQL standard, car toute sa puissance est d�j� pr�sente dans ses sp�cifications, alors qu'un grand nombre des techniques pouvant �tre utilis�es dans une attaque par injection SQL d�pendent des caract�ristiques du dialecte SQL utilis� par la base de donn�es cible. Cela signifie que les attaques par injection XPath peuvent �tre beaucoup plus adaptables et omnipr�sentes. Un autre avantage d'une attaque par injection XPath est que, contrairement � SQL, aucune ACL n'est appliqu�e, car notre requ�te peut acc�der � chaque partie du document XML.

## Objectifs des tests

- Identifier les points d'injection XPATH.

## Comment tester

Le [mod�le d'attaque XPath a �t� publi� pour la premi�re fois par Amit Klein](http://dl.packetstormsecurity.net/papers/bypass/Blind_XPath_Injection_20040518.pdf) et est tr�s similaire � l'injection SQL habituelle. Afin d'avoir une premi�re id�e du probl�me, imaginons une page de connexion qui g�re l'authentification � une application dans laquelle l'utilisateur doit entrer son nom d'utilisateur et son mot de passe. Supposons que notre base de donn�es est repr�sent�e par le fichier XML suivant�:

```xml
<?xml version="1.0" encoding="ISO-8859-1"?>
<users>
    <user>
        <username>gandalf</username>
        <password>!c3</password>
        <account>admin</account>
    </user>
    <user>
        <username>Stefan0</username>
        <password>w1s3c</password>
        <account>guest</account>
    </user>
    <user>
        <username>tony</username>
        <password>Un6R34kb!e</password>
        <account>guest</account>
    </user>
</users>
```

Une requ�te XPath qui renvoie le compte dont le nom d'utilisateur est `gandalf` et le mot de passe est `!c3` serait la suivante�:

`string(//user[username/text()='gandalf' and password/text()='!c3']/account/text())`

Si l'application ne filtre pas correctement les entr�es de l'utilisateur, le testeur pourra injecter du code XPath et interf�rer avec le r�sultat de la requ�te. Par exemple, le testeur peut saisir les valeurs suivantes�:

```text
Username: ' or '1' = '1
Password: ' or '1' = '1
```

Cela semble assez familier, n'est-ce pas? En utilisant ces param�tres, la requ�te devient�:

`string(//user[username/text()='' or '1' = '1' and password/text()='' or '1' = '1']/account/text())`

Comme dans une attaque par injection SQL courante, nous avons cr�� une requ�te qui est toujours �valu�e � vrai, ce qui signifie que l'application authentifiera l'utilisateur m�me si un nom d'utilisateur ou un mot de passe n'a pas �t� fourni. Et comme dans une attaque par injection SQL courante, avec l'injection XPath, la premi�re �tape consiste � ins�rer un guillemet simple (`'`) dans le champ � tester, introduisant une erreur de syntaxe dans la requ�te, et � v�rifier si l'application renvoie un Message d'erreur.

S'il n'y a aucune connaissance des d�tails internes des donn�es XML et si l'application ne fournit pas de messages d'erreur utiles qui nous aident � reconstruire sa logique interne, il est possible d'effectuer une [Blind XPath Injection](https://owasp.org/www -community/attacks/Blind_XPath_Injection) attaque, dont le but est de reconstruire toute la structure des donn�es. La technique est similaire � l'injection SQL bas�e sur l'inf�rence, car l'approche consiste � injecter du code qui cr�e une requ�te qui renvoie un bit d'information. [Blind XPath Injection](https://owasp.org/www-community/attacks/Blind_XPath_Injection) est expliqu� plus en d�tail par Amit Klein dans l'article r�f�renc�.

## R�f�rences

### Papiers blanc

- [Amit Klein�: "Blind XPath Injection"](http://dl.packetstormsecurity.net/papers/bypass/Blind_XPath_Injection_20040518.pdf)
- [Sp�cifications XPath 1.0](https://www.w3.org/TR/1999/REC-xpath-19991116/)
