# Tester la sécurité du transport strict HTTP

|ID          |
|------------|
|WSTG-CONF-07|

## Sommaire

La fonctionnalité HTTP Strict Transport Security (HSTS) permet à une application Web d'informer le navigateur via l'utilisation d'un en-tête de réponse spécial qu'elle ne doit jamais établir de connexion aux serveurs de domaine spécifiés à l'aide de HTTP non chiffré. Au lieu de cela, il devrait établir automatiquement toutes les demandes de connexion pour accéder au site via HTTPS. Il empêche également les utilisateurs de remplacer les erreurs de certificat.

Compte tenu de l'importance de cette mesure de sécurité, il est prudent de vérifier que le site Web utilise cet en-tête HTTP afin de s'assurer que toutes les données transitent cryptées entre le navigateur Web et le serveur.

L'en-tête HTTP strict transport security utilise deux directives :

- `max-age` : pour indiquer le nombre de secondes pendant lesquelles le navigateur doit convertir automatiquement toutes les requêtes HTTP en HTTPS.
- `includeSubDomains` : pour indiquer que tous les sous-domaines associés doivent utiliser HTTPS.
- `preload` Unofficial : pour indiquer que le ou les domaines sont sur la ou les listes de préchargement et que les navigateurs ne doivent jamais se connecter sans HTTPS.
    - Ceci est pris en charge par tous les principaux navigateurs mais ne fait pas officiellement partie de la spécification. (Voir [hstspreload.org](https://hstspreload.org/) pour plus d'informations.)

Voici un exemple d'implémentation d'en-tête HSTS :

`Strict-Transport-Security : max-age=31536000 ; inclure les sous-domaines`

L'utilisation de cet en-tête par les applications Web doit être vérifiée pour déterminer si les problèmes de sécurité suivants peuvent survenir :

- Les attaquants reniflent le trafic réseau et accèdent aux informations transférées via un canal non crypté.
- Les attaquants exploitant un manipulateur au milieu attaquent en raison du problème d'acceptation de certificats non fiables.
- Les utilisateurs qui ont saisi par erreur une adresse dans le navigateur en mettant HTTP au lieu de HTTPS, ou les utilisateurs qui ont cliqué sur un lien dans une application Web qui a indiqué par erreur l'utilisation du protocole HTTP.

## Objectifs des tests

- Examinez l'en-tête HSTS et sa validité.

## Comment tester

La présence de l'en-tête HSTS peut être confirmée en examinant la réponse du serveur via un proxy d'interception ou en utilisant curl comme suit :

```bash
$ curl -s -D- https://owasp.org | grep -i strict
Strict-Transport-Security : max-age=31536000
```

## Références

- [OWASP HTTP Strict Transport Security](https://cheatsheetseries.owasp.org/cheatsheets/HTTP_Strict_Transport_Security_Cheat_Sheet.html)
- [Série de tutoriels OWASP Appsec - Épisode 4 : Strict Transport Security](https://www.youtube.com/watch?v=zEV3HOuM_Vw)
- [Spécification HSTS](https://tools.ietf.org/html/rfc6797)
- [Activer la sécurité HTTP Strict Transport dans Apache](https://https.cio.gov/hsts/)
- [Activer la sécurité HTTP Strict Transport dans Nginx](https://www.nginx.com/blog/http-strict-transport-security-hsts-and-nginx/)
