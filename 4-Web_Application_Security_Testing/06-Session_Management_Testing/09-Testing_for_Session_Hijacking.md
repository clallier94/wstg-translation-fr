# Test de piratage de session

|ID          |
|------------|
|WSTG-SESS-09|

## Sommaire

Un attaquant qui accède aux cookies de session utilisateur peut se faire passer pour eux en présentant ces cookies. Cette attaque est connue sous le nom de détournement de session. Lorsque l'on considère les attaquants de réseau, c'est-à-dire les attaquants qui contrôlent le réseau utilisé par la victime, les cookies de session peuvent être indûment exposés à l'attaquant via HTTP. Pour éviter cela, les cookies de session doivent être marqués avec l'attribut "Sécurisé" afin qu'ils ne soient communiqués que via HTTPS.

Notez que l'attribut `Secure` doit également être utilisé lorsque l'application Web est entièrement déployée sur HTTPS, sinon l'attaque de vol de cookie suivante est possible. Supposons que `exemple.com` est entièrement déployé sur HTTPS, mais ne marque pas ses cookies de session comme `Secure`. Les étapes d'attaque suivantes sont possibles :

1. La victime envoie une requête à `http://another-site.com`.
2. L'attaquant corrompt la réponse correspondante pour qu'elle déclenche une requête vers `http://exemple.com`.
3. Le navigateur essaie maintenant d'accéder à `http://exemple.com`.
4. Bien que la demande échoue, les cookies de session sont divulgués en clair via HTTP.

Alternativement, le piratage de session peut être empêché en interdisant l'utilisation de HTTP à l'aide de [HSTS](https://en.wikipedia.org/wiki/HTTP_Strict_Transport_Security). Notez qu'il y a ici une subtilité liée à la portée des cookies. En particulier, l'adoption complète du HSTS est requise lorsque les cookies de session sont émis avec l'ensemble d'attributs `Domain`.

L'adoption complète du HSTS est décrite dans un article intitulé *Testing for Integrity Flaws in Web Sessions* par Stefano Calzavara, Alvise Rabitti, Alessio Ragazzo et Michele Bugliesi. L'adoption complète du HSTS se produit lorsqu'un hôte active le HSTS pour lui-même et tous ses sous-domaines. L'adoption partielle du HSTS se produit lorsqu'un hôte active le HSTS uniquement pour lui-même.

Avec l'attribut `Domain` défini, les cookies de session peuvent être partagés entre les sous-domaines. L'utilisation de HTTP avec des sous-domaines doit être évitée pour empêcher la divulgation de cookies non chiffrés envoyés via HTTP. Pour illustrer cette faille de sécurité, supposons que le site Web `exemple.com` active HSTS sans l'option `includeSubDomains`. Le site Web émet des cookies de session avec l'attribut `Domain` défini sur `exemple.com`. L'attaque suivante est possible :

1. La victime envoie une requête à `http://another-site.com`.
2. L'attaquant corrompt la réponse correspondante pour qu'elle déclenche une requête vers `http://fake.exemple.com`.
3. Le navigateur essaie maintenant d'accéder à `http://fake.exemple.com`, ce qui est autorisé par la configuration HSTS.
4. Étant donné que la requête est envoyée à un sous-domaine de `exemple.com` avec l'attribut `Domain` défini, elle inclut les cookies de session, qui sont divulgués en clair via HTTP.

Le HSTS complet doit être activé sur le domaine apex pour empêcher cette attaque.

## Objectifs des tests

- Identifier les cookies de session vulnérables.
- Détourner les cookies vulnérables et évaluer le niveau de risque.

## Comment tester

La stratégie de test est ciblée sur les attaquants du réseau, elle ne doit donc être appliquée qu'aux sites sans adoption complète du HSTS (les sites avec une adoption complète du HSTS sont sécurisés, car leurs cookies ne sont pas communiqués via HTTP). Nous supposons que nous avons deux comptes de test sur le site Web testé, un pour agir en tant que victime et un pour agir en tant qu'attaquant. Nous simulons un scénario dans lequel l'attaquant vole tous les cookies qui ne sont pas protégés contre la divulgation via HTTP et les présente au site Web pour accéder au compte de la victime. Si ces cookies sont suffisants pour agir au nom de la victime, le détournement de session est possible.

Voici les étapes pour exécuter ce test :

1. Connectez-vous au site en tant que victime et accédez à toute page proposant une fonction sécurisée nécessitant une authentification.
2. Supprimez de la boîte à cookies tous les cookies qui satisfont à l'une des conditions suivantes.
    - s'il n'y a pas d'adoption du HSTS : l'attribut `Secure` est défini.
    - en cas d'adoption partielle du HSTS : l'attribut `Secure` est défini ou l'attribut `Domain` n'est pas défini.
3. Enregistrez un instantané de la boîte à biscuits.
4. Déclenchez la fonction sécurisée identifiée à l'étape 1.
5. Observez si l'opération à l'étape 4 a été effectuée avec succès. Si c'est le cas, l'attaque a réussi.
6. Effacez la boîte à cookies, connectez-vous en tant qu'attaquant et accédez à la page à l'étape 1.
7. Écrivez dans la boîte à biscuits, un par un, les cookies enregistrés à l'étape 3.
8. Déclenchez à nouveau la fonction sécurisée identifiée à l'étape 1.
9. Videz la boîte à cookies et reconnectez-vous en tant que victime.
10. Observez si l'opération à l'étape 8 a été effectuée avec succès dans le compte de la victime. Si tel est le cas, l'attaque a réussi; sinon, le site est sécurisé contre le détournement de session.

Nous recommandons d'utiliser deux machines ou navigateurs différents pour la victime et l'attaquant. Cela vous permet de réduire le nombre de faux positifs si l'application Web prend des empreintes digitales pour vérifier l'accès activé à partir d'un cookie donné. Une variante plus courte mais moins précise de la stratégie de test ne nécessite qu'un seul compte de test. Il suit le même schéma, mais il s'arrête à l'étape 5 (notez que cela rend l'étape 3 inutile).

## Outils

- [OWASP ZAP](https://www.zaproxy.org)
- [JHijack - un outil de piratage de session numérique](https://sourceforge.net/projects/jhijack/)
