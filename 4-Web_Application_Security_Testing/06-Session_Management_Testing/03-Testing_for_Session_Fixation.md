# Test de durée de session

|ID          |
|------------|
|WSTG-SESS-03|

## Sommaire

La fixation de session est activée par la pratique non sécurisée consistant à conserver la même valeur des cookies de session avant et après l'authentification. Cela se produit généralement lorsque les cookies de session sont utilisés pour stocker des informations d'état avant même la connexion, par exemple, pour ajouter des articles à un panier avant de s'authentifier pour le paiement.

Dans l'exploit générique des vulnérabilités de fixation de session, un attaquant peut obtenir un ensemble de cookies de session à partir du site Web cible sans s'authentifier au préalable. L'attaquant peut alors forcer ces cookies dans le navigateur de la victime en utilisant différentes techniques. Si la victime s'authentifie ultérieurement sur le site Web cible et que les cookies ne sont pas actualisés lors de la connexion, la victime sera identifiée par les cookies de session choisis par l'attaquant. L'attaquant est alors en mesure de se faire passer pour la victime avec ces cookies connus.

Ce problème peut être résolu en actualisant les cookies de session après le processus d'authentification. Alternativement, l'attaque peut être empêchée en garantissant l'intégrité des cookies de session. Lorsque vous envisagez des attaquants de réseau, c'est-à-dire des attaquants qui contrôlent le réseau utilisé par la victime, utilisez le [HSTS](https://en.wikipedia.org/wiki/HTTP_Strict_Transport_Security) complet ou ajoutez le préfixe `__Host-` / `__Secure-` au nom du cookie.

L'adoption complète du HSTS se produit lorsqu'un hôte active le HSTS pour lui-même et tous ses sous-domaines. Ceci est décrit dans un article intitulé *Testing for Integrity Flaws in Web Sessions* par Stefano Calzavara, Alvise Rabitti, Alessio Ragazzo et Michele Bugliesi.

## Objectifs des tests

- Analyser le mécanisme d'authentification et son déroulement.
- Forcer les cookies et évaluer l'impact.

## Comment tester

Dans cette section, nous donnons une explication de la stratégie de test qui sera présentée dans la section suivante.

La première étape consiste à faire une demande au site à tester (*e.g.* `www.exemple.com`). Si le testeur demande ce qui suit :

```http
GET / HTTP/1.1
Host: www.exemple.com
```

Ils obtiendront la réponse suivante :

```html
HTTP/1.1 200 OK
Date: Wed, 14 Aug 2008 08:45:11 GMT
Server: IBM_HTTP_Server
Set-Cookie: JSESSIONID=0000d8eyYq3L0z2fgq10m4v-rt4:-1; Path=/; secure
Cache-Control: no-cache="set-cookie,set-cookie2"
Expires: Thu, 01 Dec 1994 16:00:00 GMT
Keep-Alive: timeout=5, max=100
Connection: Keep-Alive
Content-Type: text/html;charset=Cp1254
Content-Language: en-US
```

L'application définit un nouvel identifiant de session, `JSESSIONID=0000d8eyYq3L0z2fgq10m4v-rt4:-1`, pour le client.

Ensuite, si le testeur réussit à s'authentifier auprès de l'application avec le POST suivant sur `https://www.exemple.com/authentication.php` :

```http
POST /authentication.php HTTP/1.1
Host: www.exemple.com
[...]
Referer: http://www.exemple.com
Cookie: JSESSIONID=0000d8eyYq3L0z2fgq10m4v-rt4:-1
Content-Type: application/x-www-form-urlencoded
Content-length: 57

Name=Meucci&wpPassword=secret!&wpLoginattempt=Log+in
```

Le testeur observe la réponse suivante du serveur :

```http
HTTP/1.1 200 OK
Date: Thu, 14 Aug 2008 14:52:58 GMT
Server: Apache/2.2.2 (Fedora)
X-Powered-By: PHP/5.1.6
Content-language: en
Cache-Control: private, must-revalidate, max-age=0
X-Content-Encoding: gzip
Content-length: 4090
Connection: close
Content-Type: text/html; charset=UTF-8
...
HTML data
...
```

Comme aucun nouveau cookie n'a été émis lors d'une authentification réussie, le testeur sait qu'il est possible d'effectuer un détournement de session à moins que l'intégrité du cookie de session ne soit assurée.

Le testeur peut envoyer un identifiant de session valide à un utilisateur (éventuellement en utilisant une astuce d'ingénierie sociale), attendre qu'il s'authentifie, puis vérifier que des privilèges ont été attribués à ce cookie.

### Test avec les cookies forcés

Cette stratégie de test est ciblée sur les attaquants du réseau, elle ne doit donc être appliquée qu'aux sites sans adoption complète du HSTS (les sites avec une adoption complète du HSTS sont sécurisés, car tous leurs cookies sont intègres). Nous supposons que nous avons deux comptes de test sur le site Web testé, un pour agir en tant que victime et un pour agir en tant qu'attaquant. Nous simulons un scénario où l'attaquant force dans le navigateur de la victime tous les cookies qui ne sont pas fraîchement émis après la connexion et qui n'ont pas d'intégrité. Après la connexion de la victime, l'attaquant présente les cookies forcés au site Web pour accéder au compte de la victime : s'ils sont suffisants pour agir au nom de la victime, la fixation de session est possible.

Voici les étapes pour exécuter ce test :

1. Accédez à la page de connexion du site Web.
2. Enregistrez un instantané de la boîte à cookies avant de vous connecter, à l'exclusion des cookies qui contiennent le préfixe `__Host-` ou `__Secure-` dans leur nom.
3. Connectez-vous au site en tant que victime et accédez à toute page proposant une fonction sécurisée nécessitant une authentification.
4. Réglez la boîte à biscuits sur l'instantané pris à l'étape 2.
5. Déclenchez la fonction sécurisée identifiée à l'étape 3.
6. Observez si l'opération à l'étape 5 a été effectuée avec succès. Si c'est le cas, l'attaque a réussi.
7. Effacez la boîte à cookies, connectez-vous en tant qu'attaquant et accédez à la page à l'étape 3.
8. Écrivez dans la boîte à biscuits, un par un, les biscuits enregistrés à l'étape 2.
9. Déclenchez à nouveau la fonction sécurisée identifiée à l'étape 3.
10. Videz la boîte à cookies et reconnectez-vous en tant que victime.
11. Observez si l'opération à l'étape 9 a été effectuée avec succès dans le compte de la victime. Si tel est le cas, l'attaque a réussi; sinon, le site est protégé contre la fixation de session.

Nous recommandons d'utiliser deux machines ou navigateurs différents pour la victime et l'attaquant. Cela vous permet de réduire le nombre de faux positifs si l'application Web prend des empreintes digitales pour vérifier l'accès activé à partir d'un cookie donné. Une variante plus courte mais moins précise de la stratégie de test ne nécessite qu'un seul compte de test. Il suit les mêmes étapes, mais il s'arrête à l'étape 6.

## Correction

Implémentez un renouvellement de jeton de session après qu'un utilisateur s'est authentifié avec succès.

L'application doit toujours d'abord invalider l'ID de session existant avant d'authentifier un utilisateur, et si l'authentification réussit, fournir un autre ID de session.

## Outils

- [OWASP ZAP](https://www.zaproxy.org)

## Références

- [Durée de session](https://owasp.org/www-community/attacks/Session_fixation)
- [Sécurité ACROS](https://www.acrossecurity.com/papers/session_fixation.pdf)
- [Chris Shiflett](http://shiflett.org/articles/session-fixation)
