# Test de la politique de s�curit� du contenu

|ID          |
|------------|
|WSTG-CONF-12|

## Sommaire

La politique de s�curit� du contenu (CSP) est une politique de liste d'autorisation d�clarative appliqu�e via l'en-t�te de r�ponse `Content-Security-Policy` ou l'�l�ment `<meta>` �quivalent. Il permet aux d�veloppeurs de restreindre les sources � partir desquelles des ressources telles que JavaScript, CSS, images, fichiers, etc. sont charg�es. CSP est une technique de d�fense en profondeur efficace pour att�nuer le risque de vuln�rabilit�s telles que Cross Site Scripting (XSS) et Clickjacking.

La politique de s�curit� du contenu prend en charge les directives qui permettent un contr�le granulaire du flux de politiques. (Voir [R�f�rences](#r�f�rences) pour plus de d�tails.)

## Objectifs des tests

- Examinez l'en-t�te ou l'�l�ment Meta Content-Security-Policy pour identifier les erreurs de configuration.

## Comment tester

Pour tester les erreurs de configuration dans les CSP, recherchez les configurations non s�curis�es en examinant l'en-t�te de r�ponse HTTP "Content-Security-Policy" ou l'�l�ment CSP "meta" dans un outil proxy�:

- La directive `unsafe-inline` active les scripts ou les styles en ligne rendant les applications sensibles aux attaques XSS.
- La directive `unsafe-eval` permet d'utiliser `eval()` dans l'application.
- La directive `unsafe-hashes` permet l'utilisation de scripts/styles en ligne, en supposant qu'ils correspondent aux hachages sp�cifi�s.
- Les ressources telles que les scripts peuvent �tre autoris�es � �tre charg�es � partir de n'importe quelle origine en utilisant la source g�n�rique (`*`).
    - Tenez �galement compte des caract�res g�n�riques bas�s sur des correspondances partielles, tels que�: `https://*` ou `*.cdn.com`.
    - D�terminez si les sources r�pertori�es autoris�es fournissent des points de terminaison JSONP qui pourraient �tre utilis�s pour contourner CSP ou la m�me origine.
- Le cadrage peut �tre activ� pour toutes les origines en utilisant la source g�n�rique (`*`) pour la directive `frame-ancestors`.
- Les applications critiques pour l'entreprise devraient n�cessiter l'utilisation d'une politique stricte.

## Correction

Configurez une politique de s�curit� de contenu forte qui r�duit la surface d'attaque de l'application. Les d�veloppeurs peuvent v�rifier la force de la politique de s�curit� du contenu � l'aide d'outils en ligne tels que [Google CSP Evaluator](https://csp-evaluator.withgoogle.com/).

### Politique stricte

Une politique stricte est une politique qui offre une protection contre les attaques classiques stock�es, r�fl�chies et certaines des attaques DOM XSS et devrait �tre l'objectif optimal de toute �quipe essayant d'impl�menter CSP.

Google est all� de l'avant et a mis en place un guide pour adopter un CSP strict bas� sur les nonces. Bas� sur une pr�sentation � [LocoMocoSec](https://speakerdeck.com/lweichselbaum/csp-a-successful-mess-between-hardening-and-mitigation?slide=55), les deux politiques suivantes peuvent �tre utilis�es pour appliquer un politique stricte :

Mod�r� Strict Politique�:

```HTTP
script-src 'nonce-r4nd0m' 'strict-dynamic';
object-src 'none'; base-uri 'none';
```

Politique stricte verrouill�e�:

```HTTP
script-src 'nonce-r4nd0m';
object-src 'none'; base-uri 'none';
```

## Outils

- [�valuateur Google CSP](https://csp-evaluator.withgoogle.com/)
- [Auditeur CSP - Extension Burp Suite](https://portswigger.net/bappstore/35237408a06043e9945a11016fcbac18)
- [CSP Generator Chrome](https://chrome.google.com/webstore/detail/content-security-policy-c/ahlnecfloencbkpfnpljbojmjkfgnmdc) /[Firefox](https://addons.mozilla.org/en-US/firefox/addon/csp-generator/)

## R�f�rences

- [Aide-m�moire sur la politique de s�curit� du contenu OWASP] (https://cheatsheetseries.owasp.org/cheatsheets/Content_Security_Policy_Cheat_Sheet.html)
- [R�seau de d�veloppeurs Mozilla�: Politique de s�curit� du contenu] (https://developer.mozilla.org/en-US/docs/Web/HTTP/CSP)
- [CSP niveau 3 W3C](https://www.w3.org/TR/CSP3/)
- [CSP avec Google](https://csp.withgoogle.com/docs/index.html)
- [Content-Security-Policy](https://content-security-policy.com/)
- [�valuateur Google CSP](https://csp-evaluator.withgoogle.com/)
- [CSP Un g�chis r�ussi entre durcissement et att�nuation](https://speakerdeck.com/lweichselbaum/csp-a-successful-mess-between-hardening-and-mitigation)
- [Le ??mot-cl� de la liste des sources unsafe-hashes] (https://content-security-policy.com/unsafe-hashes/)
