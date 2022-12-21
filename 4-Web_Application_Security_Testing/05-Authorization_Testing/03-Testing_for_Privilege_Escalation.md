# Test d'élévation de privilèges

|ID          |
|------------|
|WSTG-ATHZ-03|

## Sommaire

Cette section décrit le problème de l'escalade des privilèges d'une étape à l'autre. Au cours de cette phase, le testeur doit vérifier qu'il n'est pas possible pour un utilisateur de modifier ses privilèges ou ses rôles dans l'application d'une manière qui pourrait permettre des attaques par escalade de privilèges.

L'élévation des privilèges se produit lorsqu'un utilisateur a accès à plus de ressources ou de fonctionnalités qu'il n'est normalement autorisé, et une telle élévation ou modification aurait dû être empêchée par l'application. Ceci est généralement causé par une faille dans l'application. Le résultat est que l'application effectue des actions avec plus de privilèges que ceux prévus par le développeur ou l'administrateur système.

Le degré d'escalade dépend des privilèges que l'attaquant est autorisé à posséder et des privilèges qui peuvent être obtenus lors d'un exploit réussi. Par exemple, une erreur de programmation qui permet à un utilisateur d'obtenir des privilèges supplémentaires après une authentification réussie limite le degré d'escalade, car l'utilisateur est déjà autorisé à détenir certains privilèges. De même, un attaquant distant obtenant le privilège de superutilisateur sans aucune authentification présente un plus grand degré d'escalade.

Habituellement, les gens se réfèrent à *escalade verticale* lorsqu'il est possible d'accéder à des ressources accordées à des comptes plus privilégiés (par exemple, acquérir des privilèges administratifs pour l'application), et à *escalade horizontale* lorsqu'il est possible d'accéder à des ressources accordées à un utilisateur configuré de manière similaire. compte (par exemple, dans une application bancaire en ligne, accéder à des informations relatives à un autre utilisateur).

## Objectifs des tests

- Identifier les points d'injection liés à la manipulation des privilèges.
- Fuzz ou tentative de contournement des mesures de sécurité.

## Comment tester

### Test de la manipulation des rôles/privilèges

Dans chaque partie de l'application où un utilisateur peut créer des informations dans la base de données (par exemple, effectuer un paiement, ajouter un contact ou envoyer un message), peut recevoir des informations (relevé de compte, détails de la commande, etc.) ou supprimer des informations (déposer des utilisateurs, des messages, etc.), il est nécessaire d'enregistrer cette fonctionnalité. Le testeur doit essayer d'accéder à ces fonctions en tant qu'un autre utilisateur afin de vérifier s'il est possible d'accéder à une fonction qui ne devrait pas être autorisée par le rôle/privilège de l'utilisateur (mais qui pourrait être autorisée en tant qu'un autre utilisateur).

#### Manipulation du groupe d'utilisateurs

Par exemple:
Le HTTP POST suivant permet à l'utilisateur qui appartient à `grp001` d'accéder à la commande n° 0001 :

```http
POST /user/viewOrder.jsp HTTP/1.1
Host: www.exemple.com
...

groupID=grp001&orderID=0001
```

Vérifiez si un utilisateur qui n'appartient pas à `grp001` peut modifier la valeur des paramètres `groupID` et `orderID` pour accéder à ces données privilégiées.

#### Manipulation du profil utilisateur

Par exemple:
La réponse suivante du serveur montre un champ masqué dans le code HTML renvoyé à l'utilisateur après une authentification réussie.

```html
HTTP/1.1 200 OK
Server: Netscape-Enterprise/6.0
Date: Wed, 1 Apr 2006 13:51:20 GMT
Set-Cookie: USER=aW78ryrGrTWs4MnOd32Fs51yDqp; path=/; domain=www.example.com
Set-Cookie: SESSION=k+KmKeHXTgDi1J5fT7Zz; path=/; domain= www.example.com
Cache-Control: no-cache
Pragma: No-cache
Content-length: 247
Content-Type: text/html
Expires: Thu, 01 Jan 1970 00:00:00 GMT
Connection: close

<form  name="autoriz" method="POST" action = "visual.jsp">
<input type="hidden" name="profile" value="SysAdmin">\

<body onload="document.forms.autoriz.submit()">
</td>
</tr>
```

Que se passe-t-il si le testeur modifie la valeur de la variable `profile` en `SysAdmin` ? Est-il possible de devenir **administrateur** ?

#### Manipulation de la valeur de condition

Par exemple:
Dans un environnement où le serveur envoie un message d'erreur contenu sous la forme d'une valeur dans un paramètre spécifique d'un ensemble de codes de réponse, comme suit :

