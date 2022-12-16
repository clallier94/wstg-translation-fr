# Test de la politique de sécurité du contenu

|ID          |
|------------|
|WSTG-CONF-12|

## Sommaire

La politique de sécurité du contenu (CSP) est une politique de liste d'autorisation déclarative appliquée via l'en-tête de réponse `Content-Security-Policy` ou l'élément `<meta>` équivalent. Il permet aux développeurs de restreindre les sources à partir desquelles des ressources telles que JavaScript, CSS, images, fichiers, etc. sont chargées. CSP est une technique de défense en profondeur efficace pour atténuer le risque de vulnérabilités telles que Cross Site Scripting (XSS) et Clickjacking.

La politique de sécurité du contenu prend en charge les directives qui permettent un contrôle granulaire du flux de politiques. (Voir [Références](#références) pour plus de détails.)

## Objectifs des tests

- Examinez l'en-tête ou l'élément Meta Content-Security-Policy pour identifier les erreurs de configuration.

## Comment tester

Pour tester les erreurs de configuration dans les CSP, recherchez les configurations non sécurisées en examinant l'en-tête de réponse HTTP `Content-Security-Policy` ou l'élément CSP `meta` dans un outil proxy :

- La directive `unsafe-inline` active les scripts ou les styles en ligne rendant les applications sensibles aux attaques XSS.
- La directive `unsafe-eval` permet d'utiliser `eval()` dans l'application.
- La directive `unsafe-hashes` permet l'utilisation de scripts/styles en ligne, en supposant qu'ils correspondent aux hachages spécifiés.
- Les ressources telles que les scripts peuvent être autorisées à être chargées à partir de n'importe quelle origine en utilisant la source générique (`*`).
    - Tenez également compte des caractères génériques basés sur des correspondances partielles, tels que : `https://*` ou `*.cdn.com`.
    - Déterminez si les sources répertoriées autorisées fournissent des points de terminaison JSONP qui pourraient être utilisés pour contourner CSP ou la même origine.
- Le cadrage peut être activé pour toutes les origines en utilisant la source générique (`*`) pour la directive `frame-ancestors`.
- Les applications critiques pour l'entreprise devraient nécessiter l'utilisation d'une politique stricte.

## Correction

Configurez une politique de sécurité de contenu forte qui réduit la surface d'attaque de l'application. Les développeurs peuvent vérifier la force de la politique de sécurité du contenu à l'aide d'outils en ligne tels que [Google CSP Evaluator](https://csp-evaluator.withgoogle.com/).

### Politique stricte

Une politique stricte est une politique qui offre une protection contre les attaques classiques stockées, réfléchies et certaines des attaques DOM XSS et devrait être l'objectif optimal de toute équipe essayant d'implémenter CSP.

Google est allé de l'avant et a mis en place un guide pour adopter un CSP strict basé sur les nonces. Basé sur une présentation à [LocoMocoSec](https://speakerdeck.com/lweichselbaum/csp-a-successful-mess-between-hardening-and-mitigation?slide=55), les deux politiques suivantes peuvent être utilisées pour appliquer un politique stricte :

Modéré Strict Politique :

```HTTP
script-src 'nonce-r4nd0m' 'strict-dynamic';
object-src 'none'; base-uri 'none';
```

Politique stricte verrouillée :

```HTTP
script-src 'nonce-r4nd0m';
object-src 'none'; base-uri 'none';
```

## Outils

- [Évaluateur Google CSP](https://csp-evaluator.withgoogle.com/)
- [Auditeur CSP - Extension Burp Suite](https://portswigger.net/bappstore/35237408a06043e9945a11016fcbac18)
- [CSP Generator Chrome](https://chrome.google.com/webstore/detail/content-security-policy-c/ahlnecfloencbkpfnpljbojmjkfgnmdc) /[Firefox](https://addons.mozilla.org/en-US/firefox/addon/csp-generator/)

## Références

- [Aide-mémoire sur la politique de sécurité du contenu OWASP](https://cheatsheetseries.owasp.org/cheatsheets/Content_Security_Policy_Cheat_Sheet.html)
- [Réseau de développeurs Mozilla : Politique de sécurité du contenu](https://developer.mozilla.org/en-US/docs/Web/HTTP/CSP)
- [CSP niveau 3 W3C](https://www.w3.org/TR/CSP3/)
- [CSP avec Google](https://csp.withgoogle.com/docs/index.html)
- [Content-Security-Policy](https://content-security-policy.com/)
- [Évaluateur Google CSP](https://csp-evaluator.withgoogle.com/)
- [CSP Un gâchis réussi entre durcissement et atténuation](https://speakerdeck.com/lweichselbaum/csp-a-successful-mess-between-hardening-and-mitigation)
- [Le ??mot-clé de la liste des sources unsafe-hashes](https://content-security-policy.com/unsafe-hashes/)
