# Test des variables de session exposées

|ID          |
|------------|
|WSTG-SESS-04|

## Sommaire

Les jetons de session (Cookie, SessionID, champ caché), s'ils sont exposés, permettront généralement à un attaquant de se faire passer pour une victime et d'accéder à l'application de manière illégitime. Il est important qu'ils soient protégés à tout moment contre les écoutes clandestines, en particulier pendant leur transit entre le navigateur client et les serveurs d'application.

Les informations ici concernent la manière dont la sécurité du transport s'applique au transfert de données sensibles d'ID de session plutôt qu'aux données en général, et peuvent être plus strictes que les politiques de mise en cache et de transport appliquées aux données servies par le site.

À l'aide d'un proxy personnel, il est possible de vérifier les éléments suivants concernant chaque demande et réponse :

- Protocole utilisé (par exemple, HTTP vs HTTPS)
- En-têtes HTTP
- Corps du message (par exemple, POST ou contenu de la page)

Chaque fois que des données d'ID de session sont transmises entre le client et le serveur, les directives et le corps du protocole, du cache et de la confidentialité doivent être examinés. La sécurité du transport fait ici référence aux ID de session transmis dans les requêtes GET ou POST, les corps de message ou d'autres moyens via des requêtes HTTP valides.

## Objectifs des tests

- Assurez-vous que le cryptage approprié est mis en œuvre.
- Vérifiez la configuration de la mise en cache.
- Évaluer la sécurité du canal et des méthodes.

## Comment tester

### Test des vulnérabilités de chiffrement et de réutilisation des jetons de session

La protection contre les écoutes clandestines est souvent assurée par le cryptage TLS, mais peut incorporer d'autres tunnels ou cryptages. Il convient de noter que le cryptage ou le hachage cryptographique de l'identifiant de session doit être considéré séparément du cryptage de transport, car c'est l'identifiant de session lui-même qui est protégé, et non les données qui peuvent être représentées par celui-ci.

Si l'ID de session peut être présenté par un attaquant à l'application pour y accéder, il doit être protégé en transit pour atténuer ce risque. Il convient donc de s'assurer que le chiffrement est à la fois la valeur par défaut et appliqué pour toute demande ou réponse où l'ID de session est transmis, quel que soit le mécanisme utilisé (par exemple, un champ de formulaire masqué). Des vérifications simples telles que le remplacement de `https://` par `http://` lors de l'interaction avec l'application doivent être effectuées, ainsi que la modification des messages de formulaire pour déterminer si une séparation adéquate entre les sites sécurisés et non sécurisés est mise en œuvre.

Notez que s'il existe également un élément sur le site où l'utilisateur est suivi avec des ID de session mais que la sécurité n'est pas présente (par exemple, en notant les documents publics qu'un utilisateur enregistré télécharge), il est essentiel qu'un ID de session différent soit utilisé. L'ID de session doit donc être surveillé lorsque le client passe des éléments sécurisés aux éléments non sécurisés pour s'assurer qu'un autre est utilisé.

> Chaque fois que l'authentification est réussie, l'utilisateur doit s'attendre à recevoir :
>
> - Un jeton de session différent
> - Un jeton envoyé via un canal crypté chaque fois qu'ils font une requête HTTP

### Test des proxies et des vulnérabilités de mise en cache

Les proxys doivent également être pris en compte lors de l'examen de la sécurité des applications. Dans de nombreux cas, les clients accéderont à l'application par le biais d'une entreprise, d'un FAI ou d'autres proxys ou passerelles compatibles avec le protocole (par exemple, des pare-feu). Le protocole HTTP fournit des directives pour contrôler le comportement des proxys en aval, et la mise en œuvre correcte de ces directives doit également être évaluée.

En général, l'ID de session ne doit jamais être envoyé via un transport non chiffré et ne doit jamais être mis en cache. L'application doit être examinée pour s'assurer que les communications cryptées sont à la fois la valeur par défaut et appliquées pour tout transfert d'ID de session. De plus, chaque fois que l'ID de session est transmis, des directives doivent être en place pour empêcher sa mise en cache par des caches intermédiaires et même locaux.

