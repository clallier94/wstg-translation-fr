# Test pour l'injection LDAP

|ID          |
|------------|
|WSTG-INPV-06|

## Sommaire

Le protocole LDAP (Lightweight Directory Access Protocol) est utilis� pour stocker des informations sur les utilisateurs, les h�tes et de nombreux autres objets. [Injection LDAP](https://wiki.owasp.org/index.php/LDAP_injection) est une attaque c�t� serveur, qui pourrait permettre la divulgation, la modification ou l'insertion d'informations sensibles sur les utilisateurs et les h�tes repr�sent�s dans une structure LDAP. . Cela se fait en manipulant les param�tres d'entr�e transmis ensuite aux fonctions internes de recherche, d'ajout et de modification.

Une application Web peut utiliser LDAP afin de permettre aux utilisateurs de s'authentifier ou de rechercher les informations d'autres utilisateurs au sein d'une structure d'entreprise. Le but des attaques par injection LDAP est d'injecter des m�tacaract�res de filtres de recherche LDAP dans une requ�te qui sera ex�cut�e par l'application.

[Rfc2254](https://www.ietf.org/rfc/rfc2254.txt) d�finit une grammaire sur la mani�re de cr�er un filtre de recherche sur LDAPv3 et �tend [Rfc1960](https://www.ietf.org/rfc/ rfc1960.txt) (LDAPv2).

Un filtre de recherche LDAP est construit en notation polonaise, �galement appel�e [notation de pr�fixe de notation polonaise](https://en.wikipedia.org/wiki/Polish_notation).

Cela signifie qu'une condition de pseudo-code sur un filtre de recherche comme celui-ci�:

`find("cn=John & userPassword=mypass")`

sera repr�sent� par :

`find("(&(cn=John)(userPassword=mypass))")`

Les conditions bool�ennes et les agr�gations de groupes sur un filtre de recherche LDAP peuvent �tre appliqu�es en utilisant les m�tacaract�res suivants�:

| M�tachar |  Sens                 |
|----------|-----------------------|
| &        |  Bool�en AND          |
| \|       |  Bool�en OR           |
| !        |  Bool�en NOT          |
| =        |  Egal                 |
| ~=       |  Environ              |
| >=       |  plus grand que       |
| <=       |  plus petit que       |
| *        |  N'importe quel caract�re |
| ()       |  Parenth�se de regroupement |

Des exemples plus complets sur la fa�on de cr�er un filtre de recherche peuvent �tre trouv�s dans la RFC associ�e.

Une exploitation r�ussie d'une vuln�rabilit� d'injection LDAP pourrait permettre au testeur de�:

- Acc�der � du contenu non autoris�
- �viter les restrictions d'application
- Recueillir des informations non autoris�es
- Ajouter ou modifier des objets dans l'arborescence LDAP

## Objectifs des tests

- Identifier les points d'injection LDAP.
- �valuer la s�v�rit� de l'injection.

## Comment tester

### Exemple�1�: Filtres de recherche

Supposons que nous ayons une application Web utilisant un filtre de recherche comme celui-ci�:

`searchfilter="(cn="+user+")"`

qui est instanci� par une requ�te HTTP comme celle-ci�:

`http://www.exemple.com/ldapsearch?user=John`

Si la valeur `John` est remplac�e par un `*`, en envoyant la requ�te�:

`http://www.exemple.com/ldapsearch?user=*`

le filtre ressemblera � :

`searchfilter="(cn=*)"`

qui correspond � chaque objet avec un attribut 'cn' �gal � n'importe quoi.

Si l'application est vuln�rable � l'injection LDAP, elle affichera tout ou partie des attributs de l'utilisateur, en fonction du flux d'ex�cution de l'application et des autorisations de l'utilisateur connect� LDAP.

Un testeur pourrait utiliser une approche par essais et erreurs, en ins�rant dans le param�tre `(`, `|`, `&`, `*` et les autres caract�res, afin de v�rifier l'application pour les erreurs.

### exemple 2 : Connexion

Si une application Web utilise LDAP pour v�rifier les informations d'identification de l'utilisateur pendant le processus de connexion et qu'elle est vuln�rable � l'injection LDAP, il est possible de contourner la v�rification d'authentification en injectant une requ�te LDAP toujours vraie (de la m�me mani�re que l'injection SQL et XPATH).

Supposons qu'une application Web utilise un filtre pour faire correspondre la paire utilisateur/mot de passe LDAP.

`searchlogin= "(&(uid="+user+")(userPassword={MD5}"+base64(pack("H*",md5(pass)))+"))";`

En utilisant les valeurs suivantes�:

```txt
user=*)(uid=*))(|(uid=*
pass=password
```

le filtre de recherche donnera :

`searchlogin="(&(uid=*)(uid=*))(|(uid=*)(userPassword={MD5}X03MO1qnZdYdgyfeuILPmQ==))";`

ce qui est correct et toujours vrai. De cette fa�on, le testeur obtiendra le statut de connexion en tant que premier utilisateur dans l'arborescence LDAP.

## Outils

- [Navigateur LDAP Softerra] (https://www.ldapadministrator.com)

## R�f�rences

- [Aide-m�moire sur la pr�vention des injections LDAP](https://cheatsheetseries.owasp.org/cheatsheets/LDAP_Injection_Prevention_Cheat_Sheet.html)

### Papiers blanc

- [Sacha Faust : Injection LDAP : vos applications sont-elles vuln�rables ?](http://www.networkdls.com/articles/ldapinjection.pdf)
- [Article IBM�: Comprendre LDAP] (https://www.redbooks.ibm.com/redbooks/pdfs/sg244986.pdf)
- [RFC 1960�: une repr�sentation sous forme de cha�ne des filtres de recherche LDAP] (https://www.ietf.org/rfc/rfc1960.txt)
- [Injection LDAP](https://www.blackhat.com/presentations/bh-europe-08/Alonso-Parada/Whitepaper/bh-eu-08-alonso-parada-WP.pdf)
