# Test de contournement du sch�ma d'authentification

|ID          |
|------------|
|WSTG-ATHN-04|

## Sommaire

En s�curit� informatique, l'authentification est le processus consistant � tenter de v�rifier l'identit� num�rique de l'exp�diteur d'une communication. Un exemple courant d'un tel processus est le processus de connexion. Tester le sch�ma d'authentification signifie comprendre le fonctionnement du processus d'authentification et utiliser ces informations pour contourner le m�canisme d'authentification.

Alors que la plupart des applications n�cessitent une authentification pour acc�der � des informations priv�es ou pour ex�cuter des t�ches, toutes les m�thodes d'authentification ne sont pas en mesure de fournir une s�curit� ad�quate. La n�gligence, l'ignorance ou la simple sous-estimation des menaces de s�curit� entra�nent souvent des sch�mas d'authentification qui peuvent �tre contourn�s en sautant simplement la page de connexion et en appelant directement une page interne qui est cens�e �tre accessible uniquement apr�s l'authentification.

De plus, il est souvent possible de contourner les mesures d'authentification en falsifiant les requ�tes et en faisant croire � l'application que l'utilisateur est d�j� authentifi�. Cela peut �tre accompli soit en modifiant le param�tre d'URL donn�, soit en manipulant le formulaire, soit en contrefaisant des sessions.

Les probl�mes li�s au sch�ma d'authentification peuvent se trouver � diff�rentes �tapes du cycle de vie du d�veloppement logiciel (SDLC), comme les phases de conception, de d�veloppement et de d�ploiement�:

