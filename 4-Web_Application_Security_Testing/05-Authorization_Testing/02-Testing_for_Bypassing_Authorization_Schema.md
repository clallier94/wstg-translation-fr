# Test du schéma d'autorisation de contournement

|ID          |
|------------|
|WSTG-ATHZ-02|

## Sommaire

Ce type de test se concentre sur la vérification de la manière dont le schéma d'autorisation a été implémenté pour chaque rôle ou privilège afin d'accéder aux fonctions et ressources réservées.

Pour chaque rôle spécifique tenu par le testeur lors de l'évaluation et pour chaque fonction et demande que l'application exécute lors de la phase de post-authentification, il est nécessaire de vérifier :

- Est-il possible d'accéder à cette ressource même si l'utilisateur n'est pas authentifié ?
- Est-il possible d'accéder à cette ressource après la déconnexion ?
- Est-il possible d'accéder à des fonctions et des ressources qui devraient être accessibles à un utilisateur qui détient un rôle ou un privilège différent ?

Essayez d'accéder à l'application en tant qu'utilisateur administratif et suivez toutes les fonctions administratives.

- Est-il possible d'accéder aux fonctions d'administration si le testeur est connecté en tant qu'utilisateur non administrateur ?
- Est-il possible d'utiliser ces fonctions d'administration en tant qu'utilisateur avec un rôle différent et pour qui cette action doit être refusée ?

## Objectifs des tests

- Évaluer si un accès horizontal ou vertical est possible.

## Comment tester

- Accéder aux ressources et mener les opérations horizontalement.
- Accéder aux ressources et mener les opérations verticalement.

### Test du schéma d'autorisation de contournement horizontal

Pour chaque fonction, rôle spécifique ou demande que l'application exécute, il est nécessaire de vérifier :

- Est-il possible d'accéder à des ressources qui devraient être accessibles à un utilisateur qui détient une identité différente avec le même rôle ou privilège ?
- Est-il possible d'opérer des fonctions sur des ressources qui devraient être accessibles à un utilisateur qui détient une identité différente ?

Pour chaque rôle :

1. Enregistrez ou générez deux utilisateurs avec des privilèges identiques.
2. Établissez et maintenez actives deux sessions différentes (une pour chaque utilisateur).
3. Pour chaque demande, modifiez les paramètres pertinents et l'identifiant de session du jeton un au jeton deux et diagnostiquez les réponses pour chaque jeton.
4. Une application sera considérée comme vulnérable si les réponses sont identiques, contiennent les mêmes données privées ou indiquent une opération réussie sur les ressources ou les données d'autres utilisateurs.

Par exemple, supposons que la fonction `viewSettings` fasse partie de chaque menu de compte de l'application avec le même rôle, et qu'il soit possible d'y accéder en demandant l'URL suivante : `https://www.exemple.com/account/viewSettings`. Ensuite, la requête HTTP suivante est générée lors de l'appel de la fonction `viewSettings` :

```http
POST /account/viewSettings HTTP/1.1
Host: www.example.com
[other HTTP headers]
Cookie: SessionID=USER_SESSION

username=example_user
```

Réponse valide et légitime :

```html
HTTP1.1 200 OK
[other HTTP headers]

{
  "username": "example_user",
  "email": "example@email.com",
  "address": "Example Address"
}
```

L'attaquant peut essayer d'exécuter cette requête avec le même paramètre `username` :

```html
POST /account/viewCCpincode HTTP/1.1
Host: www.example.com
[other HTTP headers]
Cookie: SessionID=ATTACKER_SESSION

username=example_user
```

Si la réponse de l'attaquant contient les données de `example_user`, alors l'application est vulnérable aux attaques de mouvement latéral, où un utilisateur peut lire ou écrire les données d'un autre utilisateur.

### Test d'accès aux fonctions d'administration

Par exemple, supposons que la fonction `addUser` fasse partie du menu administratif de l'application, et qu'il soit possible d'y accéder en demandant l'URL suivante `https://www.example.com/admin/addUser`.

Ensuite, la requête HTTP suivante est générée lors de l'appel de la fonction "addUser" :

```http
POST /admin/addUser HTTP/1.1
Host: www.example.com
[...]

userID=fakeuser&role=3&group=grp001
```

D'autres questions ou considérations iraient dans le sens suivant :

- Que se passe-t-il si un utilisateur non administrateur tente d'exécuter cette requête ?
- L'utilisateur sera-t-il créé ?
- Si oui, le nouvel utilisateur peut-il utiliser ses privilèges ?

### Test d'accès aux ressources affectées à un rôle différent

Diverses applications configurent des contrôles de ressources en fonction des rôles d'utilisateur. Prenons un exemple de CV ou de CV (curriculum vitae) téléchargé sur un formulaire de carrière vers un compartiment S3.

