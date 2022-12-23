# Test de l'injection d'en-tête d'hôte

|ID          |
|------------|
|WSTG-INPV-17|

## Sommaire

Un serveur Web héberge généralement plusieurs applications Web sur la même adresse IP, se référant à chaque application via l'hôte virtuel. Dans une requête HTTP entrante, les serveurs Web envoient souvent la requête à l'hôte virtuel cible en fonction de la valeur fournie dans l'en-tête Host. Sans une validation correcte de la valeur de l'en-tête, l'attaquant peut fournir une entrée non valide pour amener le serveur Web à :

- Envoi des demandes au premier hôte virtuel de la liste.
- Effectuer une redirection vers un domaine contrôlé par un attaquant.
- Effectuer un empoisonnement du cache Web.
- Manipuler la fonctionnalité de réinitialisation du mot de passe.
- Autoriser l'accès aux hôtes virtuels qui n'étaient pas destinés à être accessibles de l'extérieur.

## Objectifs des tests

- Évaluer si l'en-tête Host est analysé dynamiquement dans l'application.
- Contourner les contrôles de sécurité qui reposent sur l'en-tête.

## Comment tester

Le test initial est aussi simple que de fournir un autre domaine (c'est-à-dire "attaquant.com") dans le champ d'en-tête Host. C'est la façon dont le serveur Web traite la valeur d'en-tête qui dicte l'impact. L'attaque est valide lorsque le serveur Web traite l'entrée pour envoyer la demande à un hôte contrôlé par l'attaquant qui réside dans le domaine fourni, et non à un hôte virtuel interne qui réside sur le serveur Web.

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

Alternativement, le serveur Web peut envoyer la demande au premier hôte virtuel de la liste.

### Contournement d'en-tête d'hôte transmis par X

Dans le cas où l'injection d'en-tête Host est atténuée en vérifiant les entrées non valides injectées via l'en-tête Host, vous pouvez fournir la valeur à l'en-tête `X-Forwarded-Host`.

```http
GET / HTTP/1.1
Host: www.exemple.com
X-Forwarded-Host: www.attacker.com
[...]
```

Produisant potentiellement une sortie côté client telle que :

```html
[...]
<link src="http://www.attacker.com/link" />
[...]
```

Encore une fois, cela dépend de la façon dont le serveur Web traite la valeur d'en-tête.

### Empoisonnement du cache Web

En utilisant cette technique, un attaquant peut manipuler un cache Web pour fournir un contenu empoisonné à toute personne qui en fait la demande. Cela repose sur la capacité d'empoisonner le proxy de mise en cache exécuté par l'application elle-même, les CDN ou d'autres fournisseurs en aval. Par conséquent, la victime n'aura aucun contrôle sur la réception du contenu malveillant lors de la demande de l'application vulnérable.

```http
GET / HTTP/1.1
Host: www.attacker.com
[...]
```

Les éléments suivants seront servis à partir du cache Web, lorsqu'une victime visite l'application vulnérable.

```html
[...]
<link src="http://www.attacker.com/link" />
[...]
```

### Empoisonnement de réinitialisation de mot de passe

Il est courant que la fonctionnalité de réinitialisation de mot de passe inclue la valeur d'en-tête Host lors de la création de liens de réinitialisation de mot de passe qui utilisent un jeton secret généré. Si l'application traite un domaine contrôlé par l'attaquant pour créer un lien de réinitialisation du mot de passe, la victime peut cliquer sur le lien dans l'e-mail et permettre à l'attaquant d'obtenir le jeton de réinitialisation, réinitialisant ainsi le mot de passe de la victime.

L'exemple ci-dessous montre un lien de réinitialisation de mot de passe généré en PHP à l'aide de la valeur de `$_SERVER['HTTP_HOST']`, qui est définie en fonction du contenu de l'en-tête HTTP Host :

```php
$reset_url = "https://" . $_SERVER['HTTP_HOST'] . "/reset.php?token=" .$token;
send_reset_email($email,$rset_url);
```

En faisant une requête HTTP à la page de réinitialisation du mot de passe avec un en-tête Host falsifié, nous pouvons modifier l'endroit où pointe l'URL :

```http
POST /request_password_reset.php HTTP/1.1
Host: www.attacker.com
[...]

email=user@exemple.org
```

Le domaine spécifié (`www.attacker.com`) sera alors utilisé dans le lien de réinitialisation, qui est envoyé par e-mail à l'utilisateur. Lorsque l'utilisateur clique sur ce lien, l'attaquant peut voler le jeton et compromettre son compte.

```text
... Email snippet ...

Click on the following link to reset your password:

https://www.attacker.com/reset.php?token=12345

... Email snippet ...
```

### Accès aux hôtes virtuels privés

Dans certains cas, un serveur peut avoir des hôtes virtuels qui ne sont pas destinés à être accessibles de l'extérieur. Ceci est plus courant avec une configuration DNS [split-horizon](https://en.wikipedia.org/wiki/Split-horizon_DNS) (où les serveurs DNS internes et externes renvoient des enregistrements différents pour le même domaine).

Par exemple, une organisation peut avoir un seul serveur Web sur son réseau interne, qui héberge à la fois son site Web public (sur `www.exemple.org`) et son intranet interne (sur `intranet.exemple.org`, mais cet enregistrement n'existe que sur le serveur DNS interne). Bien qu'il ne soit pas possible de naviguer directement vers `intranet.exemple.org` depuis l'extérieur du réseau (car le domaine ne serait pas résolu), il peut être possible d'accéder à l'intranet en faisant une demande depuis l'extérieur avec l'`Hôte` suivant entête:

``` http
Hébergeur : intranet.exemple.org
```

Cela peut également être réalisé en ajoutant une entrée pour `intranet.exemple.org` à votre fichier hosts avec l'adresse IP publique de `www.exemple.org`, ou en remplaçant la résolution DNS dans votre outil de test.

## Références

- [Qu'est-ce qu'une attaque d'en-tête d'hôte ?](https://www.acunetix.com/blog/articles/automated-detection-of-host-header-attacks/)
- [Host Header Attack](https://www.briskinfosec.com/blogs/blogsdetail/Host-Header-Attack)
- [Attaques d'en-tête d'hôte HTTP](https://portswigger.net/web-security/host-header)
