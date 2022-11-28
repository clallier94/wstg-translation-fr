# Test pour l'injection LDAP

|ID          |
|------------|
|WSTG-INPV-06|

## Sommaire

Le protocole LDAP (Lightweight Directory Access Protocol) est utilisé pour stocker des informations sur les utilisateurs, les hôtes et de nombreux autres objets. [Injection LDAP](https://wiki.owasp.org/index.php/LDAP_injection) est une attaque côté serveur, qui pourrait permettre la divulgation, la modification ou l'insertion d'informations sensibles sur les utilisateurs et les hôtes représentés dans une structure LDAP. . Cela se fait en manipulant les paramètres d'entrée transmis ensuite aux fonctions internes de recherche, d'ajout et de modification.

Une application Web peut utiliser LDAP afin de permettre aux utilisateurs de s'authentifier ou de rechercher les informations d'autres utilisateurs au sein d'une structure d'entreprise. Le but des attaques par injection LDAP est d'injecter des métacaractères de filtres de recherche LDAP dans une requête qui sera exécutée par l'application.

[Rfc2254](https://www.ietf.org/rfc/rfc2254.txt) définit une grammaire sur la manière de créer un filtre de recherche sur LDAPv3 et étend [Rfc1960](https://www.ietf.org/rfc/ rfc1960.txt) (LDAPv2).

Un filtre de recherche LDAP est construit en notation polonaise, également appelée [notation de préfixe de notation polonaise](https://en.wikipedia.org/wiki/Polish_notation).

Cela signifie qu'une condition de pseudo-code sur un filtre de recherche comme celui-ci :

`find("cn=John & userPassword=mypass")`

sera représenté par :

`find("(&(cn=John)(userPassword=mypass))")`

Les conditions booléennes et les agrégations de groupes sur un filtre de recherche LDAP peuvent être appliquées en utilisant les métacaractères suivants :

| Métachar |  Sens                 |
|----------|-----------------------|
| &        |  Booléen AND          |
| \|       |  Booléen OR           |
| !        |  Booléen NOT          |
| =        |  Egal                 |
| ~=       |  Environ              |
| >=       |  plus grand que       |
| <=       |  plus petit que       |
| *        |  N'importe quel caractère |
| ()       |  Parenthèse de regroupement |

Des exemples plus complets sur la façon de créer un filtre de recherche peuvent être trouvés dans la RFC associée.

Une exploitation réussie d'une vulnérabilité d'injection LDAP pourrait permettre au testeur de :

- Accéder à du contenu non autorisé
- Éviter les restrictions d'application
- Recueillir des informations non autorisées
- Ajouter ou modifier des objets dans l'arborescence LDAP

## Objectifs des tests

- Identifier les points d'injection LDAP.
- Évaluer la sévérité de l'injection.

## Comment tester

### Exemple 1 : Filtres de recherche

Supposons que nous ayons une application Web utilisant un filtre de recherche comme celui-ci :

`searchfilter="(cn="+user+")"`

qui est instancié par une requête HTTP comme celle-ci :

`http://www.exemple.com/ldapsearch?user=John`

Si la valeur `John` est remplacée par un `*`, en envoyant la requête :

`http://www.exemple.com/ldapsearch?user=*`

le filtre ressemblera à :

`searchfilter="(cn=*)"`

qui correspond à chaque objet avec un attribut 'cn' égal à n'importe quoi.

Si l'application est vulnérable à l'injection LDAP, elle affichera tout ou partie des attributs de l'utilisateur, en fonction du flux d'exécution de l'application et des autorisations de l'utilisateur connecté LDAP.

Un testeur pourrait utiliser une approche par essais et erreurs, en insérant dans le paramètre `(`, `|`, `&`, `*` et les autres caractères, afin de vérifier l'application pour les erreurs.

### exemple 2 : Connexion

Si une application Web utilise LDAP pour vérifier les informations d'identification de l'utilisateur pendant le processus de connexion et qu'elle est vulnérable à l'injection LDAP, il est possible de contourner la vérification d'authentification en injectant une requête LDAP toujours vraie (de la même manière que l'injection SQL et XPATH).

Supposons qu'une application Web utilise un filtre pour faire correspondre la paire utilisateur/mot de passe LDAP.

`searchlogin= "(&(uid="+user+")(userPassword={MD5}"+base64(pack("H*",md5(pass)))+"))";`

En utilisant les valeurs suivantes :

```txt
user=*)(uid=*))(|(uid=*
pass=password
```

le filtre de recherche donnera :

`searchlogin="(&(uid=*)(uid=*))(|(uid=*)(userPassword={MD5}X03MO1qnZdYdgyfeuILPmQ==))";`

ce qui est correct et toujours vrai. De cette façon, le testeur obtiendra le statut de connexion en tant que premier utilisateur dans l'arborescence LDAP.

## Outils

- [Navigateur LDAP Softerra] (https://www.ldapadministrator.com)

## Références

- [Aide-mémoire sur la prévention des injections LDAP](https://cheatsheetseries.owasp.org/cheatsheets/LDAP_Injection_Prevention_Cheat_Sheet.html)

### Papiers blanc

- [Sacha Faust : Injection LDAP : vos applications sont-elles vulnérables ?](http://www.networkdls.com/articles/ldapinjection.pdf)
- [Article IBM : Comprendre LDAP] (https://www.redbooks.ibm.com/redbooks/pdfs/sg244986.pdf)
- [RFC 1960 : une représentation sous forme de chaîne des filtres de recherche LDAP] (https://www.ietf.org/rfc/rfc1960.txt)
- [Injection LDAP](https://www.blackhat.com/presentations/bh-europe-08/Alonso-Parada/Whitepaper/bh-eu-08-alonso-parada-WP.pdf)