L'application doit également être configurée pour sécuriser les données dans les caches sur HTTP/1.0 et HTTP/1.1 - RFC 2616 traite des contrôles appropriés en référence à HTTP. HTTP/1.1 fournit un certain nombre de mécanismes de contrôle du cache. `Cache-Control: no-cache` indique qu'un proxy ne doit réutiliser aucune donnée. Bien que `Cache-Control: Private` semble être une directive appropriée, cela permet toujours à un proxy non partagé de mettre en cache des données. Dans le cas des cybercafés ou d'autres systèmes partagés, cela présente un risque évident. Même avec des postes de travail mono-utilisateur, l'ID de session mis en cache peut être exposé via une compromission du système de fichiers ou lorsque des magasins réseau sont utilisés. Les caches HTTP/1.0 ne reconnaissent pas la directive `Cache-Control: no-cache`.

> Les directives `Expires : 0` et `Cache-Control : max-age=0` doivent être utilisées pour garantir que les caches n'exposent pas les données. Chaque demande/réponse transmettant des données d'ID de session doit être examinée pour s'assurer que les directives de cache appropriées sont utilisées.

### Test des vulnérabilités GET et POST

En général, les requêtes GET ne doivent pas être utilisées, car l'ID de session peut être exposé dans les journaux du proxy ou du pare-feu. Ils sont également beaucoup plus faciles à manipuler que d'autres types de transport, même s'il convient de noter que presque tous les mécanismes peuvent être manipulés par le client avec les bons outils. De plus, les attaques [Cross-site Scripting (XSS)](https://owasp.org/www-community/attacks/xss/) sont plus facilement exploitées en envoyant un lien spécialement construit à la victime. Cela est beaucoup moins probable si les données sont envoyées depuis le client sous forme de POST.

Tout code côté serveur recevant des données à partir de requêtes POST doit être testé pour s'assurer qu'il n'accepte pas les données si elles sont envoyées en tant que GET. Par exemple, considérons la requête POST suivante (`http://owaspapp.com/login.asp`) générée par une page de connexion.

```http
POST /login.asp HTTP/1.1
Host: owaspapp.com
[...]
Cookie: ASPSESSIONIDABCDEFG=ASKLJDLKJRELKHJG
Content-Length: 51

Login=Username&password=Password&SessionID=12345678
```

Si login.asp est mal implémenté, il peut être possible de se connecter en utilisant l'URL suivante : `http://owaspapp.com/login.asp?Login=Username&password=Password&SessionID=12345678`

Les scripts côté serveur potentiellement non sécurisés peuvent être identifiés en vérifiant chaque POST de cette manière.

### Test des vulnérabilités de transport

Toute interaction entre le Client et l'Application doit être testée au moins par rapport aux critères suivants.

- Comment les identifiants de session sont-ils transférés ? par exemple, GET, POST, Champ de formulaire (y compris les champs masqués)
- Les identifiants de session sont-ils toujours envoyés via un transport chiffré par défaut ?
- Est-il possible de manipuler l'application pour envoyer des identifiants de session non cryptés ? par exemple, en changeant HTTPS en HTTP ?
- Quelles directives de contrôle de cache sont appliquées aux requêtes/réponses passant des identifiants de session ?
- Ces directives sont-elles toujours présentes ? Si non, où sont les exceptions ?
- Les requêtes GET incorporant l'ID de session sont-elles utilisées ?
- Si POST est utilisé, peut-il être échangé avec GET ?

## Références

### Papiers blanc

- [RFCs 2109 & 2965 – Mécanisme de gestion d'état HTTP [D. Kristol, L. Montulli]](https://www.ietf.org/rfc/rfc2965.txt)
- [RFC 2616 – Protocole de transfert hypertexte - HTTP/1.1](https://www.ietf.org/rfc/rfc2616.txt)
