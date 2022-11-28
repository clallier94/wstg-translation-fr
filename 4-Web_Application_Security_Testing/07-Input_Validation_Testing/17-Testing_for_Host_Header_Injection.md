# Test de l'injection d'en-t�te d'h�te

|ID          |
|------------|
|WSTG-INPV-17|

## Sommaire

Un serveur Web h�berge g�n�ralement plusieurs applications Web sur la m�me adresse IP, se r�f�rant � chaque application via l'h�te virtuel. Dans une requ�te HTTP entrante, les serveurs Web envoient souvent la requ�te � l'h�te virtuel cible en fonction de la valeur fournie dans l'en-t�te Host. Sans une validation correcte de la valeur de l'en-t�te, l'attaquant peut fournir une entr�e non valide pour amener le serveur Web �:

- Envoi des demandes au premier h�te virtuel de la liste.
- Effectuer une redirection vers un domaine contr�l� par un attaquant.
- Effectuer un empoisonnement du cache Web.
- Manipuler la fonctionnalit� de r�initialisation du mot de passe.
- Autoriser l'acc�s aux h�tes virtuels qui n'�taient pas destin�s � �tre accessibles de l'ext�rieur.

## Objectifs des tests

- �valuer si l'en-t�te Host est analys� dynamiquement dans l'application.
- Contourner les contr�les de s�curit� qui reposent sur l'en-t�te.

## Comment tester

Le test initial est aussi simple que de fournir un autre domaine (c'est-�-dire "attaquant.com") dans le champ d'en-t�te Host. C'est la fa�on dont le serveur Web traite la valeur d'en-t�te qui dicte l'impact. L'attaque est valide lorsque le serveur Web traite l'entr�e pour envoyer la demande � un h�te contr�l� par l'attaquant qui r�side dans le domaine fourni, et non � un h�te virtuel interne qui r�side sur le serveur Web.

```http
GET / HTTP/1.1
Host: www.attacker.com
[...]
```

Dans le cas le plus simple, cela peut provoquer une redirection 302 vers le domaine fourni.

```http
HTTP/1.1 302 Found
[...]
Location: http://www.attacker.com/login.php

```

Alternativement, le serveur Web peut envoyer la demande au premier h�te virtuel de la liste.

### Contournement d'en-t�te d'h�te transmis par X

Dans le cas o� l'injection d'en-t�te Host est att�nu�e en v�rifiant les entr�es non valides inject�es via l'en-t�te Host, vous pouvez fournir la valeur � l'en-t�te `X-Forwarded-Host`.

```http
GET / HTTP/1.1
Host: www.exemple.com
X-Forwarded-Host: www.attacker.com
[...]
```

Produisant potentiellement une sortie c�t� client telle que�:

```html
[...]
<link src="http://www.attacker.com/link" />
[...]
```

Encore une fois, cela d�pend de la fa�on dont le serveur Web traite la valeur d'en-t�te.

### Empoisonnement du cache Web

En utilisant cette technique, un attaquant peut manipuler un cache Web pour fournir un contenu empoisonn� � toute personne qui en fait la demande. Cela repose sur la capacit� d'empoisonner le proxy de mise en cache ex�cut� par l'application elle-m�me, les CDN ou d'autres fournisseurs en aval. Par cons�quent, la victime n'aura aucun contr�le sur la r�ception du contenu malveillant lors de la demande de l'application vuln�rable.

```http
GET / HTTP/1.1
Host: www.attacker.com
[...]
```

Les �l�ments suivants seront servis � partir du cache Web, lorsqu'une victime visite l'application vuln�rable.

```html
[...]
<link src="http://www.attacker.com/link" />
[...]
```

### Empoisonnement de r�initialisation de mot de passe

Il est courant que la fonctionnalit� de r�initialisation de mot de passe inclue la valeur d'en-t�te Host lors de la cr�ation de liens de r�initialisation de mot de passe qui utilisent un jeton secret g�n�r�. Si l'application traite un domaine contr�l� par l'attaquant pour cr�er un lien de r�initialisation du mot de passe, la victime peut cliquer sur le lien dans l'e-mail et permettre � l'attaquant d'obtenir le jeton de r�initialisation, r�initialisant ainsi le mot de passe de la victime.

L'exemple ci-dessous montre un lien de r�initialisation de mot de passe g�n�r� en PHP � l'aide de la valeur de `$_SERVER['HTTP_HOST']`, qui est d�finie en fonction du contenu de l'en-t�te HTTP Host�:

```php
$reset_url = "https://" . $_SERVER['HTTP_HOST'] . "/reset.php?token=" .$token;
send_reset_email($email,$rset_url);
```

En faisant une requ�te HTTP � la page de r�initialisation du mot de passe avec un en-t�te Host falsifi�, nous pouvons modifier l'endroit o� pointe l'URL�:

```http
POST /request_password_reset.php HTTP/1.1
Host: www.attacker.com
[...]

email=user@exemple.org
```

Le domaine sp�cifi� ("www.attacker.com") sera alors utilis� dans le lien de r�initialisation, qui est envoy� par e-mail � l'utilisateur. Lorsque l'utilisateur clique sur ce lien, l'attaquant peut voler le jeton et compromettre son compte.

```text
... Email snippet ...

Click on the following link to reset your password:

https://www.attacker.com/reset.php?token=12345

... Email snippet ...
```

### Acc�s aux h�tes virtuels priv�s

Dans certains cas, un serveur peut avoir des h�tes virtuels qui ne sont pas destin�s � �tre accessibles de l'ext�rieur. Ceci est plus courant avec une configuration DNS [split-horizon](https://en.wikipedia.org/wiki/Split-horizon_DNS) (o� les serveurs DNS internes et externes renvoient des enregistrements diff�rents pour le m�me domaine).

Par exemple, une organisation peut avoir un seul serveur Web sur son r�seau interne, qui h�berge � la fois son site Web public (sur `www.exemple.org`) et son intranet interne (sur `intranet.exemple.org`, mais cet enregistrement n'existe que sur le serveur DNS interne). Bien qu'il ne soit pas possible de naviguer directement vers `intranet.exemple.org` depuis l'ext�rieur du r�seau (car le domaine ne serait pas r�solu), il peut �tre possible d'acc�der � l'intranet en faisant une demande depuis l'ext�rieur avec l'`H�te` suivant ent�te:

``` http
H�bergeur : intranet.exemple.org
```

Cela peut �galement �tre r�alis� en ajoutant une entr�e pour `intranet.exemple.org` � votre fichier hosts avec l'adresse IP publique de `www.exemple.org`, ou en rempla�ant la r�solution DNS dans votre outil de test.

## R�f�rences

- [Qu'est-ce qu'une attaque d'en-t�te d'h�te�?] (https://www.acunetix.com/blog/articles/automated-detection-of-host-header-attacks/)
- [Host Header Attack](https://www.briskinfosec.com/blogs/blogsdetail/Host-Header-Attack)
- [Attaques d'en-t�te d'h�te HTTP] (https://portswigger.net/web-security/host-header)
