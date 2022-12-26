# Test des informations sensibles envoyées via des canaux non cryptés

|ID          |
|------------|
|WSTG-CRYP-03|

## Sommaire

Les données sensibles doivent être protégées lorsqu'elles sont transmises sur le réseau. Si les données sont transmises via HTTPS ou cryptées d'une autre manière, le mécanisme de protection ne doit pas avoir de limitations ou de vulnérabilités, comme expliqué dans l'article plus large [Testing for Weak Transport Layer Security](01-Testing_for_Weak_Transport_Layer_Security.md) et dans d'autres documentations OWASP :

- [OWASP Top 10 2017 A3-Sensitive Data Exposure](https://owasp.org/www-project-top-ten/2017/A3_2017-Sensitive_Data_Exposure).
- [OWASP ASVS - Vérification V9](https://github.com/OWASP/ASVS/blob/master/4.0/en/0x17-V9-Communications.md).
- [Aide-mémoire sur la protection de la couche de transport](https://cheatsheetseries.owasp.org/cheatsheets/Transport_Layer_Protection_Cheat_Sheet.html).

En règle générale, si les données doivent être protégées lors de leur stockage, ces données doivent également être protégées lors de leur transmission. Voici quelques exemples de données sensibles :

- Informations utilisées pour l'authentification (par exemple, identifiants, codes PIN, identifiants de session, jetons, cookies…)
- Informations protégées par des lois, des réglementations ou une politique organisationnelle spécifique (par exemple, cartes de crédit, données clients)

Si l'application transmet des informations sensibles via des canaux non cryptés - par ex. HTTP - il est considéré comme un risque pour la sécurité. Les attaquants peuvent prendre le contrôle des comptes en [reniflant le trafic réseau](https://owasp.org/www-community/attacks/Manipulator-in-the-middle_attack). Quelques exemples sont l'authentification de base qui envoie des identifiants d'authentification en texte brut sur HTTP, des identifiants d'authentification basés sur un formulaire envoyés via HTTP ou la transmission en texte brut de toute autre information considérée comme sensible en raison de réglementations, de lois, de politiques organisationnelles ou de la logique métier de l'application.

Exemples d'informations d'identification personnelle (PII) :

- Numéros de sécurité sociale
- Numéros de compte bancaire
- Informations sur le passeport
- Informations liées à la santé
- Informations sur l'assurance médicale
- Informations étudiantes
- Numéros de carte de crédit et de débit
- Permis de conduire et informations d'identification d'État

## Objectifs des tests

- Identifier les informations sensibles transmises par les différents canaux.
- Évaluer la confidentialité et la sécurité des canaux utilisés.

## Comment tester

Divers types d'informations devant être protégées pourraient être transmises par l'application en clair. Pour vérifier si ces informations sont transmises via HTTP au lieu de HTTPS, capturez le trafic entre un client et un serveur d'applications Web nécessitant des informations d'identification. Pour tout message contenant des données sensibles, vérifiez que l'échange s'est produit à l'aide de HTTPS. Voir plus d'informations sur la transmission non sécurisée des identifiants [OWASP Top 10 2017 A3-Sensitive Data Exposure](https://owasp.org/www-project-top-ten/2017/A3_2017-Sensitive_Data_Exposure) ou [Transport Layer Protection Cheat Sheet](https://cheatsheetseries.owasp.org/cheatsheets/Transport_Layer_Protection_Cheat_Sheet.html).

### Exemple 1 : Authentification de base sur HTTP

Un exemple typique est l'utilisation de l'authentification de base sur HTTP. Lors de l'utilisation de l'authentification de base, les informations d'identification de l'utilisateur sont codées plutôt que chiffrées et sont envoyées sous forme d'en-têtes HTTP. Dans l'exemple ci-dessous, le testeur utilise [curl](https://curl.haxx.se/) pour tester ce problème. Notez comment l'application utilise l'authentification de base et HTTP plutôt que HTTPS.

```bash
$ curl -kis http://example.com/restricted/
HTTP/1.1 401 Authorization Required
Date: Fri, 01 Aug 2013 00:00:00 GMT
WWW-Authenticate: Basic realm="Restricted Area"
Accept-Ranges: bytes Vary:
Accept-Encoding Content-Length: 162
Content-Type: text/html

<html><head><title>401 Authorization Required</title></head>
<body bgcolor=white> <h1>401 Authorization Required</h1>  Invalid login credentials!  </body></html>
```

### Exemple 2 : Authentification basée sur un formulaire effectuée sur HTTP

Un autre exemple typique est celui des formulaires d'authentification qui transmettent les identifiants d'authentification de l'utilisateur via HTTP. Dans l'exemple ci-dessous, on peut voir HTTP utilisé dans l'attribut `action` du formulaire. Il est également possible de voir ce problème en examinant le trafic HTTP avec un proxy d'interception.

```html
<form action="http://example.com/login">
    <label for="username">User:</label> <input type="text" id="username" name="username" value=""/><br />
    <label for="password">Password:</label> <input type="password" id="password" name="password" value=""/>
    <input type="submit" value="Login"/>
</form>
```

### Exemple 3 : Cookie contenant l'ID de session envoyé via HTTP

Le cookie d'identification de session doit être transmis sur des canaux protégés. Si le cookie n'a pas le [drapeau sécurisé](../06-Session_Management_Testing/02-Testing_for_Cookies_Attributes.md) défini, il est permis à l'application de le transmettre en clair. Notez ci-dessous que la configuration du cookie est effectuée sans l'indicateur Secure et que l'intégralité du processus de connexion est effectuée en HTTP et non en HTTPS.

```http
https://secure.example.com/login

POST /login HTTP/1.1
Host: secure.example.com
[...]
Referer: https://secure.example.com/
Content-Type: application/x-www-form-urlencoded
Content-Length: 188

HTTP/1.1 302 Found
Date: Tue, 03 Dec 2013 21:18:55 GMT
Server: Apache
Set-Cookie: JSESSIONID=BD99F321233AF69593EDF52B123B5BDA; expires=Fri, 01-Jan-2014 00:00:00 GMT; path=/; domain=example.com; httponly
Location: private/
Content-Length: 0
Content-Type: text/html
```

```http
http://example.com/private

GET /private HTTP/1.1
Host: example.com
[...]
Referer: https://secure.example.com/login
Cookie: JSESSIONID=BD99F321233AF69593EDF52B123B5BDA;

HTTP/1.1 200 OK
Content-Type: text/html;charset=UTF-8
Content-Length: 730
Date: Tue, 25 Dec 2013 00:00:00 GMT
```

### Exemple 4 : Réinitialisation du mot de passe, modification du mot de passe ou autre manipulation de compte via HTTP

Si l'application Web dispose de fonctionnalités permettant à un utilisateur de modifier un compte ou d'appeler un service différent avec des informations d'identification, vérifiez que toutes ces interactions utilisent HTTPS. Les interactions à tester incluent les éléments suivants :

- Formulaires permettant aux utilisateurs de gérer un mot de passe oublié ou d'autres informations d'identification
- Formulaires permettant aux utilisateurs de modifier les informations d'identification
- Formulaires qui demandent à l'utilisateur de s'authentifier auprès d'un autre fournisseur (par exemple, traitement des paiements)

### Exemple 5 : test des informations sensibles au mot de passe dans le code source ou les journaux

Utilisez l'une des techniques suivantes pour rechercher des informations sensibles.

Vérifier si le mot de passe ou la clé de cryptage est codé en dur dans le code source ou les fichiers de configuration.

`grep -r –E "Pass | password | pwd |user | guest| admin | encry | key | decrypt | sharekey " ./PathToSearch/`

Vérifier si les journaux ou le code source peuvent contenir un numéro de téléphone, une adresse e-mail, un identifiant ou toute autre PII. Modifiez l'expression régulière en fonction du format des PII.

`grep -r " {2\}[0-9]\{6\} "  ./PathToSearch/`

## Correction

Utilisez HTTPS pour l'ensemble du site Web et redirigez toutes les requêtes HTTP vers HTTPS.

## Outils

- [curl](https://curl.haxx.se/)
- [grep](http://man7.org/linux/man-pages/man1/egrep.1.html)
- [Wireshark](https://www.wireshark.org/)
- [TCPDUMP](https://www.tcpdump.org/)

## Références

- [Transport non sécurisé OWASP](https://owasp.org/www-community/vulnerabilities/Insecure_Transport)
- [Aide-mémoire sur la sécurité du transport strict HTTP OWASP](https://cheatsheetseries.owasp.org/cheatsheets/HTTP_Strict_Transport_Security_Cheat_Sheet.html)
- [Encryptons](https://letsencrypt.org)