En tant qu'utilisateur normal, essayez d'accéder à l'emplacement de ces fichiers. Si vous parvenez à les récupérer, les modifier ou les supprimer, alors l'application est vulnérable.

### Test de la gestion des en-têtes de demande spéciale

Certaines applications prennent en charge les en-têtes non standard tels que `X-Original-URL` ou `X-Rewrite-URL` afin de permettre de remplacer l'URL cible dans les requêtes par celle spécifiée dans la valeur de l'en-tête.

Ce comportement peut être exploité dans une situation où l'application se trouve derrière un composant qui applique une restriction de contrôle d'accès en fonction de l'URL de la demande.

Le type de restriction de contrôle d'accès basé sur l'URL de la requête peut être, par exemple, le blocage de l'accès depuis Internet à une console d'administration exposée sur `/console` ou `/admin`.

Pour détecter la prise en charge de l'en-tête `X-Original-URL` ou `X-Rewrite-URL`, les étapes suivantes peuvent être appliquées.

#### 1. Envoyer une requête normale sans en-tête X-Original-Url ou X-Rewrite-Url

```http
GET / HTTP/1.1
Host: www.example.com
[...]
```

#### 2. Envoyer une requête avec un en-tête X-Original-Url pointant vers une ressource inexistante

```html
GET / HTTP/1.1
Host: www.example.com
X-Original-URL: /donotexist1
[...]
```

#### 3. Envoyer une requête avec un en-tête X-Rewrite-Url pointant vers une ressource inexistante

```html
GET / HTTP/1.1
Host: www.example.com
X-Rewrite-URL: /donotexist2
[...]
```

Si la réponse à l'une ou l'autre des requêtes contient des marqueurs indiquant que la ressource n'a pas été trouvée, cela indique que l'application prend en charge les en-têtes de requête spéciale. Ces marqueurs peuvent inclure le code d'état de réponse HTTP 404 ou un message "ressource introuvable" dans le corps de la réponse.

Une fois que la prise en charge de l'en-tête `X-Original-URL` ou `X-Rewrite-URL` a été validée, la tentative de contournement de la restriction de contrôle d'accès peut être exploitée en envoyant la requête attendue à l'application mais en spécifiant une URL "autorisée " par le composant frontal comme URL de requête principale et en spécifiant l'URL cible réelle dans l'en-tête `X-Original-URL` ou `X-Rewrite-URL` selon celui pris en charge. Si les deux sont pris en charge, essayez l'un après l'autre pour vérifier pour quel en-tête le contournement est effectif.

#### 4. Autres en-têtes à prendre en compte

Souvent, les panneaux d'administration ou les éléments de fonctionnalité liés à l'administration ne sont accessibles qu'aux clients sur les réseaux locaux, il peut donc être possible d'abuser de divers en-têtes HTTP liés au proxy ou au transfert pour y accéder. Certains en-têtes et valeurs à tester sont :

- En-têtes :
    - `X-Forwarded-For`
    - `X-Forward-For`
    - `X-Remote-IP`
    - `X-Originating-IP`
    - `X-Remote-Addr`
    - `X-Client-IP`
- Valeurs
    - `127.0.0.1` (ou n'importe quoi dans les espaces d'adressage `127.0.0.0/8` ou `::1/128`)
    - `localhost`
    - Toute adresse [RFC1918](https://tools.ietf.org/html/rfc1918) :
        - `10.0.0.0/8`
        - `172.16.0.0/12`
        - `192.168.0.0/16`
    - Adresses locales du lien : `169.254.0.0/16`

Remarque : L'inclusion d'un élément de port avec l'adresse ou le nom d'hôte peut également aider à contourner les protections périphériques telles que les pare-feu d'applications Web, etc.
Par exemple : `127.0.0.4:80`, `127.0.0.4:443`, `127.0.0.4:43982`

## Correction

Utilisez les principes du moindre privilège sur les utilisateurs, les rôles et les ressources pour vous assurer qu'aucun accès non autorisé ne se produit.

## Outils

- [OWASP Zed Attack Proxy (ZAP)](https://www.zaproxy.org/)
     - [Module complémentaire ZAP : test de contrôle d'accès] (https://www.zaproxy.org/docs/desktop/addons/access-control-testing/)
- [Suite Port Swigger Burp] (https://portswigger.net/burp)
     - [Extension Burp : AuthMatrix](https://github.com/SecurityInnovation/AuthMatrix/)
     - [Extension Burp : Autorize] (https://github.com/Quitten/Autorize)

## Références

[Norme de vérification de la sécurité des applications OWASP 4.0.1](https://github.com/OWASP/ASVS/tree/master/4.0), v4.0.1-1, v4.0.1-4, v4.0.1-9, v4. 0.1-16
