# Tester la s�curit� du transport strict HTTP

|ID          |
|------------|
|WSTG-CONF-07|

## Sommaire

La fonctionnalit� HTTP Strict Transport Security (HSTS) permet � une application Web d'informer le navigateur via l'utilisation d'un en-t�te de r�ponse sp�cial qu'elle ne doit jamais �tablir de connexion aux serveurs de domaine sp�cifi�s � l'aide de HTTP non chiffr�. Au lieu de cela, il devrait �tablir automatiquement toutes les demandes de connexion pour acc�der au site via HTTPS. Il emp�che �galement les utilisateurs de remplacer les erreurs de certificat.

Compte tenu de l'importance de cette mesure de s�curit�, il est prudent de v�rifier que le site Web utilise cet en-t�te HTTP afin de s'assurer que toutes les donn�es transitent crypt�es entre le navigateur Web et le serveur.

L'en-t�te HTTP strict transport security utilise deux directives�:

- `max-age`�: pour indiquer le nombre de secondes pendant lesquelles le navigateur doit convertir automatiquement toutes les requ�tes HTTP en HTTPS.
- `includeSubDomains`�: pour indiquer que tous les sous-domaines associ�s doivent utiliser HTTPS.
- `preload` Unofficial�: pour indiquer que le ou les domaines sont sur la ou les listes de pr�chargement et que les navigateurs ne doivent jamais se connecter sans HTTPS.
    - Ceci est pris en charge par tous les principaux navigateurs mais ne fait pas officiellement partie de la sp�cification. (Voir [hstspreload.org](https://hstspreload.org/) pour plus d'informations.)

Voici un exemple d'impl�mentation d'en-t�te HSTS�:

`Strict-Transport-Security�: max-age=31536000�; inclure les sous-domaines`

L'utilisation de cet en-t�te par les applications Web doit �tre v�rifi�e pour d�terminer si les probl�mes de s�curit� suivants peuvent survenir�:

- Les attaquants reniflent le trafic r�seau et acc�dent aux informations transf�r�es via un canal non crypt�.
- Les attaquants exploitant un manipulateur au milieu attaquent en raison du probl�me d'acceptation de certificats non fiables.
- Les utilisateurs qui ont saisi par erreur une adresse dans le navigateur en mettant HTTP au lieu de HTTPS, ou les utilisateurs qui ont cliqu� sur un lien dans une application Web qui a indiqu� par erreur l'utilisation du protocole HTTP.

## Objectifs des tests

- Examinez l'en-t�te HSTS et sa validit�.

## Comment tester

La pr�sence de l'en-t�te HSTS peut �tre confirm�e en examinant la r�ponse du serveur via un proxy d'interception ou en utilisant curl comme suit :

```bash
$ curl -s -D- https://owasp.org | grep -i strict
Strict-Transport-Security�: max-age=31536000
```

## R�f�rences

- [OWASP HTTP Strict Transport Security](https://cheatsheetseries.owasp.org/cheatsheets/HTTP_Strict_Transport_Security_Cheat_Sheet.html)
- [S�rie de tutoriels OWASP Appsec - �pisode 4 : Strict Transport Security](https://www.youtube.com/watch?v=zEV3HOuM_Vw)
- [Sp�cification HSTS] (https://tools.ietf.org/html/rfc6797)
- [Activer la s�curit� HTTP Strict Transport dans Apache](https://https.cio.gov/hsts/)
- [Activer la s�curit� HTTP Strict Transport dans Nginx](https://www.nginx.com/blog/http-strict-transport-security-hsts-and-nginx/)