- Dans la phase de conception, les erreurs peuvent inclure une mauvaise d�finition des sections d'application � prot�ger, le choix de ne pas appliquer de protocoles de cryptage forts pour s�curiser la transmission des informations d'identification, et bien d'autres.
- Dans la phase de d�veloppement, les erreurs peuvent inclure la mise en �uvre incorrecte de la fonctionnalit� de validation des entr�es ou le non-respect des meilleures pratiques de s�curit� pour le langage sp�cifique.
- Dans la phase de d�ploiement de l'application, il peut y avoir des probl�mes lors de la configuration de l'application (activit�s d'installation et de configuration) en raison d'un manque de comp�tences techniques requises ou d'un manque de bonne documentation.

## Objectifs des tests

- Assurez-vous que l'authentification est appliqu�e � tous les services qui en ont besoin.

## Comment tester

Il existe plusieurs m�thodes pour contourner le sch�ma d'authentification utilis� par une application Web�:

- Demande de page directe ([navigation forc�e](https://owasp.org/www-community/attacks/Forced_browsing))
- Modification des param�tres
- Pr�diction d'ID de session
- Injection SQL

### Demande de page directe

Si une application Web impl�mente le contr�le d'acc�s uniquement sur la page de connexion, le sch�ma d'authentification peut �tre contourn�. Par exemple, si un utilisateur demande directement une autre page via la navigation forc�e, cette page peut ne pas v�rifier les informations d'identification de l'utilisateur avant d'accorder l'acc�s. Essayez d'acc�der directement � une page prot�g�e via la barre d'adresse de votre navigateur pour tester cette m�thode.

![Demande directe � la page prot�g�e](images/Basm-directreq.jpg)\
*Figure 4.4.4-1 : Demande directe � la page prot�g�e*

###�Modification des param�tres

Un autre probl�me li� � la conception de l'authentification est lorsque l'application v�rifie une connexion r�ussie sur la base de param�tres � valeur fixe. Un utilisateur pourrait modifier ces param�tres pour acc�der aux zones prot�g�es sans fournir d'informations d'identification valides. Dans l'exemple ci-dessous, le param�tre ��authentifi頻 est remplac� par la valeur ��oui��, ce qui permet � l'utilisateur d'avoir acc�s. Dans cet exemple, le param�tre est dans l'URL, mais un proxy peut �galement �tre utilis� pour modifier le param�tre, en particulier lorsque les param�tres sont envoy�s sous forme d'�l�ments de formulaire dans une requ�te POST ou lorsque les param�tres sont stock�s dans un cookie.

```html
http://www.site.com/page.asp?authenticated=no

raven@blackbox /home $nc www.site.com 80
GET /page.asp?authenticated=yes HTTP/1.0

HTTP/1.1 200 OK
Date: Sat, 11 Nov 2006 10:22:44 GMT
Server: Apache
Connection: close
Content-Type: text/html; charset=iso-8859-1

<!DOCTYPE HTML PUBLIC "-//IETF//DTD HTML 2.0//EN">
<HTML><HEAD>
</HEAD><BODY>
<H1>You Are Authenticated</H1>
</BODY></HTML>
```

![Demande de modification de param�tre](images/Basm-parammod.jpg)\
*Figure 4.4.4-2 : Requ�te de modification de param�tre*

### Pr�diction d'ID de session

De nombreuses applications Web g�rent l'authentification � l'aide d'identificateurs de session (ID de session). Par cons�quent, si la g�n�ration d'ID de session est pr�visible, un utilisateur malveillant pourrait �tre en mesure de trouver un ID de session valide et d'obtenir un acc�s non autoris� � l'application, en se faisant passer pour un utilisateur pr�c�demment authentifi�.

Dans la figure suivante, les valeurs � l'int�rieur des cookies augmentent de mani�re lin�aire, il pourrait donc �tre facile pour un attaquant de deviner un ID de session valide.

![Valeurs des cookies au fil du temps](images/Basm-sessid.jpg)\
*Figure 4.4.4-3 : Valeurs des cookies au fil du temps*

Dans la figure suivante, les valeurs � l'int�rieur des cookies ne changent que partiellement, il est donc possible de restreindre une attaque par force brute aux champs d�finis indiqu�s ci-dessous.

![Valeurs des cookies partiellement modifi�es](images/Basm-sessid2.jpg)\
*Figure 4.4.4-4 : Valeurs de cookies partiellement modifi�es*

### Injection SQL (authentification par formulaire HTML)

L'injection SQL est une technique d'attaque largement connue. Cette section ne va pas d�crire cette technique en d�tail car il y a plusieurs sections dans ce guide qui expliquent les techniques d'injection au-del� de la port�e de cette section.

![Injection SQL](images/Basm-sqlinj.jpg)\
*Figure 4.4.4-5 : Injection SQL*

La figure suivante montre qu'avec une simple attaque par injection SQL, il est parfois possible de contourner le formulaire d'authentification.

![Attaque par injection SQL simple](images/Basm-sqlinj2.gif)\
*Figure 4.4.4-6 : Attaque par injection SQL simple*

### Comparaison simplifi�e de PHP

Si un attaquant a pu r�cup�rer le code source de l'application en exploitant une vuln�rabilit� pr�c�demment d�couverte (par exemple, la travers�e de r�pertoires) ou � partir d'un r�f�rentiel Web (applications Open Source), il pourrait �tre possible d'effectuer des attaques raffin�es contre la mise en �uvre de l'authentification. traiter.

Dans l'exemple suivant (PHPBB 2.0.12 - Vuln�rabilit� de contournement d'authentification), � la ligne 2, la fonction `unserialize()` analyse un cookie fourni par l'utilisateur et d�finit des valeurs dans le tableau `$sessiondata`. � la ligne 7, le hachage du mot de passe MD5 de l'utilisateur stock� dans la base de donn�es principale (`$auto_login_key`) est compar� � celui fourni (`$sessiondata['autologinid']`) par l'utilisateur.

```php
1. if (isset($HTTP_COOKIE_VARS[$cookiename . '_sid'])) {
2.     $sessiondata = isset($HTTP_COOKIE_VARS[$cookiename . '_data']) ? unserialize(stripslashes($HTTP_COOKIE_VARS[$cookiename . '_data'])) : array();
3.     $sessionmethod = SESSION_METHOD_COOKIE;
4. }
5. $auto_login_key = $userdata['user_password'];
6. // We have to login automagically
7. if( $sessiondata['autologinid'] == $auto_login_key )
8. {
9.     // autologinid matches password
10.     $login = 1;
11.     $enable_autologin = 1;
12. }

```

En PHP, une comparaison entre une valeur de cha�ne et une valeur bool�enne `true` est toujours `true` (car la cha�ne contient une valeur), donc en fournissant la cha�ne suivante � la fonction `unserialize()`, il est possible de contourner le contr�le d'authentification et connectez-vous en tant qu'administrateur, dont l'`userid` est 2�:

```php
a:2:{s:11:"autologinid";b:1;s:6:"userid";s:1:"2";}  // original value: a:2:{s:11:"autologinid";s:32:"8b8e9715d12e4ca12c4c3eb4865aaf6a";s:6:"userid";s:4:"1337";}
```

D�montons ce que nous avons fait dans cette cha�ne�:

1. `autologinid` est maintenant un bool�en d�fini sur `true`�: cela peut �tre vu en rempla�ant la valeur MD5 du hachage du mot de passe (`s:32:"8b8e9715d12e4ca12c4c3eb4865aaf6a"`) par `b:1`
2. `userid` est maintenant d�fini sur l'ID administrateur�: cela peut �tre vu dans le dernier morceau de la cha�ne, o� nous avons remplac� notre ID utilisateur habituel (`s:4:"1337"`) par `s:1:" 2"`

## Outils

- [WebGoat](https://owasp.org/www-project-webgoat/)
- [Proxy d'attaque Zed OWASP (ZAP)] (https://www.zaproxy.org)

## R�f�rences

- [Niels Teusink : contournement de l'authentification phpBB 2.0.12](http://blog.teusink.net/2008/12/classic-bug-phpbb-2012-authentication.html)
- [David Endler�: "Exploitation et pr�diction de la force brute de l'identifiant de session"](https://www.cgisecurity.com/lib/SessionIDs.pdf)