```text
@0`1`3`3``0`UC`1`Status`OK`SEC`5`1`0`ResultSet`0`PVValid`-1`0`0` Notifications`0`0`3`Command  Manager`0`0`0` StateToolsBar`0`0`0`
StateExecToolBar`0`0`0`FlagsToolBar`0
```

Le serveur donne une confiance implicite à l'utilisateur. Il pense que l'utilisateur répondra avec le message ci-dessus fermant la session.

Dans cette condition, vérifiez qu'il n'est pas possible d'élever les privilèges en modifiant les valeurs des paramètres. Dans cet exemple particulier, en modifiant la valeur `PVValid` de `-1` à `0` (aucune condition d'erreur), il peut être possible de s'authentifier en tant qu'administrateur auprès du serveur.

#### Manipulation de l'adresse IP

Certains sites Web limitent l'accès ou comptent le nombre de tentatives de connexion infructueuses en fonction de l'adresse IP.

Par exemple:

```text
X-Forwarded-For: 8.1.1.1
```

Dans ce cas, si le site Web utilise la valeur de `X-forwarded-For` comme adresse IP client, le testeur peut modifier la valeur IP de l'en-tête HTTP `X-forwarded-For` pour contourner l'identification de la source IP.

### Test du schéma d'autorisation de contournement vertical

Un contournement d'autorisation vertical est spécifique au cas où un attaquant obtient un rôle supérieur au sien. Le test de ce contournement se concentre sur la vérification de la manière dont le schéma d'autorisation vertical a été implémenté pour chaque rôle. Pour chaque fonction, page, rôle spécifique ou demande que l'application exécute, il est nécessaire de vérifier s'il est possible de :

- Accéder aux ressources qui ne devraient être accessibles qu'à un utilisateur de rôle supérieur.
- Exploiter des fonctions sur des ressources qui ne devraient être opérationnelles que par un utilisateur qui détient une identité de rôle supérieure ou spécifique.

Pour chaque rôle :

1. Enregistrez un utilisateur.
2. Établir et maintenir deux sessions différentes basées sur les deux rôles différents.
3. Pour chaque demande, remplacez l'identifiant de session de l'original par l'identifiant de session d'un autre rôle et évaluez les réponses pour chacune.
4. Une application sera considérée comme vulnérable si la session privilégiée la plus faible contient les mêmes données ou indique des opérations réussies sur des fonctions privilégiées plus élevées.

#### Scénario des rôles sur le site bancaire

Le tableau suivant illustre les rôles système sur un site bancaire. Chaque rôle est lié à des autorisations spécifiques pour la fonctionnalité du menu des événements :

|      ROLE      |        AUTORISATION     | AUTORISATION SUPPLÉMENTAIRE |
|----------------|-------------------------|-----------------------------|
| Administrateur | Contrôle total          | Supprimer                   |
| Gérant         | Modifier, Ajouter, Lire | Ajouter                     |
| Personnel      | Lire, Modifier          | Modifier                    |
| Client         | Lecture seule           |                             |

L'application sera considérée comme vulnérable si :

1. Le client peut exploiter des fonctions d'administrateur, de gestionnaire ou de personnel ;
2. L'utilisateur du personnel peut exploiter les fonctions de gestionnaire ou d'administrateur ;
3. Le gestionnaire peut exploiter les fonctions d'administrateur.

Supposons que la fonction `deleteEvent` fasse partie du menu du compte administrateur de l'application, et qu'il soit possible d'y accéder en demandant l'URL suivante : `https://www.exemple.com/account/deleteEvent`. Ensuite, la requête HTTP suivante est générée lors de l'appel de la fonction `deleteEvent` :

```http
POST /account/deleteEvent HTTP/1.1
Host: www.example.com
[other HTTP headers]
Cookie: SessionID=ADMINISTRATOR_USER_SESSION

EventID=1000001
```

La réponse valide :

```http
HTTP/1.1 200 OK
[other HTTP headers]

{"message": "Event was deleted"}
```

L'attaquant peut tenter d'exécuter la même requête :

```http
POST /account/deleteEvent HTTP/1.1
Host: www.exemple.com
[other HTTP headers]
Cookie: SessionID=CUSTOMER_USER_SESSION

EventID=1000002
```

Si la réponse de la requête de l'attaquant contient les mêmes données `{"message": "L'événement a été supprimé"}` l'application est vulnérable.

#### Accès à la page administrateur

Supposons que le menu administrateur fasse partie du compte administrateur.

L'application sera considérée comme vulnérable si un rôle autre que l'administrateur peut accéder au menu administrateur. Parfois, les développeurs effectuent une validation d'autorisation uniquement au niveau de l'interface graphique et laissent les fonctions sans validation d'autorisation, ce qui peut entraîner une vulnérabilité.

### Parcours d'URL

Essayez de parcourir le site Web et vérifiez si certaines des pages peuvent manquer la vérification d'autorisation.

Par exemple :

```text
/../.././userInfo.html
```

### Boîte blanche

Si la vérification de l'autorisation d'URL n'est effectuée que par correspondance partielle d'URL, il est probable que des testeurs ou des pirates puissent contourner l'autorisation par des techniques de codage d'URL.

Par exemple :

```text
startswith(), endswith(), contains(), indexOf()
```

### ID de session faible

L'ID de session faible a un algorithme peut être vulnérable à une attaque par force brute. Par exemple, un site Web utilise `MD5 (Mot de passe + ID utilisateur)` comme ID de session. Ensuite, les testeurs peuvent deviner ou générer le sessionID pour les autres utilisateurs.

## Références

### Papiers blanc

- [Wikipédia - Augmentation des privilèges](https://en.wikipedia.org/wiki/Privilege_escalation)

## Outils

- [OWASP Zed Attack Proxy (ZAP)](https://www.zaproxy.org)
